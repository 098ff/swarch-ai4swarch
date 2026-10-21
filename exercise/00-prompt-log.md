**ภาษา:** ภาษาไทย | [English](00-prompt-log.en.md)

# บันทึกการใช้ prompt

**ชื่อ / รหัสนิสิต:**
**agent ที่ใช้:** เช่น GitHub Copilot (Agent mode), Gemini Code Assist, Gemini CLI, Claude Code, Codex, Cursor, Cline: …
**โมเดล:** …
**แพ็กเกจที่ใช้:** เช่น Copilot Student, Copilot Free, Gemini รุ่นฟรี: …

บันทึกทุกคำขอที่ส่งให้ agent ช่อง "ไฟล์ที่ agent อ่าน" คือไฟล์ที่ agent รายงาน หรือที่นิสิตเห็นว่า agent เปิด

| # | งาน / run | agent + โมเดล | ข้อความที่ส่ง (ย่อ) | ไฟล์ที่ agent อ่าน | อยู่ในขอบเขตหลักฐานที่อนุญาตหรือไม่ | มีประโยชน์ (1–5) | หมายเหตุ |
|---|---|---|---|---|---|---|---|
| 1 | 1-A | GitHub Copilot (Agent mode, Auto) | `Follow prompts/1-…` run: A, evidence: `demo.proto` | `AGENTS.md`, `1-recover-architecture.md`, `demo.proto`, `1A-architecture.md` | ใช่ (อ่านเฉพาะ evidence และคำสั่ง ไม่ได้แตะโค้ดอื่น) | 5 | กู้คืนได้ 9 service contracts และไม่มโน runtime dependencies เพราะ proto เป็นเพียง interface |
| 2 | 1-B | GitHub Copilot (Agent mode, Auto: gpt-6-luna) | `Follow prompts/1-…` run: B, evidence: `demo.proto`, `kubernetes-manifests/` | `AGENTS.md`, `1-recover-architecture.md`, `demo.proto`, `kubernetes-manifests/*.yaml` | ใช่ (อ่านเฉพาะ demo.proto และ manifests ตามที่ระบุ) | 4 | สร้างตาราง Containers (12 ตัว) และ Dependencies ครบถ้วน แต่ค้างที่ขั้นตอน mermaid validator |
| 3 | 2-a | | `prompts/2-…` run a | | | | |
| 4 | 2-b | | `prompts/2-…` run b | | | | |
| 5 | 3 | | `prompts/3-…` | | | | |
| 6 | | | | | | | |
| 7 | | | | | | | |
