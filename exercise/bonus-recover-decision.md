# Bonus (optional, take-home): Recover an unwritten decision · โบนัส (ไม่บังคับ ทำที่บ้าน): กู้คืนการตัดสินใจที่ไม่ได้บันทึกไว้ (≈ 15 min · นาที)

> **Lecture link:** T5 (Dhar et al. generate decisions from code and diagrams to reduce *knowledge vaporization*), AICH4.
> **Question:** when AI explains *why* a system was built this way, how much of the explanation is evidence and how much is a confident guess?
>
> **เชื่อมโยงกับการบรรยาย:** T5 (Dhar et al. สร้างการตัดสินใจจากโค้ดและแผนภาพเพื่อลด *การสูญหายของความรู้* หรือ knowledge vaporization), AICH4
> **คำถาม:** เมื่อ AI อธิบายว่า*ทำไม*ระบบจึงถูกสร้างแบบนี้ คำอธิบายส่วนใดมาจากหลักฐาน และส่วนใดเป็นการเดาอย่างมั่นใจ

## Run · ลงมือทำ

Start a new chat or session in your agent and send:

เริ่มแชตหรือ session ใหม่ แล้วส่งข้อความนี้ให้ agent

```
Follow prompts/bonus-recover-decision.md
decision: Why do all services communicate synchronously over gRPC, using one shared demo.proto?
evidence: system/online-boutique/protos/demo.proto, system/online-boutique/src/checkoutservice/main.go, system/online-boutique/src/frontend/main.go
```

The agent writes the reconstructed ADR, including its **Claim check** table, to [`outputs/bonus-recovered-adr.md`](outputs/bonus-recovered-adr.md).

agent จะเขียน ADR ที่กู้คืนมา พร้อมตาราง **Claim check** ไว้ที่ [`outputs/bonus-recovered-adr.md`](outputs/bonus-recovered-adr.md)

## Audit the claim check · ตรวจสอบตาราง claim check

Choose **five** claims from the agent's table: at least two it marked *Evidenced* and two it marked *Inferred*. Check each one yourself.

เลือกข้อกล่าวอ้าง**ห้าข้อ**จากตารางของ agent โดยเป็นข้อที่ agent ระบุว่า *Evidenced* (มีหลักฐาน) อย่างน้อยสองข้อ และ *Inferred* (อนุมานเอา) อย่างน้อยสองข้อ แล้วตรวจสอบแต่ละข้อด้วยตัวเอง

| # | Claim (short)<br>ข้อกล่าวอ้าง (ย่อ) | AI says<br>AI ระบุว่า | You say<br>นิสิตระบุว่า | Why (file path or reasoning)<br>เหตุผล (path ของไฟล์หรือการให้เหตุผล) |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

**Count · นับ:** of all the agent's claims, how many are *Evidenced* according to the agent? · จากข้อกล่าวอ้างทั้งหมด agent ระบุว่า *Evidenced* กี่ข้อ ___ According to you? · ตามที่นิสิตตรวจ ___

## Reflect · สะท้อนคิด

1. Did the agent ever label a guess as *Evidenced*? Give an example, or explain how you checked that it didn't.
   agent เคยระบุการเดาว่าเป็น *Evidenced* หรือไม่ ยกตัวอย่าง หรืออธิบายว่าตรวจสอบอย่างไรว่าไม่มี
2. A new developer reads only this reconstructed ADR. What wrong belief could they form?
   หากนักพัฒนาใหม่อ่านเฉพาะ ADR ที่กู้คืนมานี้ เขาอาจเข้าใจผิดในเรื่องใด
3. What can AI recover from code, and what can only the original architects know? Use one example from your ADR.
   AI กู้คืนอะไรจากโค้ดได้บ้าง และอะไรที่มีเพียงสถาปนิกผู้ออกแบบเดิมเท่านั้นที่รู้ ยกตัวอย่างหนึ่งตัวอย่างจาก ADR ของนิสิต
