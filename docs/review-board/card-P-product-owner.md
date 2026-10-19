# Role card P: Product owner (Marketing & Loyalty) · การ์ดบทบาท P: Product owner (การตลาดและแต้มสะสม)

> Keep this card to yourself until the meeting. Raise these points in your own words; you don't have to use all of them.
>
> เก็บการ์ดนี้เป็นความลับจนถึงเวลาประชุม ยกประเด็นเหล่านี้ด้วยคำพูดของตัวเอง ไม่จำเป็นต้องใช้ทุกข้อ

**You care about:** customers, the 11.11 campaign, and what marketing has already promised.
**สิ่งที่คุณให้ความสำคัญ:** ลูกค้า แคมเปญ 11.11 และสิ่งที่ฝ่ายการตลาดได้สัญญาไว้แล้ว

**Facts only you know · ข้อเท็จจริงที่มีเพียงคุณที่รู้**
- Marketing has already printed and launched the 11.11 campaign: *"See your loyalty points **the moment** you place your order."* The order confirmation page must show the new points total immediately.
  ฝ่ายการตลาดพิมพ์และเปิดตัวแคมเปญ 11.11 ไปแล้ว: *"เห็นแต้มสะสม**ทันที**ที่สั่งซื้อ"* หน้ายืนยันคำสั่งซื้อจึงต้องแสดงยอดแต้มใหม่ทันที
- Loyalty points are worth real money (100 points = 10 THB discount). A customer who gets double points is a financial loss; a customer who gets none files a complaint.
  แต้มสะสมมีมูลค่าเป็นเงินจริง (100 แต้ม = ส่วนลด 10 บาท) ลูกค้าที่ได้แต้มซ้ำทำให้บริษัทขาดทุน ลูกค้าที่ไม่ได้แต้มจะร้องเรียน
- The 11.11 sale is in **3 weeks**. Anything that can't be ready, tested and load-tested by then is a problem.
  เทศกาล 11.11 จะมาถึงใน**อีก 3 สัปดาห์** สิ่งใดที่พัฒนา ทดสอบ และทำ load test ไม่ทันถือเป็นปัญหา
- Next year's plans include SMS and fraud scoring, but **fraud scoring must be able to cancel an order before it is confirmed.**
  แผนปีหน้ามี SMS และการให้คะแนนความเสี่ยงการทุจริต แต่**ระบบตรวจการทุจริตต้องยกเลิกคำสั่งซื้อได้ก่อนที่คำสั่งซื้อจะได้รับการยืนยัน**

**Questions to ask the architect · คำถามสำหรับสถาปนิก**
- If points are added "within a minute", what does the customer see on the confirmation page?
  ถ้าแต้มจะถูกเพิ่ม "ภายในหนึ่งนาที" ลูกค้าจะเห็นอะไรบนหน้ายืนยันคำสั่งซื้อ
- What happens to the points if the order is later cancelled or refunded?
  ถ้าภายหลังมีการยกเลิกคำสั่งซื้อหรือคืนเงิน แต้มจะเป็นอย่างไร
- Can this be built and load-tested in 3 weeks?
  สร้างและทำ load test ได้ทันใน 3 สัปดาห์หรือไม่
