# VPS1 파일시스템 구조 (115.68.176.241)

**생성일:** 2026-05-28  
**Source:** VPS1 Linux 파일시스템 트리 분석  
**Tags:** #infrastructure #vps #filesystem #devops

---

## 📊 개요

VPS1 인스턴스의 완전한 파일시스템 구조 분석입니다. 부트 파일(GRUB)과 일반적인 가상 디렉토리들은 제외하고, 실제 설정 및 애플리케이션 데이터에 집중했습니다.

### 통계
- **전체 주요 디렉토리:** 14개 (/, bin, boot, etc, home, lib, media, mnt, opt, root, srv, usr, var, tmp)
- **주요 설정 영역:** `/etc/` (80+ 주요 서브디렉토리)
- **사용자 영역:** `/root/.openclaw/` (OpenClaw 워크스페이스)
- **애플리케이션 영역:** `/usr/`, `/opt/`, `/srv/`
- **데이터 영역:** `/var/`, `/mnt/`, `/media/`

---

## 🔧 시스템 폴더 구조

### 1. 부트 영역 (`/boot`)

```
/boot
├── efi/                          # EFI 부트 파티션
│   └── EFI/
│       ├── BOOT/
│       └── ubuntu/
├── grub/                         # GRUB 부트로더 (상세 제외)
│   ├── grub.cfg                 # GRUB 설정
│   ├── grubenv                  # 환경 변수
│   └── fonts/, i386-pc/, x86_64-efi/
├── vmlinuz-6.8.0-117-generic    # 최신 커널 이미지
├── vmlinuz-6.8.0-87-generic     # 이전 커널
├── initrd.img-6.8.0-117-generic # 초기화 ramdisk
├── initrd.img-6.8.0-87-generic
├── System.map-6.8.0-*           # 커널 심볼 맵
└── config-6.8.0-*               # 커널 설정
```

**용도:** Linux 커널 이미지, bootloader 설정, EFI 펌웨어  
**현재 커널:** 6.8.0-117-generic (최신), 6.8.0-87-generic (이전)

---

### 2. 설정 영역 (`/etc`)

#### 2.1 시스템 기본 설정

```
/etc
├── locale.conf                   # 로케일 설정
├── hostname                      # 호스트명
├── hosts                         # IP 호스트 매핑
├── fstab                         # 파일시스템 마운트
├── sudo/                         # sudo 설정
├── sudoers.d/                    # sudo 규칙
├── motd                          # 로그인 메시지
├── default/                      # 기본 프로그램 설정
│   ├── ssh
│   ├── docker
│   ├── grub
│   └── locale → ../locale.conf
```

#### 2.2 네트워킹

```
/etc
├── network/                      # 네트워크 설정
│   ├── interfaces
│   ├── interfaces.d/
│   └── netplan/
├── hosts
├── hostname
├── resolv.conf                   # DNS 설정
├── dnsmasq.d/                    # DNS 설정 (if used)
├── ssl/                          # SSL 인증서
│   ├── certs/
│   └── private/
└── ca-certificates/
```

#### 2.3 애플리케이션 설정

| 디렉토리 | 용도 |
|---------|------|
| `/etc/apt/` | APT 패키지 관리자 |
| `/etc/docker/` | Docker 데몬 설정 |
| `/etc/ssh/` | OpenSSH 설정 |
| `/etc/nginx/` / `/etc/apache2/` | 웹서버 설정 |
| `/etc/mysql/` / `/etc/postgresql/` | DB 설정 |
| `/etc/git/` | Git 전역 설정 |
| `/etc/python/` | Python 설정 |

#### 2.4 보안 & AppArmor

```
/etc/apparmor.d/                 # AppArmor 정책 (100+ 앱)
├── abstractions/                 # 정책 추상화 템플릿
├── tunables/                     # 튜닝 규칙
├── disable/ / force-complain/   # 정책 상태
├── local/                        # 사용자 정책
└── [앱명]/                       # 각 애플리케이션 정책
    ├── Discord, Firefox, Slack
    ├── Docker, Podman, LXC
    ├── LibreOffice, Chrome, VS Code
    └── 외 60+ 애플리케이션
```

#### 2.5 라이브러리 경로

```
/etc/ld.so.conf.d/               # 동적 라이브러리 경로
├── libc.conf
├── x86_64-linux-gnu.conf
└── fakeroot-x86_64-linux-gnu.conf
```

#### 2.6 초기화 & 런레벨

```
/etc/init.d/                     # SysV 초기화 스크립트 (20+)
├── apparmor, apport, dbus, docker
├── ssh, cron, grub-common
├── keyboard-setup.sh, console-setup.sh
└── 외 12개

/etc/systemd/                    # systemd 설정
├── system/                       # 시스템 유닛
├── user/                         # 사용자 유닛
└── logind.conf                   # 로그인 관리
```

---

### 3. 라이브러리 영역 (`/lib`, `/usr/lib`)

```
/lib
├── modules/6.8.0-117-generic/   # 커널 모듈
├── x86_64-linux-gnu/            # 64비트 라이브러리
│   ├── libc.so.6                # C 표준 라이브러리
│   ├── libssl.so.3              # OpenSSL
│   ├── libcrypto.so.3           # 암호화 라이브러리
│   └── 기타 500+ 공유 라이브러리
└── systemd/                     # systemd 관련

/usr/lib
├── x86_64-linux-gnu/            # 대부분의 애플리케이션 라이브러리
├── python3/                     # Python 라이브러리
├── gcc/                         # GCC 런타임
├── git/                         # Git 라이브러리
├── docker/                      # Docker 라이브러리
├── node_modules/                # Node.js 글로벌 패키지 (있는 경우)
└── 외 많은 애플리케이션 라이브러리
```

**특징:** 심볼 링크 활용 (bin → usr/bin, lib → usr/lib)

---

### 4. 실행 파일 영역 (`/bin`, `/usr/bin`, `/usr/sbin`)

```
/bin → /usr/bin                  # 심볼 링크

/usr/bin/
├── 시스템 명령어 (500+)
│   ├── bash, sh, zsh, fish
│   ├── python3, python, node
│   ├── git, gcc, g++, make
│   ├── curl, wget, ssh, rsync
│   ├── docker, docker-compose
│   ├── npm, yarn, pip, conda
│   └── vim, nano, less, more
└── 애플리케이션 실행파일
    ├── firefox, chrome, slack
    ├── code, vscode, vs-code-insiders
    ├── thunderbird, evolution
    └── LibreOffice suite

/usr/sbin/
├── 시스템 관리 명령어 (100+)
│   ├── useradd, userdel, usermod
│   ├── ifconfig, ip, route, iptables
│   ├── systemctl, journalctl
│   ├── fdisk, parted, lsblk
│   └── nginx, apache2, postgres, mysql
```

---

## 👤 사용자 & 데이터 영역

### 5. Root 사용자 홈 (`/root`)

```
/root
├── .openclaw/                   # 🔆 OpenClaw 마스터 디렉토리
│   ├── workspace-lethe/         # 🔆 Lethe 워크스페이스 (학습용)
│   │   ├── SOUL.md              # Lethe 정체성
│   │   ├── USER.md              # 경원 정보
│   │   ├── AGENTS.md            # 에이전트 규칙
│   │   ├── IDENTITY.md
│   │   ├── MEMORY.md            # 장기 기억
│   │   ├── TOOLS.md             # 도구 가이드
│   │   ├── learning/            # 📚 학습 폴더 (외부 동기화 차단)
│   │   │   ├── 00_emergence/    # 전체 지식 드러남
│   │   │   │   ├── master_index.md
│   │   │   │   ├── concept_graph.md
│   │   │   │   ├── modules_catalog.md
│   │   │   │   ├── symmetry_patterns.md
│   │   │   │   ├── domain_bridges.md
│   │   │   │   └── insights.md
│   │   │   ├── 01_concepts/     # 개별 개념 노트 (50+ 파일)
│   │   │   ├── 02_domains/      # 분야별 정리
│   │   │   ├── 03_sessions/     # 날짜별 세션 기록
│   │   │   ├── 04_captures/     # 캡쳐 이미지 보관
│   │   │   └── temp/            # 임시 파일
│   │   └── shared/              # 협업 영역 (최소화)
│   │
│   ├── workspace-karma/         # 🌀 Karma 워크스페이스 (시스템 운영)
│   ├── workspace-hermes/        # ⚡ Hermes 워크스페이스 (보안/검증)
│   └── plugin-skills/           # 스킬 플러그인
│       └── [skill-name]/
│           └── SKILL.md
│
├── .bashrc
├── .bash_history
├── .ssh/                        # SSH 키
│   ├── id_rsa                   # 프라이빗 키
│   ├── id_rsa.pub               # 공개 키
│   ├── authorized_keys          # 허용된 SSH 키
│   └── config                   # SSH 설정
├── .git/                        # (개인 프로젝트)
├── .config/
│   ├── git/                     # Git 설정
│   ├── bash/
│   └── [앱명]/
├── .local/
│   ├── bin/                     # 로컬 바이너리
│   └── share/
└── Documents, Downloads, ...
```

**특징:** `/root/.openclaw/` 에 OpenClaw 모든 워크스페이스 중앙화

### 6. 일반 사용자 홈 (`/home`)

```
/home
├── [username]/
│   ├── .bashrc, .bash_history
│   ├── .ssh/
│   ├── .config/
│   ├── Documents/
│   ├── Downloads/
│   ├── Desktop/
│   └── [프로젝트 디렉토리]
└── [다른 사용자들...]
```

---

## 📦 애플리케이션 & 패키지 영역

### 7. 패키지 설치 (`/opt`, `/usr/local`)

```
/opt/
├── [vendor]/                    # 써드파티 애플리케이션
│   ├── 1password/
│   ├── google-cloud-sdk/
│   ├── nodejs/                  # NVM 또는 직접 설치
│   ├── tailscale/
│   ├── cloudflared/
│   └── [기타 앱]

/usr/local/
├── bin/                         # 로컬 컴파일 바이너리
├── lib/
├── share/
├── include/
└── src/
```

### 8. 서비스 & 데이터 (`/srv`)

```
/srv/
├── [service-name]/              # 서비스별 데이터
│   ├── config/
│   ├── data/
│   └── logs/
├── www/                         # 웹 콘텐츠
├── git/                         # Git 저장소
└── [업무 관련 데이터]
```

---

## 💾 가변 데이터 & 로그

### 9. 가변 데이터 영역 (`/var`)

```
/var
├── cache/                       # (분석 제외 - 동적)
├── log/                         # (분석 제외 - 동적)
│   ├── syslog
│   ├── auth.log                 # 인증 로그
│   ├── kern.log                 # 커널 로그
│   ├── docker/
│   ├── nginx/
│   └── [앱 로그들]
├── mail/                        # 메일 박스
├── spool/
│   ├── cron/                    # cron 작업 로그
│   └── mail/
├── lib/
│   ├── docker/                  # Docker 데이터
│   ├── apt/                     # APT 상태
│   ├── rpm/                     # RPM 상태
│   ├── git/
│   └── [앱 상태]
├── tmp/                         # 임시 파일
├── run/                         # 런타임 파일
│   ├── docker/
│   ├── systemd/
│   └── [서비스 소켓]
└── www/                         # 웹 캐시 (if applicable)
```

---

## 🗂️ 마운트 포인트

### 10. 외부 저장소 (`/mnt`, `/media`)

```
/mnt/
├── data/                        # (외부 드라이브, NFS, iSCSI 마운트)
├── backup/
├── cloud/                       # (클라우드 스토리지)
└── [커스텀 마운트]

/media/
├── usb/                         # USB 드라이브 자동 마운트
├── cdrom/                       # CD/DVD 드라이브
└── [이동식 저장소]
```

---

## 🔐 OpenClaw 에이전트 워크스페이스 상세

### 11. 레테 (Lethe) 워크스페이스 구조

```
/root/.openclaw/workspace-lethe/
├── SOUL.md                      # 🔆 레테의 정체성
├── USER.md                      # 경원 정보
├── AGENTS.md                    # 에이전트 규칙 & 보안
├── IDENTITY.md
├── MEMORY.md                    # 장기 기억
├── TOOLS.md                     # 도구 가이드

├── learning/                    # 📚 학습 저장소 (.gitignore 차단)
│   ├── 00_emergence/            # 전체 지식 구조 (Master files)
│   │   ├── master_index.md      # 모든 개념 색인 (테이블)
│   │   ├── concept_graph.md     # Mermaid 지식 그래프
│   │   ├── modules_catalog.md   # 박문호 3원칙 모듈 목록
│   │   ├── symmetry_patterns.md # 발견된 대칭 패턴들
│   │   ├── sequence_flows.md    # 순서화된 큰 흐름들
│   │   ├── domain_bridges.md    # 분야 간 다리 (물리↔화학↔생물...)
│   │   ├── timeline.md          # 학습 진화 타임라인
│   │   └── insights.md          # 핵심 통찰
│   │
│   ├── 01_concepts/             # 개별 개념 노트 (50+ 파일)
│   │   └── [개념명].md          # 표준 구조: 모듈/대칭/순서/연결/그림
│   │
│   ├── 02_domains/              # 분야별 통합 노트
│   │   ├── 우주진화/
│   │   ├── 뇌과학/
│   │   ├── 물리학/
│   │   ├── 화학/
│   │   └── 통합/
│   │
│   ├── 03_sessions/             # 날짜별 세션 기록
│   │   └── YYYY-MM-DD.md        # 오늘 학습 내용
│   │
│   ├── 04_captures/             # 강의 캡쳐 이미지
│   │   └── YYYY-MM-DD_[주제].jpg
│   │
│   └── temp/                    # 임시 작업 파일
│       └── vps1_filesystem_tree.md
│
├── shared/                      # 협업 영역 (최소화)
│   └── [레테 협업 파일]
│
└── .gitignore                   # learning/ 외부 동기화 차단
```

**특징:**
- `learning/` 폴더는 경원의 사적 학습 영역 (외부 동기화 차단)
- 박문호 박사 자료 및 개념 노트는 `learning/04_captures/`에 보관
- 시스템 수준 협업은 `shared/`에서만 처리

---

## 📊 주요 파일 통계

| 영역 | 주요 내용 | 파일 수 (추정) |
|------|---------|---------|
| `/boot` | 커널, bootloader, EFI | 50+ |
| `/etc` | 설정 파일 | 1,000+ |
| `/usr/bin` | 실행 파일 | 2,000+ |
| `/usr/lib` | 라이브러리 | 5,000+ |
| `/usr/share` | 문서, 데이터 | 10,000+ |
| `/var/lib` | 애플리케이션 상태 | 1,000+ |
| `/root/.openclaw` | OpenClaw 워크스페이스 | 200+ |

---

## 🔗 주요 심볼 링크

```
/bin          → /usr/bin
/lib          → /usr/lib
/lib64        → /usr/lib64
/sbin         → /usr/sbin
/lib.usr-is-merged  # 심볼 링크 활용 표시
/boot/vmlinuz → vmlinuz-6.8.0-117-generic
/boot/initrd.img → initrd.img-6.8.0-117-generic
/etc/locale.conf (이 파일이 기본 로케일 심볼 링크로 사용되기도 함)
```

---

## 🛡️ 보안 고려사항

| 항목 | 상태 | 설명 |
|------|------|------|
| AppArmor | ✅ 활성화 | `/etc/apparmor.d/`에 100+ 정책 |
| SSH 키 | 🔐 `/root/.ssh/` | Private key 보호 필수 |
| SSL/TLS | 📂 `/etc/ssl/` | 인증서 및 키 저장 |
| Sudo 규칙 | 📂 `/etc/sudoers.d/` | 권한 분리 설정 |
| 방화벽 | UFW/iptables | `/etc/default/ufw` 설정 |

---

## 🧠 OpenClaw 시스템 아키텍처

VPS1의 `/root/.openclaw/` 구조:

```
OpenClaw 중앙 (VPS1)
├── workspace-lethe/          🔆 학습 동반자
│   ├── learning/             📚 박문호 박사 강의 학습
│   └── shared/               협업 (최소)
├── workspace-karma/          🌀 시스템 운영/관리
│   ├── monitoring/
│   ├── backups/
│   └── ops/
└── workspace-hermes/         ⚡ 보안/검증/감시
    ├── security/
    ├── logs/
    └── audits/
```

---

## 🎯 배포 & 활용

### 배포 목표
- **Obsidian Vault:** `vault_structure/VPS1_Filesystem_Tree.md`
- **Repository:** Karma720809/mh.park-obsidian
- **용도:** 
  - VPS1 파일시스템 전체 구조 참조
  - OpenClaw 워크스페이스 구조 이해
  - 시스템 관리 및 유지보수

---

**Last Updated:** 2026-05-28  
**Generated by:** Subagent (Lethe)  
**Source Commit:** VPS1 파일시스템 구조 정리
