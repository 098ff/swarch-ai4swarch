# ADR-0002: ใช้ RabbitMQ และ Transactional Outbox สำหรับงานประมวลผลหลังการสั่งซื้อ

- **Status:** Proposed
- **Date:** 2026-10-22
- **Deciders:** ทีมสถาปัตยกรรมซอฟต์แวร์ (นิสิต / Software Architect)

## Context and problem

ข้อกำหนด **R-12** ระบุว่าหลังจากลูกค้ายืนยันการสั่งซื้อ (`PlaceOrder`) สำเร็จ Online Boutique ต้องดำเนินการ 3 ประการ:
1. ส่งอีเมลยืนยันคำสั่งซื้อผ่าน **MailFast** (ผู้ให้บริการภายนอกรายใหม่ แทน mock `emailservice`)
2. มอบแต้มสะสม (**loyalty points**) ผ่าน `loyaltyservice` ที่จะพัฒนาขึ้นใหม่
3. บันทึก analytics event **"order placed"** ส่งให้ทีมการตลาด
และในปีหน้าฝ่ายการตลาดมีแผนจะเพิ่มงานหลังการสั่งซื้ออีก เช่น SMS, fraud scoring และ recommendations update

### ข้อเท็จจริงและข้อจำกัดปัจจุบัน (ตาม `docs/scenario.md`):
- ปัจจุบัน `checkoutservice.PlaceOrder` ทำงานแบบ **synchronous ผ่าน gRPC** ต่อเนื่องกัน (`paymentservice` -> `shippingservice` -> `emailservice`) โดย latency p95 อยู่ที่ ~300 ms แต่มี SLA กำหนดว่า p95 ต้อง **ไม่เกิน 500 ms**
- ช่วงเทศกาล 11.11 มีปริมาณโหลดพีกสูงสุดที่ **200 orders/second**
- บริการภายนอก **MailFast** มี latency สูงมาก (p95 = 1.5 วินาที, p99 = 8 วินาที), มี failure rate ราว 0.5% และมีข้อจำกัด Rate Limit ห้ามยิงเกิน **50 emails/second**
- แต้มสะสมต้องมอบให้สำเร็จ **ภายใน 1 นาที** และต้องได้ **ครั้งเดียวเท่านั้น (Exactly-once outcome)** ห้ามให้ซ้ำ
- ความพร้อมใช้งาน (Availability): คำสั่งซื้อ **ต้องไม่ล้มเหลว** เพียงเพราะ MailFast, loyalty หรือ analytics ล่ม
- ทีมงานมีนักพัฒนา 4 คน เชี่ยวชาญ Go/gRPC และ 2 คนเคยใช้งาน **RabbitMQ** ระบบรันบน Kubernetes และมีนโยบายเลือกใช้ส่วนประกอบแบบ Open-source / Self-hosted

## Decision drivers

1. **Availability:** การล่มหรือช้าของ MailFast, loyaltyservice หรือ analytics ต้องไม่กระทบการสั่งซื้อของลูกค้า
2. **Performance (Latency):** รักษา latency p95 ของ `PlaceOrder` ให้ไม่เกิน 500 ms (การเรียก MailFast แบบ synchronous ซึ่งกินเวลา 1.5–8 วินาทีจะทำให้ผิด SLA ทันที)
3. **Scalability & Rate Limiting:** รองรับโหลดพีก 200 orders/s โดยต้องควบคุมอัตราส่งให้ MailFast ไม่เกิน 50 req/s
4. **Correctness (Idempotency):** รับประกันว่าลูกค้าได้รับแต้มสะสมภายใน 1 นาที และไม่มีทางได้รับแต้มซ้ำแม้มีการส่งข้อความซ้ำ (At-least-once delivery)
5. **Data Durability:** ป้องกันการสูญหายของข้อความหาก broker หรือ service ขัดข้องหลังสั่งซื้อสำเร็จ
6. **Modifiability & Operational Feasibility:** เพิ่ม post-order actions ใหม่ได้ง่ายในอนาคต โดยใช้เทคโนโลยีที่ทีมดูแลได้เองบน Kubernetes

## Considered options

1. **Synchronous gRPC calls จาก `checkoutservice`**: เรียก MailFast, loyaltyservice และ analytics เรียงกันเหมือนโค้ดปัจจุบัน
2. **Asynchronous Event-Driven ด้วย RabbitMQ พร้อม Transactional Outbox Pattern**: เมื่อสั่งซื้อสำเร็จ ให้เขียน event ลง Outbox table แล้วนำส่งเข้า RabbitMQ โดยให้ consumers แยกกันประมวลผลอิสระ
3. **Synchronous Post-Order Orchestration Service**: สร้าง service ตัวกลางคอยยิงกระจายแบบ parallel synchronous ไปยังบริการปลายทาง

### Pros and cons of the options

| Option | Pros | Cons |
|---|---|---|
| **1. Synchronous gRPC** | โค้ดตรงไปตรงมา ไม่ต้องเพิ่มโครงสร้างพื้นฐานใหม่ | ผิด SLA latency (1.5s > 500ms); ล้มเหลวง่ายเมื่อระบบภายนอกล่ม; รองรับโหลดพีก 200 req/s ไม่ได้เพราะ MailFast ติด rate limit 50 req/s |
| **2. RabbitMQ + Outbox (เลือก)** | แยก checkout ออกจากงานเบื้องหลังชัดเจน (p95 < 500ms); คิวทำหน้าที่ buffer โหลด 200 orders/s ส่งให้ MailFast ที่ 50/s; Outbox ป้องกันข้อมูลหาย; ขยาย consumer ใหม่ได้ง่าย; ตรงกับทักษะ RabbitMQ ในทีม | เพิ่มภาระในการติดตั้ง จัดการ และเฝ้าระวัง RabbitMQ Cluster และ Outbox publisher บน Kubernetes |
| **3. Orchestrator Service** | ซ่อนความซับซ้อนของปลายทางจาก checkoutservice | ยังคงมีปัญหาเรื่อง latency และ failure propagation หากปลายทางล่ม; ไม่สามารถ buffer โหลดเกิน 50 req/s ของ MailFast ได้โดยตรง |

## Decision

เลือก **Option 2: Asynchronous Event-Driven Architecture โดยใช้ RabbitMQ ร่วมกับ Transactional Outbox Pattern** โดยมีรายละเอียดสถาปัตยกรรมดังนี้:

1. **Transactional Outbox ใน `checkoutservice`:**
   - เมื่อตัดเงินและจองจัดส่งสำเร็จ `checkoutservice` จะบันทึกสถานะคำสั่งซื้อพร้อมเขียน event `OrderPlaced` ลงใน local persistent storage ใน transaction เดียวกัน ก่อนตอบยืนยันสำเร็จให้ลูกค้า (latency เพิ่มเพียงระดับมิลลิวินาที ไม่กระทบ SLA 500 ms)
   - มี Background Relay Worker คอยอ่าน event จาก Outbox ไป publish เข้า RabbitMQ พร้อม acknowledge เมื่อส่งสำเร็จ เพื่อการันตี **At-least-once delivery** ป้องกัน message สูญหาย
2. **RabbitMQ Topology & Independent Queues:**
   - ใช้ Fanout Exchange ชื่อ `order.v1.events` กระจายข้อความไปยัง 3 คิวแยกอิสระ ได้แก่:
     - `order.email.queue` (สำหรับ Email Consumer ส่ง MailFast)
     - `order.loyalty.queue` (สำหรับ `loyaltyservice`)
     - `order.analytics.queue` (สำหรับ Analytics Consumer)
   - หากคิวใดคิวหนึ่งมีปัญหา จะไม่กระทบคิวอื่น (Fault Isolation)
3. **Dead-Letter Exchange (DLX) & Retry Policy:**
   - ทุกคิวจะมี Dead-Letter Queue (DLQ) รองรับ หากประมวลผลล้มเหลวจะทำ Exponential Backoff Retry สูงสุด 3 ครั้ง หากไม่สำเร็จจะย้ายเข้า DLQ เพื่อให้ทีมตรวจสอบ ไม่ให้บล็อกคิวหลัก
4. **Rate Limiting สำหรับ MailFast:**
   - Email Consumer จะติดตั้ง Rate Limiter (Token Bucket) จำกัดอัตราการยิงออกไม่เกิน **45 requests/second** (ต่ำกว่า limit 50/s เพื่อความปลอดภัย) โดยคิว `order.email.queue` จะทำหน้าที่เป็น Buffer ซับโหลดพีก 200 orders/s
5. **Idempotency สำหรับ Loyalty Points:**
   - ทุก event มี `order_id` กำกับ โดย `loyaltyservice` จะทำ Unique Constraint / Idempotency Key บน `order_id` ก่อนมอบแต้ม หากพบ `order_id` เดิมซ้ำจะตอบรับว่าสำเร็จโดยไม่เพิ่มแต้มซ้ำ ทำให้ได้ผลลัพธ์ **Exactly-once outcome** อย่างแท้จริง

## Consequences

- **Positive:**
  - **Performance:** เวลาตอบสนอง `PlaceOrder` คงที่ที่ ~300 ms (ต่ำกว่า SLA 500 ms มาก) เพราะไม่ต้องรอ MailFast (p95 1.5s)
  - **Availability:** แม้ MailFast หรือ loyaltyservice จะล่ม คำสั่งซื้อของลูกค้ายังคงสำเร็จ 100% ข้อความจะรออยู่ในคิว
  - **Scalability:** สามารถรับทราฟฟิกช่วงพีก 11.11 ที่ 200 orders/s ได้สบาย โดยคิวจะทยอยปล่อยงานให้ MailFast ตาม rate limit ที่รับไหว
  - **Modifiability:** ในอนาคตเมื่อฝ่ายการตลาดต้องการเพิ่ม SMS, Fraud Scoring หรือ Recommendations สามารถเพิ่ม Queue และ Consumer ใหม่มาผูกกับ `order.v1.events` ได้ทันทีโดยไม่ต้องแก้โค้ด `checkoutservice`
- **Negative:**
  - **Operational Complexity:** ต้องติดตั้ง ดูแล สำรองข้อมูล และเฝ้าระวัง RabbitMQ Cluster บน Kubernetes
  - **Eventual Consistency:** ลูกค้าจะได้รับอีเมลและแต้มสะสมแบบ asynchronous (ภายในไม่กี่วินาทีถึง 1 นาที) แทนที่จะได้ในทันทีขณะกดสั่งซื้อ
- **Risks and Mitigations:**
  - *ความเสี่ยง:* ข้อความในคิวค้างสะสมจนแต้มสะสมเกินกรอบ 1 นาทีช่วงพีก 200 req/s
    *การแก้ไข:* Scale จำนวน replica ของ `loyaltyservice` consumer ให้เพียงพอ และตั้งค่า Alert แจ้งเตือนเมื่อ Queue Depth เกินค่าที่กำหนด
  - *ความเสี่ยง:* RabbitMQ โหนดดับ
    *การแก้ไข:* คอนฟิก RabbitMQ แบบ Quorum Queues (High Availability) กระจาย 3 โหนดบน Kubernetes และเปิดโหมด Durable Message

## Validation

1. **Load Testing:** ทดสอบยิงคำสั่งซื้อ 200 orders/s ด้วย `loadgenerator` จำลองโหลดช่วง 11.11 เป็นเวลา 30 นาที:
   - ตรวจสอบ p95 latency ของ `PlaceOrder` ต้อง $\le$ 500 ms
   - ตรวจสอบว่า MailFast ได้รับ request ไม่เกิน 50 req/s ตลอดเวลา
2. **Failure Injection Test:**
   - สั่งปิด MailFast จำลองสถานการณ์ล่ม 20 นาที: คำสั่งซื้อต้องสำเร็จตามปกติ และเมื่อ MailFast กลับมา ข้อความในคิวต้องถูกส่งออกจนครบ
   - ทดสอบส่ง message ซ้ำด้วย `order_id` เดิม ตรวจสอบว่าแต้มสะสมไม่เพิ่มขึ้นซ้ำ
3. **SLA Monitoring:** ตั้งค่า Prometheus & Grafana ตรวจวัดเวลาตั้งแต่คำสั่งซื้อสำเร็จจนกระทั่งได้แต้มสะสม (End-to-End latency) ต้องไม่เกิน 60 วินาทีที่ p99
