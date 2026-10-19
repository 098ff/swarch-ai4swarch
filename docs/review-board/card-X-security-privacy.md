# Role card X: Security & privacy officer (groups of 5) · การ์ดบทบาท X: เจ้าหน้าที่ความปลอดภัยและความเป็นส่วนตัว (กลุ่ม 5 คน)

> Keep this card to yourself until the meeting. Raise these points in your own words; you don't have to use all of them.
>
> เก็บการ์ดนี้เป็นความลับจนถึงเวลาประชุม ยกประเด็นเหล่านี้ด้วยคำพูดของตัวเอง ไม่จำเป็นต้องใช้ทุกข้อ

**You care about:** customer data, legal compliance, and the company's reputation.
**สิ่งที่คุณให้ความสำคัญ:** ข้อมูลลูกค้า การปฏิบัติตามกฎหมาย และชื่อเสียงของบริษัท

**Facts only you know · ข้อเท็จจริงที่มีเพียงคุณที่รู้**
- MailFast stores data in **US data centres**. Thailand's PDPA requires a legal basis and adequate safeguards for transferring customers' personal data abroad; the legal team has **not yet approved** MailFast.
  MailFast เก็บข้อมูลใน**ศูนย์ข้อมูลที่สหรัฐอเมริกา** พ.ร.บ.คุ้มครองข้อมูลส่วนบุคคล (PDPA) ของไทยกำหนดให้การส่งข้อมูลส่วนบุคคลของลูกค้าไปต่างประเทศต้องมีฐานทางกฎหมายและมาตรการคุ้มครองที่เพียงพอ ซึ่งฝ่ายกฎหมาย**ยังไม่ได้อนุมัติ** MailFast
- The analytics team wants the "order placed" event, but under the company's policy **analytics may not receive email addresses or full names**.
  ทีม analytics ต้องการเหตุการณ์ "มีการสั่งซื้อ" แต่ตามนโยบายบริษัท **ระบบ analytics ห้ามได้รับอีเมลหรือชื่อ-นามสกุลของลูกค้า**
- Any queue or log that stores email addresses or addresses counts as personal data: it must be **encrypted at rest** and have a **retention limit** (max 30 days).
  คิวหรือ log ใดที่เก็บอีเมลหรือที่อยู่ถือเป็นข้อมูลส่วนบุคคล ต้อง**เข้ารหัสขณะจัดเก็บ (encrypted at rest)** และมี**ระยะเวลาเก็บรักษาจำกัด** (ไม่เกิน 30 วัน)

**Questions to ask the architect · คำถามสำหรับสถาปนิก**
- What personal data goes into each message, and who can read the queue?
  แต่ละข้อความมีข้อมูลส่วนบุคคลอะไรบ้าง และใครอ่านคิวได้บ้าง
- Does the analytics event contain anything that identifies a customer?
  เหตุการณ์ analytics มีข้อมูลใดที่ระบุตัวลูกค้าได้หรือไม่
- What is the fallback if legal does not approve MailFast before 11.11?
  ถ้าฝ่ายกฎหมายไม่อนุมัติ MailFast ก่อน 11.11 จะมีแผนสำรองอย่างไร
