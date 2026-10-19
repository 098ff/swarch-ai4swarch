# Task 3: Get the AI's review · งานที่ 3: ให้ AI รีวิว ADR (≈ 5 min · นาที)

> **Lecture link:** P5 human–AI collaboration: the AI as *Chief Analyst*. A 2026 study found LLMs often matched or beat student teams at spotting ATAM risks and trade-offs ([arXiv 2603.28914](https://arxiv.org/abs/2603.28914)). Part B tests that claim.
>
> **เชื่อมโยงกับการบรรยาย:** P5 การทำงานร่วมกันระหว่างมนุษย์กับ AI โดย AI รับบทเป็น *Chief Analyst* งานวิจัยปี 2026 พบว่า LLM มักทำได้เท่าหรือดีกว่าทีมนักศึกษาในการหาความเสี่ยงและ trade-off แบบ ATAM ([arXiv 2603.28914](https://arxiv.org/abs/2603.28914)) ส่วน B จะทดสอบข้ออ้างนี้

Start a new chat or session and send, with the path of **your final ADR**:

เริ่มแชตหรือ session ใหม่ แล้วส่งข้อความด้านล่าง โดยใส่ path ของ **ADR ฉบับสุดท้ายของนิสิต**

```
Follow prompts/3-review-adr.md
adr: docs/adr/0002-<your-title>.md
```

The agent writes a one-page ATAM-style review to [`outputs/3-review.md`](outputs/3-review.md). It must not edit your ADR; if it does, reject or undo the change.

agent จะเขียนผลรีวิวแบบ ATAM ความยาวหนึ่งหน้าไว้ที่ [`outputs/3-review.md`](outputs/3-review.md) agent ต้องไม่แก้ไข ADR ของนิสิต หากแก้ไข ให้ปฏิเสธหรือ undo การเปลี่ยนแปลงนั้น

**Don't act on the review yet.** Read it, commit it, and bring it to the review board in Part B. The board will decide which of its findings matter.

**ยังไม่ต้องแก้ไขตามผลรีวิว** ให้อ่าน commit และนำไปใช้ในที่ประชุมคณะกรรมการรีวิวในส่วน B คณะกรรมการจะเป็นผู้ตัดสินว่าข้อค้นพบใดสำคัญ

- **My first impression of the AI's review** (one sentence) · **ความประทับใจแรกต่อผลรีวิวของ AI** (หนึ่งประโยค): …
