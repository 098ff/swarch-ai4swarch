**ภาษา:** ภาษาไทย | [English](README.en.md)

# AI for Software Architecture: แบบฝึกหัดลงมือทำ

**2110521 Software Architecture · สัปดาห์ที่ 12 · Tutorial 5 / Assignment 5**

ในแบบฝึกหัดนี้ นิสิตจะใช้ **AI coding agent ใน VS Code** เป็นผู้ช่วยด้านสถาปัตยกรรมซอฟต์แวร์ กับระบบ microservices จริงชื่อ [Online Boutique](system/online-boutique/) โดยใช้ agent ตัวใดก็ได้ที่นิสิตมี เช่น GitHub Copilot, Gemini, Claude, Codex, Cursor หรืออื่น ๆ แล้วประเมินว่างานที่ AI ทำออกมานั้นดีแค่ไหน เป้าหมายไม่ใช่ให้ AI ทำงานแทน แต่เพื่อค้นหาว่า **AI ช่วยสถาปนิกซอฟต์แวร์ได้ตรงไหน และตรงไหนที่ยังเชื่อถือไม่ได้** ซึ่งเป็นข้อสรุปหลักของการบรรยายสัปดาห์นี้

นิสิตจะได้:

1. **กู้คืน (recover) สถาปัตยกรรม** จากซอร์สโค้ด (หัวข้อ T7: architecture recovery)
2. **กู้คืนการตัดสินใจ** ที่ไม่เคยถูกบันทึกไว้ (T5: decision recovery, knowledge vaporization)
3. **ตัดสินใจเรื่องใหม่** ด้วย ADR ที่ AI ช่วยเขียน ทั้งแบบมีและไม่มีบริบท (T3: ADR generation)
4. **รีวิว** ADR แบบเดียวกับคณะกรรมการสถาปัตยกรรม แล้วรับผิดชอบฉบับสุดท้ายด้วยตัวเอง (AICH4, "Chief Strategist")

เวลาที่ใช้โดยประมาณ: ราว 75 นาทีในคาบเรียน

---

## ก่อนเข้าเรียน: เตรียมเครื่อง (≈ 15 นาที ควรทำล่วงหน้า)

1. **ติดตั้ง [VS Code](https://code.visualstudio.com/)**
2. **เตรียม AI agent ที่อ่านและเขียนไฟล์ใน workspace ได้** ใช้ตัวใดตัวหนึ่งต่อไปนี้ก็ได้

   | Agent | ใช้งานที่ไหนใน VS Code | ตัวเลือกฟรีสำหรับนิสิต |
   |---|---|---|
   | **GitHub Copilot** ในโหมด **Agent** | หน้าต่าง Chat | Copilot Student ผ่าน [GitHub Education](https://docs.github.com/en/copilot/how-tos/copilot-on-github/set-up-copilot/enable-copilot/set-up-for-students) (การอนุมัติอาจใช้เวลาหลายวัน) หรือ Copilot Free |
   | **Gemini Code Assist** ในโหมด agent | แผง Gemini | รุ่นฟรีด้วยบัญชี Google ส่วนตัว |
   | **Gemini CLI** | terminal ใน VS Code | รุ่นฟรีด้วยบัญชี Google |
   | **Claude Code**, **Codex**, **Cursor**, **Cline** ฯลฯ | extension, terminal หรือตัว editor | ขึ้นอยู่กับเครื่องมือ |

   การแชตธรรมดาที่เปิดไฟล์ไม่ได้จะใช้ไม่ได้ เพราะ agent ต้องอ่านโค้ดและเขียนผลลัพธ์ลงไฟล์
3. **สร้างสำเนา repository นี้เป็นของตัวเอง**
   - กด **Use this template → Create a new repository** (ตั้งเป็น private ได้)
   - clone ลงเครื่อง แล้วเปิดโฟลเดอร์ใน VS Code
4. **ติดตั้ง extension ที่แนะนำ** เมื่อ VS Code ถาม คือ **Markdown Preview Mermaid Support** สำหรับแสดงแผนภาพที่นิสิตจะสร้าง
5. **ตรวจว่า agent เห็นกติกาของ repo นี้** เปิดแชตหรือ session ใหม่ แล้วถามว่า *"What are the rules for AI agents in this repository?"* agent ควรสรุปเนื้อหาจาก [`AGENTS.md`](AGENTS.md) ได้ หากไม่ได้ ให้ขึ้นต้นข้อความของทุกงานด้วย `Read AGENTS.md first.`

## แบบฝึกหัดนี้ใช้กับ agent ใดก็ได้อย่างไร

- **กติกาอยู่ใน [`AGENTS.md`](AGENTS.md)** ซึ่ง agent ส่วนใหญ่อ่านอัตโนมัติ และมีไฟล์ชี้ทางขนาดเล็กให้เครื่องมือที่หาไฟล์ชื่ออื่นโหลดกติกาเดียวกัน ได้แก่ `GEMINI.md` สำหรับ Gemini, `CLAUDE.md` สำหรับ Claude Code และ `.github/copilot-instructions.md` สำหรับ Copilot
- **โจทย์แต่ละงานเป็นไฟล์ Markdown ธรรมดาในโฟลเดอร์ [`prompts/`](prompts/)** ใบงานจะบอกข้อความที่ต้องส่งให้ agent ไว้ครบ เช่น

  ```
  Follow prompts/1-recover-architecture.md
  run: A
  evidence: system/online-boutique/protos/demo.proto
  ```

- **agent จะเขียนผลลัพธ์ไว้ใน `exercise/outputs/`** นิสิตตรวจไฟล์นั้นแล้วจึงยอมรับการเปลี่ยนแปลง หาก agent พยายามแก้ไขไฟล์อื่น ให้ปฏิเสธหรือ undo
- **agent ต้องไม่** แก้ไข `system/`, `prompts/`, ตัวอย่าง ADR หรือใบงานของนิสิต และต้องไม่รันคำสั่งใน terminal หาก agent ขอรันคำสั่ง ให้ปฏิเสธ

เนื่องจากทุกคนใช้ไฟล์โจทย์และกติกาชุดเดียวกัน จึงเปรียบเทียบผลจาก agent ต่างตัวกันได้ และการเปรียบเทียบนี้เป็นส่วนหนึ่งของงานที่ 1

## โควตาการใช้งาน

งานหลักใช้ประมาณ **8–10 คำขอ (agent requests)** ส่วน Run C (ไม่บังคับ) ในงานที่ 1 เพิ่มอีกหนึ่งคำขอ แต่เป็นคำขอที่ใช้โควตามาก

- คำขอแต่ละครั้งของ agent จะอ่านไฟล์และเขียนผลลัพธ์ จึงใช้โควตามากกว่าการแชตทั่วไป **ยิ่งอ่านไฟล์น้อยยิ่งประหยัด**
- **เริ่มแชตหรือ session ใหม่ทุกครั้งที่ทำ run ใหม่** เพราะบทสนทนาเก่าจะถูกส่งไปพร้อมทุกคำขอ
- หากโควตาฟรีหมดระหว่างทำ สามารถเปลี่ยนไปใช้ agent ตัวอื่นได้ โดยบันทึกไว้ใน prompt log

## กติกา

- **บันทึกทุกคำขอ** ไว้ใน [`exercise/00-prompt-log.md`](exercise/00-prompt-log.md) ได้แก่ ใช้ agent และโมเดลใด ถามอะไร agent อ่านไฟล์ใด และอยู่ในขอบเขตหลักฐานที่อนุญาตหรือไม่
- **ตรวจสอบคำกล่าวอ้างกับโค้ดจริง** คำกล่าวอ้างที่ไม่มี path ของไฟล์เป็นหลักฐาน ถือเป็นการเดาจนกว่านิสิตจะตรวจสอบเอง
- **อย่าวางคำตอบของ AI เป็นข้อสรุปของตัวเอง** โดยไม่ระบุที่มา ในใบงานแยกช่องสำหรับ *คำตอบของ AI* กับ *ความเห็นของนิสิต* ไว้แล้ว

## งานที่ต้องทำ

| # | ใบงาน | ไฟล์โจทย์ | จำนวนคำขอ |
|---|---|---|---|
| 1 | [กู้คืนสถาปัตยกรรม](exercise/01-recover-architecture.md) | `prompts/1-recover-architecture.md` | 2 (+1 ไม่บังคับ) |
| 2 | [กู้คืนการตัดสินใจที่ไม่ได้บันทึกไว้](exercise/02-recover-decision.md) | `prompts/2-recover-decision.md` | 1 |
| 3 | [เขียน ADR สำหรับความต้องการใหม่](exercise/03-write-adr.md) | `prompts/3-write-adr.md` | 2 |
| 4 | [รีวิวและรับผิดชอบฉบับสุดท้าย](exercise/04-review-adr.md) | `prompts/4-review-adr.md` | 1 |

> ใบงานเขียนเป็นภาษาอังกฤษ นิสิตตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## การส่งงาน (Assignment 5 ส่งภายในท้ายคาบ)

1. กรอกใบงานทั้งหมดในโฟลเดอร์ `exercise/` เก็บไฟล์ที่ agent สร้างไว้ใน `exercise/outputs/` และเขียน ADR ฉบับสุดท้ายไว้ที่ `docs/adr/0002-*.md`
2. commit และ push ขึ้น repository ของตัวเอง
3. ส่ง URL ของ repository ใน MCV หาก repository เป็น private ให้เพิ่มผู้สอนเป็น collaborator

## โครงสร้างของ repository

```
AGENTS.md                   กติกาสำหรับ AI agent ทุกตัว (+ ไฟล์ชี้ทาง GEMINI.md, CLAUDE.md, .github/copilot-instructions.md)
prompts/                    ไฟล์โจทย์สี่งานที่ให้ agent ทำตาม
docs/
  adr/template.md           รูปแบบ ADR ที่ใช้ในวิชา
  adr/0001-…                ตัวอย่าง ADR (ใช้ในงานที่ 3)
  scenario.md               บริบททางธุรกิจสำหรับงานที่ 3
exercise/                   ใบงาน (นิสิตแก้ไขไฟล์เหล่านี้)
  outputs/                  ไฟล์ที่ AI agent เขียน
system/online-boutique/     ระบบที่ใช้วิเคราะห์ (ฉบับตัดทอน)
```

## ที่มาและลิขสิทธิ์

ระบบที่ใช้ศึกษาคือ [Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo) ของ Google ภายใต้ Apache License 2.0 ดูสิ่งที่เปลี่ยนแปลงสำหรับแบบฝึกหัดนี้ได้ที่ [`system/README.md`](system/README.md)

การออกแบบแบบฝึกหัดอ้างอิงจาก Bucaioni et al., *AI Support for Software Architecture Practice: A Systematic Review and Future Directions*, ACM TOSEM 35(9), 2026
