# รายงาน Docker: Build, Publish, Pull, Compose

สถานะ: ร่างคำอธิบายสำหรับทบทวน ต้องทดลองจริงแล้วปรับเป็นภาษาของตนเองก่อนส่ง

- ผู้จัดทำ: นายศิวากร พิมพะนิตย์ รหัส 671540006002-3
- GitHub repository: [เติมลิงก์ public repository]
- Docker Hub: https://hub.docker.com/r/DOCKERHUB_USERNAME/ceksu-badge
- Image ของเพื่อน: [เติมชื่อจริง]/ceksu-badge:1.0
- เพื่อนที่จับคู่: นายวรปรัชญ์ วรปัญญา รหัส 681540006001-3

## 1. หลัง pull tag 1.0 กลับมาแสดงเวอร์ชันอะไร เพราะอะไร
ผลที่คาดหวังคือ Version 1.0 และพื้นหลังสีน้ำเงิน เพราะดึง image ที่เผยแพร่ไว้ด้วย tag 1.0
การแก้ไฟล์ในเครื่องและ build tag 1.1 ไม่ได้เปลี่ยน image ของ tag 1.0 โดยอัตโนมัติ
อย่างไรก็ตาม tag เป็นชื่ออ้างอิงที่เปลี่ยนปลายทางได้เมื่อ push ทับ ส่วน digest ระบุเนื้อหา image แน่นอน
ผลทดสอบจริง: [เติมหลังทดลองส่วน 2.3]

## 2. แก้ HTML แล้ว compose up -d ทำไมไม่เปลี่ยน
เมื่อมี image ที่ build ไว้แล้ว คำสั่ง up -d ปกติไม่ได้ rebuild เพียงเพราะไฟล์ HTML เปลี่ยน
Dockerfile ใช้ COPY จึงคัดลอกไฟล์เข้า image ตอน build และไม่ได้เชื่อมไฟล์ในเครื่องกับ container
ใช้ docker compose up -d --build mybadge เพื่อ build ใหม่และให้ Compose สร้าง container ใหม่จาก image ที่เปลี่ยน
ผลทดสอบก่อนและหลัง --build: [เติมผลที่พบจริง]

## 3. ทำไมแก้ mybadge ได้ แต่แก้ friend ไม่ได้
mybadge ใช้ build: . และมีซอร์สของเรา จึงแก้ไฟล์แล้ว rebuild ได้
friend ใช้ image ที่เพื่อนเผยแพร่ และเราไม่มีซอร์สของเพื่อนในโปรเจกต์นี้
การแก้ index.html ของเราจึงไม่มีผลกับ friend; ต้องใช้ image รุ่นใหม่ที่เพื่อนเผยแพร่
แม้แก้ไฟล์ภายใน container ได้ทางเทคนิค การแก้เช่นนั้นไม่เปลี่ยน image และหายไปเมื่อสร้าง container ใหม่

## 4. จะส่ง Docker Hub หรือ GitHub ให้รุ่นน้อง
ถ้าจุดประสงค์คือรันอย่างเดียว ฉันเลือก Docker Hub เพราะ pull แล้วรันได้โดยไม่ต้อง build จากซอร์ส
ข้อดีคือใช้ artifact เดียวกันได้สะดวก ข้อเสียคือแก้ซอร์สต่อไม่ได้และต้องมี image ที่รองรับสถาปัตยกรรมเครื่อง
ถ้าต้องการให้ศึกษาและพัฒนาต่อ ฉันจะส่ง GitHub เพิ่ม เพราะมีซอร์ส Dockerfile Compose และคู่มือ
GitHub แก้ไขต่อได้สะดวก แต่ต้อง build และอาจได้ dependency ต่างกันหากไม่ได้ตรึงเวอร์ชัน

## 5. ถ้าเพื่อน push ทับ tag 1.0
container ที่รันอยู่จะไม่เปลี่ยนทันที และ image ใน cache อาจยังเป็นตัวเดิมจนกว่าจะ pull ใหม่
เมื่อ docker compose pull friend แล้ว docker compose up -d friend จะใช้งาน image ที่ดึงมาใหม่
เพื่อให้รันซ้ำได้เนื้อหาเดิม ให้ตรึง image เป็นชื่อ repository@sha256:ค่าจริงจาก RepoDigests
การใช้ tag ใหม่ช่วยจัดรุ่น แต่ tag เพียงอย่างเดียวไม่รับประกันว่าเนื้อหาจะไม่ถูก push ทับ

## ผลสำรวจ image ของเพื่อน
- ExposedPorts: [ผลจริงจาก inspect]
- Cmd: [ผลจริงจาก inspect]
- Entrypoint: [ผลจริงจาก inspect]
- Base image: [ข้อสรุปจาก history และหลักฐาน; หากระบุ tag แน่นอนไม่ได้ ให้ระบุข้อจำกัด]
- หมายเหตุ: history แสดงชั้นและคำสั่ง build แต่ไม่ได้รับประกันว่าจะบอกชื่อ/tag ของ base image เดิมครบ

## หลักฐาน
1. screenshots/01-local-run.png — หน้า Version 1.0 พร้อม URL :8080
2. screenshots/02-dockerhub-tags.png — Docker Hub แสดง tags 1.0 และ 1.1
3. screenshots/03-pull-proof.png — ไม่เหลือสอง tags ของงาน แล้ว pull 1.0 กลับ
4. screenshots/04-inspect-friend.png — inspect port ของ image เพื่อน
5. screenshots/05-compose-two-tabs.png — สองหน้าต่าง browser แสดง :8080 และ :8081 พร้อมกัน

## เอกสารอ้างอิง
- https://docs.docker.com/reference/cli/docker/compose/up/
- https://docs.docker.com/reference/cli/docker/compose/pull/
- https://docs.docker.com/reference/cli/docker/image/inspect/

