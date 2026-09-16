# Docker Digital Badge

งานเว็บแนะนำตัว HTML/CSS ไฟล์เดียว เสิร์ฟด้วย nginx:alpine ไม่มีไฟล์ภาพ ฟอนต์ หรือ JavaScript ภายนอก
เมื่อมี image ในเครื่องแล้ว หน้าเว็บทำงานได้โดยไม่ต้องต่ออินเทอร์เน็ต (การ build/pull ครั้งแรกต้องเข้าถึง registry)

ผู้จัดทำ: นายศิวากร พิมพะนิตย์ รหัส 671540006002-3 สาขาวิศวกรรมคอมพิวเตอร์ มหาวิทยาลัยกาฬสินธุ์
เพื่อนที่จับคู่: นายวรปรัชญ์ วรปัญญา รหัส 681540006001-3 สาขาวิศวกรรมคอมพิวเตอร์ มหาวิทยาลัยกาฬสินธุ์

> ยังต้องกรอก Docker Hub username และ image ของเพื่อนจริงก่อนส่ง และตรวจข้อความเป้าหมายหลังเรียนจบ
> REPORT.md เป็นร่าง ไม่ใช่ผลทดสอบสำเร็จ และยังไม่มีภาพหลักฐาน 5 ภาพ

## สถานะการทดสอบในเครื่องนี้
- Build สำเร็จด้วย local tag `badge-assignment-671540006002-3:1.0`
- Container `badge-assignment-671540006002-3-test` เปิดเว็บที่ http://localhost:8080 ได้ HTTP 200 และตรวจข้อความ Version 1.0/รหัสนักศึกษาผ่าน
- `docker compose config --quiet` ผ่าน แต่ยังไม่ได้รัน friend เนื่องจากรอชื่อ image จริง
- ยังไม่ได้ publish, ทดสอบ pull จาก Docker Hub หรือสร้างภาพหลักฐาน

ก่อนเริ่มขั้นตอนด้านล่าง ให้ลบ container ทดสอบนี้เพื่อคืน port 8080 (ลบเฉพาะของงานนี้):
```powershell
docker rm -f badge-assignment-671540006002-3-test
```

## โครงสร้าง
```text
<รหัสนักศึกษา>-badge/
├── index.html
├── Dockerfile
├── .dockerignore
├── .gitignore
├── docker-compose.yml
├── REPORT.md
├── README.md
└── screenshots/
```

## เตรียมงาน
ติดตั้งและเปิด Docker Desktop ในโหมด Linux containers ใช้ Docker Compose v2 ขึ้นไป และ Git
คำสั่งทั้งหมดด้านล่างใช้ PowerShell โดยเปิด terminal ในโฟลเดอร์ที่มี Dockerfile

1. โฟลเดอร์งานคือ `671540006002-3-badge`
2. ตรวจข้อมูลใน index.html และปรับข้อความเป้าหมายเป็นของตนเอง
3. กำหนด Compose name เป็น `671540006002-3-badge` เรียบร้อยแล้ว
4. แก้ image ของ friend เป็นชื่อที่ได้รับจากเพื่อน และแก้ port ฝั่ง container ตามผล inspect
5. สร้าง repository `ceksu-badge` แบบ Public บน Docker Hub

```powershell
docker version
docker compose version
$DockerHubUser = 'YOUR_DOCKERHUB_USERNAME'
$MyImage = "${DockerHubUser}/ceksu-badge"
$FriendImage = 'FRIEND_USERNAME/ceksu-badge:1.0'
```
แทนค่าทั้งสอง username จริงก่อนใช้งาน และตั้งตัวแปรใหม่เมื่อเปิด terminal ใหม่

## 1. Build และทดสอบ Version 1.0
ตรวจว่า index.html แสดง Version 1.0 และ `--background: #102e50`
```powershell
docker build -t "${MyImage}:1.0" .
docker run -d --name badge-test -p 8080:80 "${MyImage}:1.0"
Invoke-WebRequest http://localhost:8080 | Select-Object StatusCode
```
เปิด http://localhost:8080 ถ่ายภาพพร้อมแถบ URL เป็น `screenshots/01-local-run.png`
หาก port ถูกใช้อยู่ ใช้ `docker ps` ตรวจสอบก่อน อย่าลบ container ของงานอื่น

Dockerfile ใช้ FROM เพื่อเลือก nginx:alpine, COPY เพื่อใส่หน้าเว็บลง document root และ EXPOSE เพื่อบอก container port
EXPOSE ไม่ได้เปิด port บนเครื่อง host; `-p 8080:80` ทำหน้าที่ส่ง host port 8080 ไป container port 80
CMD และ ENTRYPOINT ใช้ค่าที่สืบทอดจาก base image เพื่อเริ่ม Nginx

## 2. Publish สองเวอร์ชัน
```powershell
docker login
docker push "${MyImage}:1.0"
```
ล็อกอินด้วยตนเอง ห้ามใส่ token หรือรหัสผ่านลงไฟล์โปรเจกต์
จากนั้นแก้ index.html สองจุด: `Version 1.0` → `Version 1.1` และ `--background: #102e50` → `--background: #51203c`
```powershell
docker build -t "${MyImage}:1.1" .
docker push "${MyImage}:1.1"
```
หยุดเมื่อคำสั่งใดล้มเหลวและแก้สาเหตุก่อนทำขั้นต่อไป อย่า build หรือ push ทับ tag 1.0
เปิดหน้า Tags บน Docker Hub ตรวจว่าเป็น Public และเห็นทั้ง 1.0/1.1 ถ่าย `screenshots/02-dockerhub-tags.png`

## 3. ลบ image ของงานแล้วพิสูจน์ pull
คำสั่งนี้ลบเฉพาะ container ทดสอบและสอง tags ของงาน ไม่ต้องลบ image ของโปรเจกต์อื่น
```powershell
docker rm -f badge-test
docker rmi "${MyImage}:1.0"
docker rmi "${MyImage}:1.1"
docker images
docker image ls --filter "reference=${MyImage}:*"
docker pull "${MyImage}:1.0"
docker run -d --name badge-test -p 8080:80 "${MyImage}:1.0"
```
ถ่าย terminal โดยเห็นลำดับรายการ image ที่ไม่มีสอง tags ของงาน แล้ว pull กลับ เป็น `screenshots/03-pull-proof.png`
หากลบไม่สำเร็จ ตรวจ `docker ps -a --filter "ancestor=${MyImage}:1.0"` และแก้เฉพาะ container ของงานนี้ก่อน อย่าใช้ prune ทั้งเครื่อง
layer ของ base image อาจยังแชร์กับ image อื่น จึงอาจเห็น Already exists แทนการดาวน์โหลดทุก layer ซึ่งเป็นพฤติกรรมปกติ
เปิด :8080 อีกครั้ง ต้องเห็น Version 1.0 สีเดิม แม้ไฟล์ในเครื่องเป็น 1.1 แล้ว บันทึกผลจริงใน REPORT.md

## 4. สำรวจ image เพื่อน
แลกเฉพาะชื่อ image ห้ามรับซอร์สหรือ repository ของเพื่อน
```powershell
docker pull $FriendImage
docker image inspect --format '{{json .Config.ExposedPorts}}' $FriendImage
docker image inspect --format '{{json .Config.Cmd}}' $FriendImage
docker image inspect --format '{{json .Config.Entrypoint}}' $FriendImage
docker history --no-trunc $FriendImage
```
ถ่ายผล inspect port เป็น `screenshots/04-inspect-friend.png` และบันทึก Cmd/Entrypoint/base image พร้อมหลักฐานใน REPORT.md
ExposedPorts เป็น metadata ไม่ได้ยืนยันว่าโปรแกรมฟัง port อยู่จริง จึงต้องทดสอบเข้าเว็บด้วย
history ใช้หาเบาะแสของ base image แต่บาง image อาจไม่สามารถยืนยันชื่อ/tag ต้นฉบับได้ ให้ระบุข้อจำกัดตามจริง

## 5. รัน Compose
แก้ image ของ friend ให้เป็นค่าจริงใน docker-compose.yml และใช้ container port ที่ตรวจพบ (ทั่วไปคือ 80)
mybadge มี build: . จึงสร้างจากซอร์สเรา; friend มี image: จึงใช้ของเพื่อนจาก registry
restart: unless-stopped ให้กลับมาทำงานหลัง restart เว้นแต่เราหยุดไว้เอง
```powershell
docker rm -f badge-test
docker compose config
docker compose up -d
docker compose ps
```
เปิด http://localhost:8080 (ของเรา 1.1) และ http://localhost:8081 (ของเพื่อน)
จัด browser สองหน้าต่างข้างกันให้เห็น URL ทั้งคู่ ถ่าย `screenshots/05-compose-two-tabs.png`

## 6. ทดลองแก้เว็บตามส่วน 4.3
เพิ่ม `<p>Updated by compose</p>` ใน main ของ index.html โดยนำออกมาจาก comment แล้วรัน:
```powershell
docker compose up -d
```
รีเฟรชและจดผลจริง: โดยปกติยังเป็นเว็บเดิม เพราะยังใช้ image ที่ build ไว้ ไม่ได้ mount ไฟล์สด
จากนั้นรัน:
```powershell
docker compose up -d --build mybadge
```
รีเฟรชเว็บ ต้องเห็นข้อความใหม่ คำสั่งนี้สร้าง image ใหม่จาก HTML แล้วอัปเดต container
ไม่ต้องใช้ --no-cache สำหรับการแก้ไฟล์ปกติ เพราะ COPY จะตรวจพบเนื้อหาที่เปลี่ยน

## ให้คนอื่นรันงาน
เมื่อกรอกค่าจริงและ push repo แล้ว คนอื่น clone repo เข้ามาในโฟลเดอร์นี้และรัน:
```powershell
docker compose up -d --build
docker compose ps
```
หากต้องการรันเฉพาะ image ของเราโดยไม่ใช้ซอร์ส:
```powershell
docker run -d --name my-badge -p 8080:80 YOUR_DOCKERHUB_USERNAME/ceksu-badge:1.1
```
เลือกวิธีใดวิธีหนึ่งต่อ host port 8080 เพื่อไม่ให้ชนกัน

## เตรียมสอบและแก้ปัญหา
- ย้าย friend ไป host port 9000: แก้ `8081:80` เป็น `9000:80` (คง container port ตามผล inspect) แล้ว `docker compose up -d friend`
- สร้างระบบกลับมา: `docker compose down` แล้ว `docker compose up -d` โดย down ลบเฉพาะระบบ Compose นี้
- เพื่อนออก 1.2: แก้ image tag ใน Compose แล้ว `docker compose pull friend` และ `docker compose up -d friend`
- ป้องกัน tag ถูกทับ: อ่าน digest ด้วยคำสั่งด้านล่าง แล้วใส่ `ชื่อบัญชี/ceksu-badge@sha256:ค่าจริง` ใน image
- เว็บไม่ขึ้น: ตรวจ `docker compose ps` และ `docker compose logs --tail 50` พร้อมตรวจว่า Docker Desktop เปิดอยู่

```powershell
docker image inspect --format '{{json .RepoDigests}}' $FriendImage
```

## ส่งขึ้น GitHub
สร้าง public repository เปล่าด้วยบัญชีของตนเอง แล้วแทน URL จริง:
```powershell
git init
git add index.html Dockerfile .dockerignore .gitignore docker-compose.yml README.md REPORT.md screenshots
git diff --cached
git commit -m "Complete Docker badge assignment"
git branch -M main
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/YOUR_STUDENT_ID-badge.git
git push -u origin main
```
ตรวจ diff ก่อน commit ว่าไม่มี .env, token, password หรือข้อมูลที่ไม่เกี่ยวกับงาน

## ตรวจส่งงาน
- [ ] ข้อมูลจริงครบ ชื่อโฟลเดอร์และ Compose name เป็นรหัสนักศึกษา-badge
- [ ] Docker Hub เป็น Public มี 1.0/1.1 คนอื่น pull ได้ และสองรุ่นมีสี/ข้อความต่างกัน
- [ ] ใช้ image ของเพื่อนจริงและบันทึกผล inspect
- [ ] เปิดเว็บได้ทั้งสอง port และทดสอบ down/up แล้ว
- [ ] ภาพจริงครบ 5 ภาพตามชื่อกำหนด
- [ ] REPORT.md เติมผลจริงและเรียบเรียงด้วยภาษาของตนเองครบ
- [ ] GitHub เป็น Public และไม่มี credentials

เอกสาร: [Compose up](https://docs.docker.com/reference/cli/docker/compose/up/), [Compose pull](https://docs.docker.com/reference/cli/docker/compose/pull/)
