# Scenario: what happens after an order is placed · สถานการณ์: สิ่งที่เกิดขึ้นหลังจากลูกค้าสั่งซื้อ

> A **fictional** business scenario for Task 2. The numbers are made up but internally consistent. Treat them as facts.
>
> สถานการณ์ทางธุรกิจ**สมมติ**สำหรับงานที่ 2 ตัวเลขเป็นตัวเลขสมมติแต่สอดคล้องกันภายใน ให้ถือว่าเป็นข้อเท็จจริง

## The new requirement (R-12) · ความต้องการใหม่ (R-12)

> After an order is placed, Online Boutique must:
> 1. send the order confirmation email through **MailFast**, a new external email provider, which replaces the mock `emailservice` logic;
> 2. award **loyalty points** through a new `loyaltyservice` (to be built);
> 3. record an **"order placed" analytics event** for the marketing team.
>
> Marketing expects **more post-order actions next year** (SMS, fraud scoring, a recommendations update).

ภาษาไทย (สำหรับอ่านเท่านั้น ให้วางข้อความภาษาอังกฤษด้านบนลงในข้อความถึง agent): หลังจากลูกค้าสั่งซื้อ Online Boutique ต้อง
1. ส่งอีเมลยืนยันคำสั่งซื้อผ่าน **MailFast** ซึ่งเป็นผู้ให้บริการอีเมลภายนอกรายใหม่ แทนที่ logic จำลองใน `emailservice`
2. มอบ**แต้มสะสม (loyalty points)** ผ่าน `loyaltyservice` ใหม่ (ยังต้องสร้าง)
3. บันทึก**เหตุการณ์ analytics "มีการสั่งซื้อ"** ให้ทีมการตลาด

ทีมการตลาดคาดว่า**ปีหน้าจะมีงานหลังการสั่งซื้อเพิ่มขึ้นอีก** (SMS, การให้คะแนนความเสี่ยงการทุจริต, การอัปเดตระบบแนะนำสินค้า)

## How it works today · ระบบทำงานอย่างไรในปัจจุบัน

`checkoutservice.PlaceOrder` calls `paymentservice`, `shippingservice` and then `emailservice` **synchronously over gRPC**, in sequence. If the email call fails, checkout only logs a warning, and the order still succeeds.

`checkoutservice.PlaceOrder` เรียก `paymentservice`, `shippingservice` แล้วจึงเรียก `emailservice` **แบบ synchronous ผ่าน gRPC** ตามลำดับ หากการเรียกอีเมลล้มเหลว checkout จะบันทึกแค่คำเตือน (warning) และคำสั่งซื้อยังสำเร็จตามปกติ

## Quality goals and constraints · เป้าหมายด้านคุณภาพและข้อจำกัด

| Topic · หัวข้อ | Fact | ข้อเท็จจริง |
|---|---|---|
| Checkout latency | p95 of `PlaceOrder` must stay **≤ 500 ms** (today ≈ 300 ms) | p95 ของ `PlaceOrder` ต้องไม่เกิน **500 ms** (ปัจจุบัน ≈ 300 ms) |
| Peak load | **200 orders/s** during the 11.11 sale | **200 คำสั่งซื้อ/วินาที** ช่วงเทศกาล 11.11 |
| MailFast | p95 **1.5 s**, p99 **8 s**; fails ≈ **0.5 %** of calls; rate limit **50 emails/s** | p95 **1.5 วินาที**, p99 **8 วินาที**; ล้มเหลว ≈ **0.5%** ของการเรียก; จำกัดที่ **50 อีเมล/วินาที** |
| Loyalty points | must be awarded **within 1 minute** of the order, **exactly once** (no double points) | ต้องให้แต้ม**ภายใน 1 นาที**หลังสั่งซื้อ และ**ครั้งเดียวเท่านั้น** (ห้ามได้แต้มซ้ำ) |
| Availability | an order must **never fail** because email, loyalty or analytics is down | คำสั่งซื้อต้อง**ไม่ล้มเหลว**เพราะระบบอีเมล แต้มสะสม หรือ analytics ล่ม |
| Team | 4 developers; strong in Go and gRPC; two have used **RabbitMQ** before | นักพัฒนา 4 คน ถนัด Go และ gRPC; สองคนเคยใช้ **RabbitMQ** |
| Operations | everything already runs on Kubernetes; the company prefers **open-source, self-hosted** components over cloud-only managed services | ทุกอย่างรันบน Kubernetes อยู่แล้ว บริษัทเลือกใช้ component แบบ **open-source ที่ติดตั้งเอง** มากกว่าบริการ managed ที่มีเฉพาะบนคลาวด์ |
