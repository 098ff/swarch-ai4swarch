Agent: AI assistant using Copilot SDK in VS Code; Model: ไม่ทราบ

# ADR-0002: ใช้ RabbitMQ สำหรับงานหลังวางคำสั่งซื้อ

- **Status:** Proposed
- **Date:** 2026-10-22
- **Deciders:** ยังไม่ระบุ

## Context and problem

ข้อกำหนด R-12 ระบุว่าเมื่อวางคำสั่งซื้อแล้ว ระบบต้องส่งอีเมลยืนยันผ่าน MailFast แทน logic จำลองของ `emailservice`, มอบ loyalty points ผ่าน `loyaltyservice` และบันทึก analytics event ให้ทีมการตลาด อีกทั้งคาดว่าจะเพิ่มงาน SMS, fraud scoring และอัปเดต recommendations ในปีหน้า (ข้อกำหนด R-12 ใน `docs/scenario.md`)

ปัจจุบัน `checkoutservice.PlaceOrder` เรียก `paymentservice` และ `shippingservice` ก่อนเรียก `emailservice` แบบ synchronous; หากส่งอีเมลล้มเหลวจะบันทึก warning แต่ยังตอบกลับว่าสั่งซื้อสำเร็จ (`PlaceOrder` และ `sendOrderConfirmation` ใน `system/online-boutique/src/checkoutservice/main.go`; รายละเอียดพฤติกรรมปัจจุบันและ R-12 ใน `docs/scenario.md`). ข้อกำหนดกำหนดให้ p95 ของ `PlaceOrder` ไม่เกิน 500 ms (ปัจจุบันประมาณ 300 ms) ขณะที่ MailFast มี p95 1.5 วินาที, จำกัด 50 อีเมล/วินาที และช่วง peak มี 200 คำสั่งซื้อ/วินาที นอกจากนี้ loyalty points ต้องได้รับภายใน 1 นาทีและห้ามให้คะแนนซ้ำ และระบบต้องไม่ทำให้คำสั่งซื้อล้มเหลวเมื่อบริการอีเมล, loyalty หรือ analytics ใช้งานไม่ได้ (ข้อเท็จจริง R-12 ใน `docs/scenario.md`).

โค้ดที่ให้มาสร้าง `orderID` ใน `PlaceOrder` แต่ไม่แสดงการบันทึกคำสั่งซื้อหรือ outbox แบบ durable (`PlaceOrder` ใน `system/online-boutique/src/checkoutservice/main.go`). จึงยังไม่มีหลักฐานว่ามีพื้นที่เก็บข้อมูลซึ่งรองรับการบันทึกคำสั่งซื้อและ event แบบ atomic อยู่แล้ว

## Decision drivers

1. **Availability:** ความล้มเหลวของ MailFast, `loyaltyservice` หรือ analytics ต้องไม่ทำให้คำสั่งซื้อล้มเหลว (R-12 ใน `docs/scenario.md`).
2. **Performance:** รักษา p95 ของ `PlaceOrder` ไม่เกิน 500 ms; การเรียก MailFast แบบ synchronous ที่มี p95 1.5 วินาทีไม่สอดคล้องกับเป้าหมายนี้ (R-12 ใน `docs/scenario.md`).
3. **Correctness:** ให้คะแนน loyalty ภายใน 1 นาทีและป้องกันการให้คะแนนซ้ำ (R-12 ใน `docs/scenario.md`).
4. **Scalability:** รองรับ 200 คำสั่งซื้อ/วินาที แม้ MailFast จำกัดไว้ที่ 50 อีเมล/วินาที (R-12 ใน `docs/scenario.md`).
5. **Modifiability และการปฏิบัติการ:** รองรับ post-order actions ใหม่ โดยใช้แนวทางที่เหมาะกับ Kubernetes และความชอบ open-source/self-hosted; ทีมมีนักพัฒนา 4 คนที่ถนัด Go/gRPC และ 2 คนมีประสบการณ์ RabbitMQ (R-12 ใน `docs/scenario.md`).

## Considered options

1. เรียก MailFast, `loyaltyservice` และ analytics แบบ synchronous จาก `checkoutservice`
2. ส่ง post-order event ผ่าน RabbitMQ และให้ consumer แยกกันประมวลผล
3. ส่ง event ผ่านบริการ post-order orchestration แบบ synchronous ซึ่งเรียกงานปลายทางพร้อมกัน

### Pros and cons of the options

| Option | Pros | Cons |
|---|---|---|
| เรียกปลายทางแบบ synchronous | โครงสร้างคล้ายการเรียก gRPC ที่ `checkoutservice` ใช้อยู่ในปัจจุบัน (`PlaceOrder` ใน `system/online-boutique/src/checkoutservice/main.go`); เข้าใจ flow ได้ตรงไปตรงมา | p95 ของ MailFast (1.5 วินาที) สูงกว่า budget ของ checkout (500 ms); การเรียกปลายทางเพิ่ม coupling และเพิ่มโอกาสที่ความล่าช้าจะกระทบ checkout; ต้องจัดการ rate limit 50 อีเมล/วินาที ขณะ peak 200 คำสั่งซื้อ/วินาที (R-12 ใน `docs/scenario.md`) |
| RabbitMQ และ consumers แยกตามงาน | แยกเวลาตอบ checkout ออกจากงานปลายทาง; queue ช่วยรองรับอัตราการสั่งซื้อที่สูงกว่า rate limit ของ MailFast; เพิ่ม consumer สำหรับงานใหม่ได้โดยไม่ต้องเพิ่มการเรียกแบบ synchronous ใน `PlaceOrder`; RabbitMQ เป็น open-source/self-hosted และมีสมาชิกทีมเคยใช้ (R-12 ใน `docs/scenario.md`) | เพิ่ม broker และภาระการดูแลบน Kubernetes; ต้องออกแบบ durable event publication, retry, rate limiting และ idempotency; ยังไม่พบ order store/outbox ในหลักฐานที่ให้มา จึงต้องกำหนดวิธีป้องกัน event สูญหายก่อนรับประกันการทำงานครบถ้วน |
| บริการ orchestration เรียกปลายทางพร้อมกัน | ลดเวลารวมเมื่อทุกปลายทางตอบเร็ว; ซ่อนรายละเอียดปลายทางจาก `checkoutservice` | ยังผูกผลสำเร็จและเวลาตอบของ flow เข้ากับปลายทาง; ไม่แก้ข้อจำกัด rate limit และการทำงานเมื่อปลายทางล่มได้ด้วยตัวเอง; เพิ่มบริการใหม่แต่ยังต้องมี retry/idempotency และกลไก durable ที่ไม่ได้ระบุในข้อกำหนด |

## Decision

เลือก **RabbitMQ พร้อม consumers แยกตาม post-order action** เพื่อให้ `checkoutservice` ไม่ต้องรอ MailFast, `loyaltyservice` หรือ analytics และเพื่อรองรับการเพิ่ม SMS, fraud scoring และ recommendations ในอนาคต แนวทางนี้สอดคล้องกับข้อจำกัด latency และ peak/rate limit มากกว่าการเรียกงานเหล่านั้นแบบ synchronous อีกทั้งสอดคล้องกับความชอบ self-hosted และประสบการณ์ RabbitMQ ของทีม (R-12 ใน `docs/scenario.md`).

ให้ event มี `orderID` ที่ไม่ซ้ำและใช้เป็น idempotency key เพื่อให้ consumer ของ loyalty ตรวจสอบและบันทึกการประมวลผลซ้ำอย่างปลอดภัย; ใช้ durable queue และ retry เพื่อให้ event ที่รอประมวลผลยังทำงานต่อได้เมื่อ consumer หรือผู้ให้บริการปลายทางกลับมา ทั้งหมดนี้เป็นข้อเสนอของ ADR ไม่ใช่ความสามารถที่ยืนยันว่ามีอยู่ในโค้ดปัจจุบัน (`orderID` มีการสร้างใน `PlaceOrder` แต่ยังไม่พบการเก็บ event แบบ durable ใน `system/online-boutique/src/checkoutservice/main.go`).

ก่อนถือว่าการตัดสินใจนี้รับประกันการทำงานครบถ้วน ต้องกำหนด durable outbox หรือกลไกเทียบเท่าที่ทำให้การยอมรับคำสั่งซื้อกับการบันทึก event เชื่อถือได้ โดยไม่ทำให้ความล้มเหลวของบริการปลายทางล้มคำสั่งซื้อ ข้อมูลที่ให้มายังไม่แสดง order store หรือวิธีทำให้การชำระเงิน, การจัดส่ง, การยอมรับคำสั่งซื้อ และ outbox สอดคล้องกัน (`PlaceOrder` ใน `system/online-boutique/src/checkoutservice/main.go`). การรับประกัน “exactly once” ให้หมายถึงผลของการให้คะแนนไม่ซ้ำด้วย idempotency ไม่ใช่การรับประกันว่า message ถูกส่งผ่าน broker เพียงครั้งเดียว

## Consequences

- **Positive:** การตอบ `PlaceOrder` ไม่ต้องรอ latency ของ MailFast (R-12 ใน `docs/scenario.md`; ปัจจุบันการส่ง confirmation เกิดหลังจัดส่งใน `PlaceOrder` ของ `system/online-boutique/src/checkoutservice/main.go`) (**คุณภาพ: performance**).
- **Positive:** queue ช่วยรับช่วงที่อัตราคำสั่งซื้อสูงกว่า rate limit ของ MailFast และ consumer สามารถจำกัดอัตราการเรียก provider ได้ (**คุณภาพ: scalability และ availability**; ตัวเลข 200 orders/s และ 50 emails/s มาจาก R-12 ใน `docs/scenario.md`).
- **Positive:** consumers แยกกันทำให้งาน post-order ใหม่เพิ่มได้โดยไม่เพิ่ม synchronous dependency ใน checkout (**คุณภาพ: modifiability**; งานในอนาคตตาม R-12 ใน `docs/scenario.md`).
- **Negative:** RabbitMQ และ durable event storage เพิ่มส่วนประกอบที่ทีมต้อง deploy, monitor และดูแลบน Kubernetes (**คุณภาพ: operational simplicity และ cost**; สภาพแวดล้อมและความชอบ self-hosted ตาม R-12 ใน `docs/scenario.md`).
- **Negative:** การประมวลผลแบบ asynchronous หมายความว่าอีเมลหรือ analytics อาจล่าช้า และต้องติดตามอายุของ event ค้างอยู่ (**คุณภาพ: timeliness/observability**).
- **Risks and mitigations:** message ซ้ำอาจให้ loyalty points ซ้ำได้; consumer ต้อง deduplicate ด้วย `orderID` และบันทึกผลให้คงทนก่อนยืนยันการประมวลผล (**คุณภาพ: correctness**). ตรวจวัดว่า points ถูกมอบภายใน 1 นาทีและไม่มีการให้ซ้ำตามเกณฑ์ R-12 (`docs/scenario.md`).
- **Risks and mitigations:** หาก checkout ตอบว่าสำเร็จแต่ event ยังไม่ถูกบันทึก durable งานปลายทางอาจสูญหาย; ต้องเลือกและทดสอบกลไก outbox หรือเทียบเท่าก่อนเปิดใช้จริง เพราะโค้ดที่ให้มายังไม่แสดง order store/outbox (`PlaceOrder` ใน `system/online-boutique/src/checkoutservice/main.go`) (**คุณภาพ: reliability**).
- **Risks and mitigations:** MailFast อาจถูกเรียกเกิน 50 อีเมล/วินาที; จำกัด concurrency/rate ที่ consumer และติดตาม queue backlog พร้อมอัตราการส่ง (**คุณภาพ: compliance และ availability**; limit ตาม R-12 ใน `docs/scenario.md`).

## Validation

- ทดสอบโหลด `PlaceOrder` ที่ 200 orders/s และยืนยัน p95 ไม่เกิน 500 ms ทั้งเมื่อ consumers ทำงานปกติและเมื่อ MailFast, `loyaltyservice` หรือ analytics หยุดทำงาน (R-12 ใน `docs/scenario.md`).
- จำลอง MailFast ที่ p95 1.5 วินาที, p99 8 วินาที, failure 0.5% และ rate limit 50 emails/s; ตรวจว่า checkout ยังคงตอบตามเป้าหมายและ consumer ไม่ส่งเกิน limit (R-12 ใน `docs/scenario.md`).
- ทดสอบการ retry และส่ง message ซ้ำ พร้อมยืนยันว่า loyalty points ถูกมอบภายใน 1 นาทีและไม่เกิดผลซ้ำ (R-12 ใน `docs/scenario.md`).
- ทดสอบการ restart broker/consumer และการกู้คืนจาก backlog; ยืนยันว่า event ไม่สูญหายด้วยกลไก durable ที่เลือก และติดตามเวลาใน queue สำหรับทุก action.

## Open questions

- จะใช้ datastore ใดสำหรับ order/outbox และจะบันทึกคำสั่งซื้อกับ event อย่าง durable/atomic อย่างไร? หลักฐานที่ให้มายังไม่แสดง order store หรือ outbox (`PlaceOrder` ใน `system/online-boutique/src/checkoutservice/main.go`).
- หากการบันทึก durable event ล้มเหลว จะจัดการการตอบคำสั่งซื้ออย่างไรให้สอดคล้องทั้งข้อกำหนด “คำสั่งซื้อต้องไม่ล้มเหลวเพราะบริการปลายทางล่ม” และการไม่สูญเสีย post-order action?
- จะออกแบบ schema/versioning ของ event, retry policy, dead-letter handling และการแจ้งเตือนเมื่อใกล้เกิน SLA 1 นาทีอย่างไร?
- loyalty service จะจัดเก็บ idempotency key และผลการให้คะแนนแบบใด เพื่อป้องกันการให้ซ้ำแม้ consumer ทำงานซ้ำหรือ restart?
- MailFast ใช้ API/protocol และวิธี authentication ใด และต้องจัดการข้อมูลส่วนบุคคล/ความลับของผู้รับอย่างไร? Not evidenced in the provided files.
- ต้องบันทึก analytics event ที่ใดและด้วย protocol ใด? Not evidenced in the provided files.
- ใครเป็นผู้ตัดสินใจและรับผิดชอบการดำเนินงาน RabbitMQ และ outbox? จำนวนทีมและประสบการณ์ RabbitMQ มีระบุ แต่ผู้รับผิดชอบไม่ได้ระบุ (R-12 ใน `docs/scenario.md`).
