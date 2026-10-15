**ภาษา:** ภาษาไทย | [English](README.en.md)

# AI for Software Architecture: แบบฝึกหัดลงมือทำ

**2110521 Software Architecture · สัปดาห์ที่ 12 · Tutorial 5 / Assignment 5**

ในแบบฝึกหัดนี้ นิสิตจะใช้ **GitHub Copilot ใน VS Code** เป็นผู้ช่วยด้านสถาปัตยกรรมซอฟต์แวร์ กับระบบ microservices จริงชื่อ [Online Boutique](system/online-boutique/) แล้วประเมินว่างานที่ AI ทำออกมานั้นดีแค่ไหน เป้าหมายไม่ใช่ให้ AI ทำงานแทน แต่เพื่อค้นหาว่า **AI ช่วยสถาปนิกซอฟต์แวร์ได้ตรงไหน และตรงไหนที่ยังเชื่อถือไม่ได้** ซึ่งเป็นข้อสรุปหลักของการบรรยายสัปดาห์นี้

นิสิตจะได้:

1. **กู้คืน (recover) สถาปัตยกรรม** จากซอร์สโค้ด (หัวข้อ T7: architecture recovery)
2. **กู้คืนการตัดสินใจ** ที่ไม่เคยถูกบันทึกไว้ (T5: decision recovery, knowledge vaporization)
3. **ตัดสินใจเรื่องใหม่** ด้วย ADR ที่ AI ช่วยเขียน ทั้งแบบมีและไม่มีบริบท (T3: ADR generation)
4. **รีวิว** ADR แบบเดียวกับคณะกรรมการสถาปัตยกรรม แล้วรับผิดชอบฉบับสุดท้ายด้วยตัวเอง (AICH4, "Chief Strategist")

เวลาที่ใช้โดยประมาณ: ราว 75 นาทีในคาบเรียน

---

## ก่อนเข้าเรียน: เตรียมเครื่อง (≈ 15 นาที ควรทำล่วงหน้า)

1. **สมัคร Copilot**
   - นิสิตที่ยืนยันตัวตนแล้วใช้ Copilot ได้ฟรีผ่าน [GitHub Education](https://docs.github.com/en/copilot/how-tos/copilot-on-github/set-up-copilot/enable-copilot/set-up-for-students) แต่การอนุมัติและเปิดใช้งาน **อาจใช้เวลาหลายวัน** จึงควรสมัครตั้งแต่ตอนนี้
   - หากยังไม่ได้รับสิทธิ์ก่อนเข้าเรียน **Copilot Free** ก็เพียงพอสำหรับแบบฝึกหัดนี้ (ดู [โควตาการใช้ Copilot](#โควตาการใช้-copilot))
2. **ติดตั้ง [VS Code](https://code.visualstudio.com/)** และลงชื่อเข้าใช้ GitHub ใน VS Code (เมนู Accounts มุมล่างซ้าย)
3. **สร้างสำเนา repository นี้เป็นของตัวเอง**
   - กด **Use this template → Create a new repository** (ตั้งเป็น private ได้)
   - clone ลงเครื่อง แล้วเปิดโฟลเดอร์ใน VS Code
4. **ติดตั้ง extension ที่แนะนำ** เมื่อ VS Code ถาม (รายการอยู่ใน [`.vscode/extensions.json`](.vscode/extensions.json))
   - **GitHub Copilot Chat**
   - **Markdown Preview Mermaid Support** สำหรับแสดงแผนภาพที่นิสิตจะสร้าง
5. **ตรวจสอบว่าใช้งานได้**
   1. เปิดหน้าต่าง Chat (`Ctrl+Alt+I` / `⌃⌘I`)
   2. เลือก **Ask** ที่ตัวเลือก agent
   3. พิมพ์ `/` แล้วตรวจว่ามีคำสั่งทั้งสี่: `/recover-architecture`, `/recover-decision`, `/write-adr`, `/review-adr`

> ทุกคนใช้ prompt ชุดเดียวกัน ซึ่งอยู่ใน [`.github/prompts/`](.github/prompts/) ส่วนกติกาที่ Copilot ใช้ร่วมกันอยู่ใน [`.github/copilot-instructions.md`](.github/copilot-instructions.md) ควรอ่านทั้งสองไฟล์ เพราะการรู้ว่า AI ได้รับคำสั่งอะไรเป็นส่วนหนึ่งของการประเมินสิ่งที่ AI ตอบ

## โควตาการใช้ Copilot

แบบฝึกหัดทั้งหมดใช้ประมาณ **10–12 คำขอ (chat requests)** ซึ่งอยู่ในโควตาของ Copilot Free

- ใช้โหมด **Ask** ไม่ใช่โหมด Agent เพราะโหมด Ask ตอบจากไฟล์ที่แนบเท่านั้น ไม่ไล่อ่านทั้ง repository จึงเร็วกว่าและใช้เครดิตน้อยกว่า
- ตั้งโมเดลเป็น **Auto** ไว้ เว้นแต่ผู้สอนจะบอกเป็นอย่างอื่น
- **แนบบริบทอย่างตั้งใจ** โดยใช้ **Add Context** (📎) หรือลากไฟล์/โฟลเดอร์มาวางในแชต การเลือกว่าจะแนบไฟล์ใดเป็นประเด็นหนึ่งที่แบบฝึกหัดนี้ต้องการให้สังเกต

## กติกา

- **บันทึกทุก prompt** ไว้ใน [`exercise/00-prompt-log.md`](exercise/00-prompt-log.md) ได้แก่ ถามอะไร แนบไฟล์อะไร และได้คำตอบอะไร
- **ตรวจสอบคำกล่าวอ้างกับโค้ดจริง** คำกล่าวอ้างที่ไม่มี path ของไฟล์เป็นหลักฐาน ถือเป็นการเดาจนกว่านิสิตจะตรวจสอบเอง
- **อย่าวางคำตอบของ AI เป็นข้อสรุปของตัวเอง** โดยไม่ระบุที่มา ในใบงานแยกช่องสำหรับ *คำตอบของ AI* กับ *ความเห็นของนิสิต* ไว้แล้ว

## งานที่ต้องทำ

| # | ใบงาน | Prompt | จำนวนคำขอ |
|---|---|---|---|
| 1 | [กู้คืนสถาปัตยกรรม](exercise/01-recover-architecture.md) | `/recover-architecture` | 2–3 |
| 2 | [กู้คืนการตัดสินใจที่ไม่ได้บันทึกไว้](exercise/02-recover-decision.md) | `/recover-decision` | 1–2 |
| 3 | [เขียน ADR สำหรับความต้องการใหม่](exercise/03-write-adr.md) | `/write-adr` | 2–3 |
| 4 | [รีวิวและรับผิดชอบฉบับสุดท้าย](exercise/04-review-adr.md) | `/review-adr` | 1–2 |

> ใบงานเขียนเป็นภาษาอังกฤษ นิสิตตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## การส่งงาน (Assignment 5 ส่งภายในท้ายคาบ)

1. กรอกใบงานทั้งหมดในโฟลเดอร์ `exercise/` และเขียน ADR ฉบับสุดท้ายไว้ที่ `docs/adr/0002-*.md`
2. commit และ push ขึ้น repository ของตัวเอง
3. ส่ง URL ของ repository ใน MCV หาก repository เป็น private ให้เพิ่มผู้สอนเป็น collaborator

## โครงสร้างของ repository

```
.github/
  copilot-instructions.md   กติกาของ Copilot ที่ใช้ร่วมกันใน repo นี้
  prompts/                  prompt สี่คำสั่ง (slash commands)
docs/
  adr/template.md           รูปแบบ ADR ที่ใช้ในวิชา
  adr/0001-…                ตัวอย่าง ADR (ใช้ในงานที่ 3)
  scenario.md               บริบททางธุรกิจสำหรับงานที่ 3
exercise/                   ใบงาน (นิสิตแก้ไขไฟล์เหล่านี้)
system/online-boutique/     ระบบที่ใช้วิเคราะห์ (ฉบับตัดทอน)
```

## ที่มาและลิขสิทธิ์

ระบบที่ใช้ศึกษาคือ [Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo) ของ Google ภายใต้ Apache License 2.0 ดูสิ่งที่เปลี่ยนแปลงสำหรับแบบฝึกหัดนี้ได้ที่ [`system/README.md`](system/README.md)

การออกแบบแบบฝึกหัดอ้างอิงจาก Bucaioni et al., *AI Support for Software Architecture Practice: A Systematic Review and Future Directions*, ACM TOSEM 35(9), 2026
