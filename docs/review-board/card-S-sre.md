# Role card S: Site reliability engineer (Operations) · การ์ดบทบาท S: Site reliability engineer (ฝ่ายปฏิบัติการ)

> Keep this card to yourself until the meeting. Raise these points in your own words; you don't have to use all of them.
>
> เก็บการ์ดนี้เป็นความลับจนถึงเวลาประชุม ยกประเด็นเหล่านี้ด้วยคำพูดของตัวเอง ไม่จำเป็นต้องใช้ทุกข้อ

**You care about:** keeping the system running at 3 a.m., on-call load, and cost.
**สิ่งที่คุณให้ความสำคัญ:** ทำให้ระบบทำงานได้แม้ตอนตีสาม ภาระงานเวร on-call และค่าใช้จ่าย

**Facts only you know · ข้อเท็จจริงที่มีเพียงคุณที่รู้**
- The two developers who "used RabbitMQ before" used it **only on their laptops**. Nobody has run a message broker in production here. The on-call rotation is **2 people**.
  นักพัฒนาสองคนที่ "เคยใช้ RabbitMQ" เคยใช้**แค่บนโน้ตบุ๊กของตัวเอง** ยังไม่มีใครในบริษัทเคยดูแล message broker บน production และเวร on-call มีเพียง **2 คน**
- Operations has a rule: **no new stateful component in production without** monitoring dashboards, alerting, backups and a written runbook. Writing these takes about **1 week**.
  ฝ่ายปฏิบัติการมีกฎว่า**ห้ามนำ component ที่เก็บ state ตัวใหม่ขึ้น production หากไม่มี** dashboard สำหรับ monitoring, การแจ้งเตือน, การสำรองข้อมูล และ runbook ที่เขียนไว้ ซึ่งต้องใช้เวลาเตรียมประมาณ **1 สัปดาห์**
- Last year's 11.11 sale had a **20-minute outage** of the email provider; nobody noticed until customers complained on social media.
  เทศกาล 11.11 ปีที่แล้ว ผู้ให้บริการอีเมล**ล่มไป 20 นาที** และไม่มีใครรู้จนกระทั่งลูกค้าบ่นในโซเชียลมีเดีย
- The Kubernetes cluster is near its node limit; a highly available broker (3 nodes) needs budget approval, which takes 2 weeks.
  Kubernetes cluster ใกล้ถึงขีดจำกัดจำนวน node แล้ว การติดตั้ง broker แบบ high availability (3 node) ต้องขออนุมัติงบประมาณ ซึ่งใช้เวลา 2 สัปดาห์

**Questions to ask the architect · คำถามสำหรับสถาปนิก**
- Who gets paged when the broker or a consumer is down, and how do they know?
  เมื่อ broker หรือ consumer ล่ม ใครจะถูกเรียก และเขาจะรู้ได้อย่างไร
- What happens to messages stuck in the queue during an outage, and how do we replay them? Is there a dead-letter queue?
  ข้อความที่ค้างอยู่ในคิวระหว่างระบบล่มจะเป็นอย่างไร และจะส่งซ้ำได้อย่างไร มี dead-letter queue หรือไม่
- How will we trace one order across the asynchronous boundary when a customer says "I never got my email"?
  ถ้าลูกค้าบอกว่า "ไม่เคยได้รับอีเมลเลย" เราจะติดตามคำสั่งซื้อหนึ่งรายการข้ามขอบเขตแบบ asynchronous ได้อย่างไร
