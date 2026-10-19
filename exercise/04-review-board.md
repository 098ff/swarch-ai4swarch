# Part B: Architecture review board · ส่วน B: คณะกรรมการรีวิวสถาปัตยกรรม (≈ 25 min · นาที, groups of 4 · กลุ่มละ 4 คน)

> **Lecture link:** AICH4 (reviews need domain expertise; *plausible ≠ appropriate*), Pillar 5 (AI as Chief Analyst, humans as Chief Strategist), JSS (85% of studies keep a human in the loop).
> **Question:** what does a human review board catch that the AI's review missed, and why?
>
> **เชื่อมโยงกับการบรรยาย:** AICH4 (การรีวิวต้องอาศัยความเชี่ยวชาญในโดเมน; *ฟังดูดี ≠ เหมาะสม*), Pillar 5 (AI เป็น Chief Analyst มนุษย์เป็น Chief Strategist), JSS (85% ของงานวิจัยให้มนุษย์มีส่วนร่วมในกระบวนการ)
> **คำถาม:** คณะกรรมการที่เป็นมนุษย์จับประเด็นใดได้ที่ผลรีวิวของ AI พลาดไป และเพราะเหตุใด

The company's architecture review board meets to approve the R-12 design before the 11.11 sale. One ADR goes to the board: the **architect** presents it, the **AI analyst** presents the AI's review, and two stakeholders bring knowledge the AI never saw.

คณะกรรมการรีวิวสถาปัตยกรรมของบริษัทประชุมเพื่ออนุมัติการออกแบบ R-12 ก่อนเทศกาลลดราคา 11.11 โดยนำ ADR หนึ่งฉบับเข้าที่ประชุม **สถาปนิก**นำเสนอ ADR, **AI analyst** นำเสนอผลรีวิวของ AI และผู้มีส่วนได้ส่วนเสียสองคนนำข้อมูลที่ AI ไม่เคยเห็นมาด้วย

## 1. Form the board · จัดตั้งคณะกรรมการ (2 min · นาที)

In groups of 4, choose **one** member's ADR to review (`docs/adr/0002-*.md`, plus their `exercise/outputs/3-review.md`). Then assign roles:

ในกลุ่มละ 4 คน ให้เลือก ADR ของสมาชิก**หนึ่งคน**มารีวิว (`docs/adr/0002-*.md` พร้อม `exercise/outputs/3-review.md` ของคนนั้น) แล้วแบ่งบทบาท

| Role · บทบาท | Who · ใคร | Brings · สิ่งที่นำมา |
|---|---|---|
| **Architect · สถาปนิก** | the ADR's author · ผู้เขียน ADR | defends the ADR · ปกป้อง ADR |
| **AI analyst** | | reads out the AI's review only, and may not add their own opinions · อ่านผลรีวิวของ AI เท่านั้น ห้ามใส่ความเห็นของตัวเอง |
| **Product owner** | | role card P (business side) · การ์ดบทบาท P (ฝั่งธุรกิจ) |
| **Site reliability engineer (SRE)** | | role card S (operations side) · การ์ดบทบาท S (ฝั่งปฏิบัติการ) |

Groups of 5 add a **security & privacy officer** (role card X). In groups of 3, the architect also plays the AI analyst.

กลุ่มละ 5 คน ให้เพิ่ม**เจ้าหน้าที่ความปลอดภัยและความเป็นส่วนตัว** (การ์ดบทบาท X) กลุ่มละ 3 คน ให้สถาปนิกรับบท AI analyst ด้วย

**Role cards** are in [`docs/review-board/`](../docs/review-board/). Each stakeholder reads **only their own card** and keeps it secret until the meeting.

**การ์ดบทบาท**อยู่ใน [`docs/review-board/`](../docs/review-board/) ผู้มีส่วนได้ส่วนเสียแต่ละคนอ่าน**เฉพาะการ์ดของตัวเอง** และเก็บเป็นความลับจนถึงเวลาประชุม

## 2. Meeting · การประชุม (15 min · นาที)

| Min · นาที | Step · ขั้นตอน |
|---|---|
| 3 | **Architect** presents the decision, the options considered and the main consequences.<br>**สถาปนิก**นำเสนอการตัดสินใจ ทางเลือกที่พิจารณา และผลที่ตามมาหลัก ๆ |
| 3 | **AI analyst** reads the AI review's findings and verdict, word for word.<br>**AI analyst** อ่านข้อค้นพบและคำตัดสินของ AI แบบคำต่อคำ |
| 7 | **Open discussion.** Each stakeholder raises the concerns from their card, in their own words. The architect responds; anyone may ask the AI analyst "did the AI see this?"<br>**อภิปรายเปิด** ผู้มีส่วนได้ส่วนเสียแต่ละคนยกข้อกังวลจากการ์ดของตัวเองด้วยคำพูดของตัวเอง สถาปนิกตอบ และทุกคนถาม AI analyst ได้ว่า "AI เห็นประเด็นนี้หรือไม่" |
| 2 | **Decision.** The board votes: accept / accept with changes / reject.<br>**ตัดสินใจ** คณะกรรมการลงมติ: อนุมัติ / อนุมัติโดยมีเงื่อนไขให้แก้ไข / ไม่อนุมัติ |

## 3. Record the decision · บันทึกผลการตัดสินใจ (8 min · นาที, together · ทำร่วมกัน)

**ADR reviewed · ADR ที่รีวิว:** `docs/adr/0002-…` by · โดย …
**Board members and roles · สมาชิกและบทบาท:** …

**Decision · มติ:** accept / accept with changes / reject · อนุมัติ / อนุมัติโดยมีเงื่อนไข / ไม่อนุมัติ

**Required changes · สิ่งที่ต้องแก้ไข:**
1. …
2. …
3. …

**Who raised what? · ใครยกประเด็นใด** List every concern the board discussed. · ระบุทุกข้อกังวลที่คณะกรรมการอภิปราย

| # | Concern<br>ข้อกังวล | Raised by (AI review / architect / PO / SRE / security)<br>ผู้ยกประเด็น | Was it in the AI's review?<br>อยู่ในผลรีวิวของ AI หรือไม่ | Could the AI have known? Why or why not?<br>AI มีทางรู้ได้หรือไม่ เพราะเหตุใด |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

**The AI's best finding** (one the humans had missed) · **ข้อค้นพบที่ดีที่สุดของ AI** (ที่มนุษย์มองข้ามไป): …
**The AI's weakest finding** (generic, wrong or irrelevant) · **ข้อค้นพบที่อ่อนที่สุดของ AI** (กว้างเกินไป ผิด หรือไม่เกี่ยวข้อง): …

## 4. Individual reflection · สะท้อนคิดรายบุคคล (≤ 150 words · คำ, each student · ทุกคน, after class is fine · ทำหลังคาบได้)

Write in [`05-reflection.md`](05-reflection.md): after Part A and Part B, which parts of an architect's work would you trust AI with, and which would you keep for humans? Use one concrete example from your board meeting.

เขียนใน [`05-reflection.md`](05-reflection.md): หลังจากทำส่วน A และส่วน B แล้ว นิสิตจะไว้วางใจให้ AI ทำงานส่วนใดของสถาปนิก และจะเก็บงานส่วนใดไว้ให้มนุษย์ ยกตัวอย่างที่เป็นรูปธรรมหนึ่งตัวอย่างจากการประชุมคณะกรรมการ

> Each group member commits this record to their own repository (copy the same text), so everyone submits a complete repo.
>
> สมาชิกทุกคนในกลุ่ม commit บันทึกนี้ลงใน repository ของตัวเอง (คัดลอกข้อความเดียวกัน) เพื่อให้ทุกคนส่ง repository ที่ครบถ้วน
