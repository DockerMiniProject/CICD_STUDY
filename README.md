# 📌 **Spring Boot CI/CD 자동화 프로젝트**

### 🎯 **프로젝트 목적**
Spring Boot 애플리케이션의 **빌드 → 원격 서버 배포 → 실행** 과정을 Jenkins를 활용해 **자동화(CI/CD)** 하는 파이프라인 구축

---

## 👨‍👨‍👦‍👦 팀원 소개  
| <img src="https://github.com/wns5120.png" width="200px"> | <img src="https://github.com/Aunsxm.png" width="200px"> | <img src="https://github.com/andytjdqls.png" width="200px"> |
| :---: | :---: | :---: |
| [유호준](https://github.com/wns5120) | [장수현](https://github.com/Aunsxm) | [이성빈](https://github.com/andytjdqls) |

---

## 🛠️ **CI/CD 파이프라인 개요**

1. **CI (Continuous Integration)**  
   - GitHub 저장소에서 변경사항(push) 발생 시,  
     Jenkins가 이를 감지하여 코드를 `git pull`
   - `.jar` 파일로 빌드 후 `myserver01`의 공유 디렉터리에 복사

2. **CD (Continuous Deployment)**  
   - `inotify-tools`를 활용해 `.jar` 파일 변경 감지
   - 감지되면 `myserver02`로 `.jar` 파일을 `scp`로 이관
   - 해당 `.jar` 파일을 `myserver02`에서 자동 실행

---

## 🧩 **1. Jenkins Credentials + VirtualBox + SSH 구성 단계**

### 🔐 Jenkins Credential을 활용한 자동 인증 SSH 구성

- **개요**: Jenkins에서 원격 서버로 `.jar` 파일을 전송하거나 명령어 실행 시, 비밀번호 없이 SSH 접속을 위해 Credential 설정이 필요

#### ✅ 구성 절차

1. **VirtualBox에 Ubuntu VM 설치**
   - Jenkins 설치 대상 및 원격 서버용 VM 구성

2. **Jenkins에서 사용할 SSH Key 생성**
   ```bash
   ssh-keygen -t rsa -b 4096 -C "jenkins@ci"
   ```

3. **원격 서버(Ubuntu)에 공개키 등록**
   ```bash
   ssh-copy-id -i ~/.ssh/id_rsa.pub user@remote-server
   ```

4. **Jenkins Credential 등록**
   - Jenkins > 관리 > Credentials > Global > Add credentials  
   - **Kind**: SSH Username with private key  
   - **Username**: 원격 서버 계정  
   - **Private Key**: 직접 입력 (id_rsa)

5. **Jenkins Pipeline 또는 SCP 명령어에서 Credential ID 활용**

---

## 🧩 **2. VirtualBox + SSH 구성 단계**

### 📦 VirtualBox 기반 Ubuntu VM 간 SSH 통신 설정

- **목표**: 동일 호스트(VirtualBox)에서 실행 중인 VM 간 보안 연결 설정

#### ✅ 구성 절차

1. **VirtualBox 포트 포워딩 설정**
   - VM1: 호스트포트 2022 → 게스트 22  
   - VM2: 호스트포트 2023 → 게스트 22

2. **VM1 → VM2 SSH 접속을 위한 키 생성**
   ```bash
   ssh-keygen
   ssh-copy-id -p 22 user@192.168.56.102   # VM2 IP 기준
   ```

3. **테스트**
   ```bash
   ssh user@192.168.56.102
   ```

4. **자동화 스크립트에서 SSH 사용**
   - `scp`, `ssh`, `rsync` 등 사용 가능

---

## 🧩 **3. VMware Workstation + SSH 구성 단계**

### 🖥️ VMware 기반 Ubuntu VM 간 SSH 포트포워딩 및 통신 설정

- **목표**: 호스트 Windows ↔ VM Ubuntu ↔ VM Ubuntu 간 SSH 통신

#### ✅ 구성 절차

1. **VMware에서 NAT or 브리지 모드로 설정**
   - 또는 포트포워딩 수동 설정:
     - VM1: 2022 → 22
     - VM2: 2023 → 22

2. **VM 간 SSH 설정**
   - VM1에서 SSH 키 생성 후 VM2에 공개키 등록
   - 비밀번호 없는 SSH 접속 가능하게 설정

3. **Windows 호스트에서 각 VM 접속 확인**
   ```bash
   ssh -p 2022 user@localhost  # VM1
   ssh -p 2023 user@localhost  # VM2
   ```

4. **CI/CD 환경 구성**
   - Jenkins는 VM1에 설치 (Jenkins에서 VM2로 배포)
   - SCP 및 SSH 통해 `.jar` 파일 자동 이관

---

## ✨ 정리 비교표

| 구분 | Jenkins Credential | VirtualBox + SSH | VMware + SSH |
|------|--------------------|------------------|---------------|
| 인증 방식 | SSH Key + Jenkins Credential 등록 | SSH Key 직접 등록 | SSH Key 직접 등록 |
| 포트 포워딩 | VirtualBox NAT 설정 필요 | VirtualBox NAT 설정 필요 | VMware 포트포워딩 또는 브리지 |
| 구성 목적 | Jenkins 자동화 인증 | VM 간 수동 연결 | Windows ↔ VM, VM 간 연결 |

---





