# System Requirements — WorkTrack Time Logging

*อ้างอิงจาก ScreenRef และ spec.md | Created: 2026-04-26*

---

## 1. หน้าปฏิทิน (Calendar View)

### 1.1 โครงสร้างปฏิทิน

- **SR-CAL-01**: ระบบแสดงปฏิทินแบบ Month View ตั้งแต่ Sun–Sat มี 5–6 แถว
- **SR-CAL-02**: วันล้นจากเดือนก่อน/หลังต้องแสดงอยู่ใน cell แต่ text สีเทาจาง เพื่อบอกว่าเป็น overflow day
- **SR-CAL-03**: วันที่ปัจจุบัน (today) แสดง date number ภายในวงกลมสีม่วง

### 1.2 Event ใน Day Cell

- **SR-CAL-04**: event ประเภท Project / BAU แสดงเป็น pill สีน้ำเงิน พร้อม label ชื่อ project
- **SR-CAL-05**: event ประเภท Leave แสดงเป็น pill สี pink-gradient
- **SR-CAL-06**: เซลล์วันที่แสดงยอดรวมชั่วโมง (daily total) ที่มุมขวาล่าง เช่น `8h`, `12h`
- **SR-CAL-07**: หากวันนั้นไม่มี log อยู่ ยอดรวมชั่วโมงไม่ต้องแสดง (หรือแสดง `0h` ตามนโยบาย)

### 1.3 วันหยุด (Holiday Labels)

- **SR-CAL-08**: วันหยุดแสดง label ชื่อวันหยุดใต้ตัวเลขวันที่ภายใน cell รองรับ bilingual (EN–TH) เช่น `Chakri Memorial Day-วันจักรี`
- **SR-CAL-09**: วันหยุดพิเศษหลายวันติดกัน (เช่น วันสงกรานต์ 3 วัน) แต่ละ cell ต้องแสดง label ของตัวเอง
- **SR-CAL-10**: background ของ weekends (Sat/Sun) แสดงสีเทาอ่อนต่างจาก weekdays

### 1.4 Sidebar ซ้าย

- **SR-CAL-11**: sidebar แสดงวันที่ที่เลือกในรูปแบบ `DD-MMM-YYYY` (เช่น `26-APR-2026`) และยอดรวมชั่วโมงของวันนั้นทันทีที่คลิก
- **SR-CAL-12**: ปุ่ม `+ New Work Log` อยู่ด้านบนสุดของ sidebar เป็น primary action
- **SR-CAL-13**: เมื่อคลิกปุ่ม `+ New Work Log` ระบบเปิด modal และ prefill วันที่ที่กำลัง selected

---

## 2. Modal — New Work Log / Edit Work Log

### 2.1 โครงสร้างหลัก

- **SR-MOD-01**: modal มี 4 tabs: `Project / BAU`, `Other Work`, `Activities`, `Leave`
- **SR-MOD-02**: tab ที่ active แสดงด้วยสีพื้นหลัง (น้ำเงินสำหรับ Project/BAU, เขียวสำหรับ Other Work)
- **SR-MOD-03**: title ของ modal เปลี่ยนตามบริบท: `New Work Log` เมื่อสร้างใหม่, `Edit Work Log` เมื่อแก้ไข
- **SR-MOD-04**: ปุ่ม `×` มุมขวาบนปิด modal โดยไม่ save
- **SR-MOD-05**: panel ด้านขวาของ modal แสดง **Work Log List** (staging area) ตลอดเวลา ไม่ว่า tab ไหนจะ active

### 2.2 Work Log List (Staging Panel)

- **SR-STG-01**: เมื่อยังไม่มี item แสดง empty state: `No items` และ helper text `You can add multiple logs at the time and they will show up here`
- **SR-STG-02**: เมื่อมี item อยู่ใน list แสดง group header ระบุประเภท เช่น `Other Work` พร้อมปุ่ม `+ Add new`
- **SR-STG-03**: แต่ละ item แสดง: icon + ชื่อ entry, date range (`DD-MMM-YYYY TO DD-MMM-YYYY`), total duration (เช่น `12h`), และปุ่ม edit
- **SR-STG-04**: ปุ่ม `+ Add new` ใน staging panel เพิ่ม draft entry ใหม่โดยไม่ลบ item ที่มีอยู่
- **SR-STG-05**: ปุ่ม `Save` (final) อยู่มุมขวาล่างของ panel — disabled เมื่อ list ว่าง, enabled เมื่อมีอย่างน้อย 1 item

### 2.3 Tab: Project / BAU

- **SR-BAU-01**: มี search box สำหรับกรองชื่อ project + ปุ่ม dropdown สำหรับเลือกแบบ list
- **SR-BAU-02**: แสดง section `Select project/BAU stage` เป็น card grid (2 แถว × 3 คอลัมน์)
- **SR-BAU-03**: มี 6 stage card พร้อม icon และ bullet-list ของ activities ย่อย:

| Stage | Activities |
|---|---|
| Impact Assessment | Impact Assessment, Solution & Architecture Design |
| Initiation | Develop Project Charter, Identify Stakeholders, Procurement Process, Project Kick-off (Team Member), Request & Assign Resources |
| Planning | Project Planning, Project Communication Plan, Detail Requirement Gathering, Identify Risks, Define WBS |
| Execution | Design, Build, Test (SIT, UAT etc.), Security Scanning, Deployment |
| Control & Monitoring | Project Progress Update, Project Performance Tracking, Bug Fixing |
| Project Closure | SRA documents preparation, Post go live support, Transfer to Operations, Project evaluation, Close project |

- **SR-BAU-04**: การคลิก stage card จะ highlight card ที่เลือก และ activity ย่อยที่อยู่ใน card นั้นถูก map เป็น activity ของ entry
- **SR-BAU-05**: ระบบต้องเลือก project และ stage ก่อนจึงจะ `Save to Work Log List` ได้

### 2.4 Tab: Other Work

- **SR-OTH-01**: มี work type selector แสดงชื่อประเภทงาน พร้อม icon, ปุ่ม `<` `>` สำหรับ navigate ระหว่าง type
- **SR-OTH-02**: มี `Select Business` dropdown — required เมื่อ work type กำหนดให้เลือก business unit; มีปุ่ม `×` clear การเลือก
- **SR-OTH-03**: ช่องกรอก `StartDate`, `EndDate`, `Time Spent (per day)` — ทั้ง 3 ฟิลด์อยู่ใน row เดียวกัน
- **SR-OTH-04**: quick chips เวลา: `15m`, `30m`, `1h`, `2h`, `3h`, `4h` — คลิกแล้ว set ค่า Time Spent ทันที
- **SR-OTH-05**: textarea `Description` (optional)
- **SR-OTH-06**: validate `EndDate >= StartDate` ก่อน staging
- **SR-OTH-07**: ปุ่ม `Delete` (พร้อม trash icon สีเทา) มุมซ้ายล่างสำหรับลบ item ที่กำลัง edit ออกจาก staging list
- **SR-OTH-08**: ปุ่ม `Save to Work Log List` มุมขวาล่าง — validate ฟิลด์ required แล้ว push item เข้า staging panel

---

## 3. Validation Rules

| Rule ID | Condition | Behavior |
|---|---|---|
| VR-001 | Time Spent (per day) ≤ 0 | reject ก่อน staging |
| VR-002 | EndDate < StartDate | reject ก่อน staging |
| VR-003 | BAU tab ไม่เลือก project | block Save to Work Log List |
| VR-004 | BAU tab ไม่เลือก stage | block Save to Work Log List |
| VR-005 | staging list ว่างเปล่า | Save (final) button disabled |
| VR-006 | activity ไม่ตรงกับ stage ที่เลือก | reject ก่อน staging (server-side) |
| VR-007 | spent รวมเกิน policy max ต่อวัน | reject พร้อม validation error |

---

## 4. Data Derivation

- **SR-CALC-01**: `Total Duration` ต่อ entry = `Time Spent (per day)` × จำนวนวันระหว่าง StartDate–EndDate (นับตามนโยบาย เช่น เฉพาะวันทำงาน)
- **SR-CALC-02**: daily total ใน calendar cell = ผลรวม `spent_minutes` ของทุก log ใน `work_date` นั้นของ user
- **SR-CALC-03**: daily total ใน sidebar = ผลรวมเดียวกับ SR-CALC-02 อัปเดต real-time เมื่อคลิกเลือกวัน

---

## 5. Interaction States

| Element | State | Visual |
|---|---|---|
| Selected tab | active | สีพื้นหลัง (น้ำเงิน/เขียว), text ขาว |
| Inactive tab | default | พื้นขาว/เทาอ่อน, text เทา |
| Today | default | วงกลมสีม่วงครอบ date number |
| Overflow day | default | text date สีเทาจาง |
| Weekend cell | default | background สีเทาอ่อน |
| Save (final) button | list ว่าง | disabled / grayed out |
| Save (final) button | list มี item | enabled สีม่วง |
| Business dropdown | selected | แสดงค่า + ปุ่ม `×` |
| Stage card | selected | border highlight หรือ background เปลี่ยน |

---

## 6. Traceability — ScreenRef → spec.md FR

| Screen | Element | FR |
|---|---|---|
| calendar.jpg | Month View Sun–Sat 5–6 rows | FR-014 |
| calendar.jpg | Selected-day sidebar + daily total | FR-015, FR-018 |
| calendar.jpg | `+ New Work Log` sidebar button | FR-016 |
| calendar.jpg | Blue pill (project event) | FR-017 |
| calendar.jpg | Pink pill (leave event) | FR-017 |
| calendar.jpg | Holiday label in cell | FR-019 |
| select-day-pickup-project.jpg | 4-tab modal | FR-020 |
| select-day-pickup-project.jpg | Work Log List staging panel | FR-021, FR-029 |
| select-day-pickup-project.jpg | Stage card grid + activity bullets | FR-002, FR-003, FR-023 |
| select-day-pickup-project.jpg | Search box for project | FR-001, FR-023 |
| when-spenttime-to-project.jpg | StartDate / EndDate / Time Spent fields | FR-025, FR-026 |
| when-spenttime-to-project.jpg | Quick time chips 15m–4h | FR-027 |
| when-spenttime-to-project.jpg | Total duration in staged item | FR-028 |
| when-spenttime-to-project.jpg | `Save to Work Log List` button | FR-022 |
| when-spenttime-to-project.jpg | `+ Add new` button | FR-030 |
| when-spenttime-to-project.jpg | Delete staged item | FR-031 |
| when-spenttime-to-project.jpg | Final `Save` button | FR-032, FR-033 |
