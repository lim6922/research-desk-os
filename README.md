제공해주신 세부적인 **Supabase 설정 가이드와 보안 모델, 개발 노트**를 모두 포함하여, 한 번에 복사해서 사용할 수 있도록 최종 리포지토리용 `README.md`를 구성해 드립니다.

```markdown
# 📊 Research Desk OS

> **투자 아이디어와 리서치 가설을 체계적으로 기록하고 검증하기 위한 개인 리서치 시스템**

Research Desk는 단순한 메모장이 아닙니다. 투자 아이디어를 **[가설 → 근거 → 무효화 조건 → 검증 → 연결]** 구조로 관리하여, 시간 축 기반으로 리서치 데이터를 축적하고 논리적으로 검증하는 것을 목표로 합니다.

🔗 **[Live Demo 실행하기](https://lim6922.github.io/research-desk-os/)**

---

## ✨ Key Features

### 📝 Desk (분석 기록)
- **종목 관리:** 종목명, 코드, 섹터, 태그 기반 기록
- **가설 중심:** 핵심 가설 수립 및 근거 URL 저장
- **리스크 관리:** 아이디어 무효화 조건(Risk) 명시
- **매매 기록:** 진입가, 목표가, 손절가 관리 및 이미지 첨부
- **연결성:** 가설 간 링크 연결 및 댓글 기능

### 📺 Monitor
- 외부 투자 모니터링 화면(iframe) 연동
- 일정 투자 체크 및 보조 모니터링 대시보드 활용

### 🔍 Explorer (리서치 탐색)
- **강력한 검색:** 종목, 섹터, 태그, 본문 내용 통합 검색
- **히스토리:** 최근 분석 히스토리 조회 및 가설 연결망 탐색

### 📋 Report (리포트 작성)
- 카드 기반 리포트 작성 및 출력 지원 (Executive Summary / Macro / Sector / Conclusion)

### 📈 Quote (시세 조회)
- Yahoo Finance API 기반 실시간 시세 조회 및 전체 동기화 기능

---

## 💾 Data Storage Structure

Research Desk는 **Local-First** 구조를 사용하여 빠른 반응성을 보장하고 클라우드에 스냅샷을 백업합니다.

- **Local Storage (`rd_v10_5`):** 브라우저 내 즉각적인 데이터 저장
- **Cloud Storage (Supabase):** 사용자별 데이터를 `rd_snapshots` 테이블에 JSONB 형태로 동기화

---

## ⌨️ Keyboard Shortcuts

### Global Navigation
| Shortcut | Action |
| :--- | :--- |
| **Ctrl + Enter** | 저장 (Save) |
| **Ctrl + Shift + F** | 검색 (Search) |
| **Ctrl + Shift + C** | 종목 코드 조회 (Code lookup) |
| **Ctrl + Shift + Q** | 시세 갱신 (Quote refresh) |
| **Ctrl + Alt + Q** | 전체 시세 동기화 (Sync all quotes) |
| **Ctrl + Alt + D** | Desk 메뉴 이동 |
| **Ctrl + Alt + M** | Monitor 메뉴 이동 |
| **Ctrl + Alt + E** | Explorer 메뉴 이동 |
| **Ctrl + Alt + R** | Report 메뉴 이동 |

### Markdown Editing
| Shortcut | Action |
| :--- | :--- |
| **Ctrl + B** | **Bold** |
| **Ctrl + I** | *Italic* |
| **Ctrl + U** | <u>Underline</u> |
| **Ctrl + H** | <mark>Highlight</mark> |

---

## 🚀 Supabase Setup (처음 세팅 가이드)

프로젝트를 포크하여 본인만의 클라우드 환경을 구축하려면 아래 단계를 따르세요.

### 1️⃣ Supabase 프로젝트 생성
1. [Supabase](https://supabase.com/) 접속 후 **New Project** 클릭
2. 프로젝트 이름, Database Password, Region 설정 후 생성 완료

### 2️⃣ API 정보 확인 및 코드 입력
- **Dashboard → Project Settings → API**에서 아래 값 확인 후 코드에 적용
  - `Project URL`
  - `anon public key`
- ⚠️ **주의:** `service_role key`는 보안상 절대 프론트엔드 코드에 노출하지 마세요.

### 3️⃣ Authentication 설정
- **Authentication → Providers → Google** 활성화

### 4️⃣ Google OAuth Client 생성 (Google Cloud Console)
1. [Google Cloud Console](https://console.cloud.google.com)에서 새 프로젝트 생성
2. **OAuth Client ID** 생성
   - **Authorized JavaScript origins:** `https://<your-id>.github.io`
   - **Authorized redirect URIs:** `https://<your-project-ref>.supabase.co/auth/v1/callback`
3. 생성된 `Client ID`와 `Client Secret`을 Supabase Google Provider 설정에 입력

### 5️⃣ Supabase Redirect URL 설정
- **Authentication → URL Configuration**
  - **Site URL:** `https://<your-id>.github.io`
  - **Redirect URLs:** 
    - `https://<your-id>.github.io/research-desk-os/`
    - `https://<your-id>.github.io/research-desk-os/index.html`

### 6️⃣ GitHub Pages 배포
- 리포지토리 **Settings → Pages**에서 `main` 브랜치를 소스로 선택하여 배포

### 7️⃣ Database Schema & RLS 설정
Supabase **SQL Editor**에서 아래 스크립트를 실행하여 테이블과 보안 정책을 설정합니다.

```sql
-- 1. 화이트리스트 테이블
create table if not exists public.allowed_users (
  email text primary key,
  note text,
  created_at timestamptz not null default now()
);

alter table public.allowed_users enable row level security;

create policy "allowed_users_select_own" on public.allowed_users
for select to authenticated using (email = auth.email());

-- 2. 스냅샷 테이블
create table if not exists public.rd_snapshots (
  user_id uuid primary key references auth.users on delete cascade,
  data jsonb not null,
  data_version bigint not null default 0,
  updated_at timestamptz not null default now()
);

alter table public.rd_snapshots enable row level security;

-- RLS 정책 (본인 데이터이면서 화이트리스트에 등록된 경우만 허용)
create policy "rd_snapshots_select_own_allowed" on public.rd_snapshots
for select to authenticated using (
  user_id = auth.uid() and exists (select 1 from public.allowed_users where email = auth.email())
);

create policy "rd_snapshots_insert_own_allowed" on public.rd_snapshots
for insert to authenticated with check (
  user_id = auth.uid() and exists (select 1 from public.allowed_users where email = auth.email())
);

create policy "rd_snapshots_update_own_allowed" on public.rd_snapshots
for update to authenticated using (
  user_id = auth.uid() and exists (select 1 from public.allowed_users where email = auth.email())
);

-- 3. 화이트리스트 사용자 등록 (본인 이메일 입력 필수)
insert into public.allowed_users (email, note)
values ('your-email@gmail.com', 'owner')
on conflict (email) do update set note = excluded.note;
```

---

## 🔒 Security Model
- **인증 흐름:** Google Login → Supabase Auth → RLS Policy → `allowed_users` 화이트리스트 확인
- **보안 특징:**
  - `anon` 키 공개를 통한 클라이언트 직접 통신
  - RLS(Row Level Security)를 통한 사용자별 데이터 완전 격리
  - 화이트리스트 시스템으로 비인가 사용자의 데이터 업로드 원천 차단

---

## 🛠 Development & Error Handling

### Common Errors
- **RLS Error (`new row violates row-level security policy`):**
  - 로그인 상태인지 확인하세요.
  - `allowed_users` 테이블에 본인 이메일이 정확히 등록되었는지 확인하세요.
  - `user_id`와 `auth.uid()`가 일치하는지 `select auth.uid(), auth.email();` 쿼리로 확인하세요.

### Notes
- GitHub Pages 기반의 정적 배포 최적화
- 로컬 성능 극대화를 위한 LocalStorage 우선 구조
- 향후 Supabase Storage를 통한 이미지 관리 및 협업 기능 확장 예정

---
Copyright © 2024 Research Desk OS. All rights reserved.
```
