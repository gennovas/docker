# 🚀 ติดตั้ง Docker + WSL2 บน Windows 11 และย้ายไป Drive D:

## 1. เปิดฟีเจอร์ที่จำเป็น
เปิด PowerShell (Run as Administrator) แล้วรัน:
```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
Restart เครื่อง 1 ครั้ง
## 2. ติดตั้ง Linux Distro (Ubuntu)
```powershell
wsl --install -d Ubuntu
```
ตรวจสอบว่าเป็น WSL2:
```powershell
wsl --status
```
## 3. ติดตั้ง Docker Desktop
1. ดาวน์โหลด [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)
2. ระหว่างขั้นตอนการติดตั้ง ให้เลือกตัวเลือกดังนี้:
   - ✅ **Use WSL2 instead of Hyper-V**  
   - ✅ **Enable integration with Ubuntu**
     
## 4. ย้าย Docker Data ไปที่ Drive D:
คุณสามารถเลือกได้ 2 วิธี ขึ้นอยู่กับว่าต้องการย้ายเฉพาะ **Docker Data** หรือย้ายทั้ง **WSL Distro (Ubuntu)** ไปที่ D:

---
### วิธี A: เปลี่ยน `data-root` ของ Docker
1. **ปิด Docker Desktop**
2. เปิดไฟล์ config ที่ตำแหน่ง:  %AppData%\Docker\settings.json
3. เพิ่มหรือแก้ไขบรรทัด:
```json
{
  "data-root": "D:\\DockerData"
}
```
4. สร้างโฟลเดอร์ใหม่:
```makefile
D:\DockerData
```
5. เปิด Docker Desktop ใหม่
→ ตอนนี้ Docker จะเก็บ images, containers, volumes ที่ D:\DockerData

### วิธี B: ย้าย WSL Distro (Ubuntu) ไปที่ D:
ถ้าต้องการให้ทั้ง WSL ถูกย้ายไปที่ D: (รวม Docker data ด้วย)
1. Export Ubuntu เดิม
```powershell
wsl --import Ubuntu D:\WSL\Ubuntu D:\WSL\Ubuntu.tar --version 2
```
2. ลบ Ubuntu เดิม
```powershell
wsl --unregister Ubuntu
```
3. Import ใหม่ไปที่ D:
```powershell
wsl --import Ubuntu D:\WSL\Ubuntu D:\WSL\Ubuntu.tar --version 2
```
4. ตอนนี้ไฟล์ rootfs (ext4.vhdx) จะถูกเก็บที่:
```makefile
D:\WSL\Ubuntu\
```
---
## ติดตั้ง SQL Server 2022 บน Docker (Linux)
```bash
docker pull mcr.microsoft.com/mssql/server:2022-latest
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=YourStrong!Passw0rd" \
   -p 1433:1433 --name TTC-Container \
   -d mcr.microsoft.com/mssql/server:2022-latest
```
เข้าไปใน SQL Server:
```bash
docker exec -it TTC-Container /opt/mssql-tools18/bin/sqlcmd \
   -S localhost -U SA -P "YourStrong!Passw0rd"
```
ตั้งค่า @@SERVERNAME:
```sql
EXEC sp_dropserver @@SERVERNAME, 'droplogins';
EXEC sp_addserver 'TTC-SQLDB', 'local';
```
รีสตาร์ท container:
```bash
docker restart TTC-Container
```
ตรวจสอบ:
```sql
SELECT @@SERVERNAME;
```
---
## ติดตั้ง IIS บน Docker
### 1. เปิดใช้ Windows Container
Docker Desktop บน Windows สามารถสลับได้ 2 โหมด
- Linux Containers (default)
- Windows Containers
เพราะ IIS รันได้เฉพาะ Windows → ต้องสลับไปที่ Switch to Windows Containers
(คลิกขวาที่ไอคอน Docker Desktop → เลือก Switch to Windows Containers...)
### 2. ดึง IIS Image จาก Microsoft
```powershell
docker pull mcr.microsoft.com/windows/servercore/iis
```
### 3. รัน Container พร้อม IIS
```powershell
docker run -d -p 8080:80 --name IIS-Container mcr.microsoft.com/windows/servercore/iis
```
- -d = run แบบ background
- -p 8080:80 = map port 8080 ของ host → 80 ของ container
- --name IIS-Container = ตั้งชื่อ container
### 4. ทดสอบการทำงาน
```arduino
http://localhost:8080
```
จะเจอหน้า IIS default page ✅
### 5. เข้าไปแก้ไขไฟล์ใน IIS
```powershell
docker exec -it IIS-Container powershell
```
web root จะอยู่ที่:
```makefile
C:\inetpub\wwwroot
```
### 6. Map โฟลเดอร์จาก Host → Container (ถ้าอยากแก้ไฟล์นอก container)
```powershell
docker run -d -p 8080:80 --name IIS-Container `
  -v C:\MyWebsite:C:\inetpub\wwwroot `
  mcr.microsoft.com/windows/servercore/iis
```
- IIS-Container คือ ชื่อ Container
- mcr.microsoft.com/windows/servercore/iis คือ ชื่อ Image
