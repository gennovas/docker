# 🚀 ติดตั้ง Docker + WSL2 บน Windows 11 และย้ายไป Drive D:

## 1. เปิดฟีเจอร์ที่จำเป็น
เปิด PowerShell (Run as Administrator) แล้วรัน:
```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Hyper-V-All /all /norestart
dism.exe /online /enable-feature /featurename:Containers /all /norestart
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
     
## 4. ย้าย Unbuntu ไปที่ Drive D:
```bash
wsl --list --verbose
wsl --export Ubuntu D:\docker\ubuntu.tar
wsl --unregister Ubuntu
wsl --import Ubuntu D:\docker\ubuntu D:\docker\ubuntu.tar --version 2
```
---
## 5. กำหนด Disk Image Location
- เข้า Docker Desktop
- ไปที่ Settings -> Resources
- เปลี่ยน Location ไปที่ D:\Docker\Images
- กด Apply
---

## 6. ติดตั้ง SQL Server 2022 บน Docker (Linux)
```bash
docker pull mcr.microsoft.com/mssql/server:2022-latest
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=YourStrong!Passw0rd" \
   -p 1433:1433 --name MSSQL2022 \
   -d mcr.microsoft.com/mssql/server:2022-latest
```
เข้าไปใน SQL Server:
```bash
docker exec -it MSSQL2022 /opt/mssql-tools18/bin/sqlcmd \
   -S localhost -U SA -P "YourStrong!Passw0rd"
```
หรือ เข้าใช้งานด้วย SSMS
1. Server Name = localhost
2. Authentication = SQL Server Authentication
3. Login = sa
4. Password =
5. Encryption = Mandatory
6. Trust server certificate= true

ตั้งค่า @@SERVERNAME:
```sql
EXEC sp_dropserver @@SERVERNAME, 'droplogins';
EXEC sp_addserver 'SERVERNAME', 'local';
```
รีสตาร์ท container:
```bash
docker restart MSSQL2022
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

## 7. 🐳 Docker List Commands Cheat Sheet

| Command                | Description                                |
|-------------------------|--------------------------------------------|
| `docker images`         | แสดง images ทั้งหมดที่อยู่ในเครื่อง       |
| `docker ps`             | แสดง containers ที่กำลังรันอยู่            |
| `docker ps -a`          | แสดง containers ทั้งหมด (รวมที่หยุดแล้ว)   |
| `docker volume ls`      | แสดง volumes ทั้งหมด                       |
| `docker network ls`     | แสดง networks ทั้งหมด                      |
| `docker service ls`     | แสดง services (เฉพาะ Docker Swarm)         |
| `docker context ls`     | แสดง contexts (multi-docker environment)   |
| `docker buildx ls`      | แสดง builder instances (BuildKit)          |

