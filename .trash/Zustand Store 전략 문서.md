## 목표

- 컨텍스트로 store를 반드시 1개만 생성

- Zustand 라이브러리를 사용하여 전역 상태 관리

- Next.js 14 App Router 환경에 최적화

  

## 현재 프로젝트 상태

- **프레임워크**: Next.js 14.2.33 (App Router)

- **언어**: TypeScript

- **기존 구조**:

  - `src/lib/api-client.ts` - API 클라이언트 및 타입 정의 (Player, Team)

  - `app/layout.tsx` - 루트 레이아웃

- **상태 관리**: 현재 없음

  

## 구현 전략

  

### 1. 의존성 설치

```bash

pnpm add zustand

```

  

### 2. 디렉토리 구조 설계

```

frontend/

├── src/

│   ├── lib/

│   │   ├── api-client.ts (기존)

│   │   └── store/

│   │       ├── index.ts          # 단일 store export

│   │       ├── useStore.ts       # Zustand store 정의

│   │       └── types.ts          # Store 관련 타입 정의

│   └── ...

└── app/

```

  

### 3. Store 구조 설계

  

#### 3.1 타입 정의 (`src/lib/store/types.ts`)

- 기존 `api-client.ts`의 타입들을 재사용하거나 확장

- Store 상태 타입 정의:

  - `Player[]` - 선수 목록

  - `Team[]` - 팀 목록

  - `loading` - 로딩 상태

  - `error` - 에러 상태

  - `selectedPlayer` - 선택된 선수

  - `selectedTeam` - 선택된 팀

  - 기타 필요한 상태들

  

#### 3.2 Store 정의 (`src/lib/store/useStore.ts`)

- **단일 Store 원칙**: 하나의 `useStore` 훅만 생성

- **Slice 패턴 적용**: 기능별로 slice를 나누되, 하나의 store에 통합

  - 예: `playerSlice`, `teamSlice`, `uiSlice` 등

- **Actions 정의**:

  - Player 관련: `fetchPlayers`, `setSelectedPlayer`, `createPlayer`, `updatePlayer`, `deletePlayer`, `searchPlayers`

  - Team 관련: `fetchTeams`, `setSelectedTeam`, `createTeam`, `updateTeam`, `deleteTeam`, `searchTeams`

  - 공통: `setLoading`, `setError`, `resetError`

- **API 통합**: `api-client.ts`의 메서드들을 store actions에서 호출

  

#### 3.3 Store Export (`src/lib/store/index.ts`)

- 단일 진입점 제공

- `useStore` 훅 export

- 타입 export (필요시)

- Selector 함수들 export (성능 최적화용)

  

### 4. Store 구현 패턴

  

#### 4.1 Zustand Store 기본 구조

```typescript

// 단일 store에 모든 상태와 액션 포함

interface StoreState {

  // Player 관련

  players: Player[];

  selectedPlayer: Player | null;

  // Team 관련

  teams: Team[];

  selectedTeam: Team | null;

  // UI 상태

  loading: boolean;

  error: string | null;

  // Actions

  fetchPlayers: () => Promise<void>;

  setSelectedPlayer: (player: Player | null) => void;

  // ... 기타 actions

}

```

  

#### 4.2 Selector 패턴 적용

- 성능 최적화를 위해 필요한 상태만 선택하는 selector 함수 제공

- 예: `usePlayerStore`, `useTeamStore` 같은 커스텀 훅으로 래핑

  

### 5. Next.js App Router 통합

  

#### 5.1 Client Component에서 사용

- Zustand는 클라이언트 사이드 라이브러리이므로 `'use client'` 디렉티브 필요

- Store를 사용하는 컴포넌트는 Client Component로 선언

  

#### 5.2 Server Component와의 분리

- Server Component에서는 store 사용 불가

- 필요한 경우 Client Component로 분리하거나, Server Component에서 데이터를 fetch한 후 Client Component로 전달

  

### 6. 구현 단계

  

#### Step 1: 의존성 설치

- `pnpm add zustand` 실행

  

#### Step 2: 타입 정의

- `src/lib/store/types.ts` 생성

- Store 상태 인터페이스 정의

  

#### Step 3: Store 구현

- `src/lib/store/useStore.ts` 생성

- Zustand `create` 함수로 store 생성

- 모든 상태와 액션을 하나의 store에 통합

  

#### Step 4: Export 설정

- `src/lib/store/index.ts` 생성

- 단일 진입점으로 `useStore` export

- 필요시 selector 함수들 export

  

#### Step 5: 컴포넌트에서 사용

- Client Component에서 `useStore` 훅 사용

- 필요한 상태와 액션만 선택하여 사용

  

### 7. 주의사항

  

#### 7.1 단일 Store 원칙

- **절대 여러 개의 store를 만들지 않음**

- 모든 상태는 하나의 `useStore`에 통합

- 필요시 slice 패턴으로 논리적 분리하되, 물리적으로는 하나의 store

  

#### 7.2 타입 안정성

- TypeScript를 활용한 완전한 타입 정의

- Store의 모든 상태와 액션에 타입 지정

  

#### 7.3 성능 최적화

- Zustand의 자동 selector 최적화 활용

- 불필요한 리렌더링 방지를 위한 selector 패턴 사용

  

#### 7.4 API 통합

- `api-client.ts`의 메서드를 store actions에서 호출

- 에러 처리 및 로딩 상태 관리

  

### 8. 예상 파일 구조

  

```

src/lib/store/

├── index.ts          # export { useStore } from './useStore';

├── useStore.ts       # 단일 Zustand store 정의

└── types.ts          # StoreState 인터페이스 정의

```

  

### 9. 사용 예시 (참고용)

  

```typescript

// Client Component에서 사용

'use client'

import { useStore } from '@/lib/store'

  

export default function PlayerList() {

  const { players, loading, fetchPlayers } = useStore(

    (state) => ({

      players: state.players,

      loading: state.loading,

      fetchPlayers: state.fetchPlayers,

    })

  )

  // 또는 개별 selector 사용

  const players = useStore((state) => state.players)

  const fetchPlayers = useStore((state) => state.fetchPlayers)

  // ...

}

```

  

## 체크리스트

  

- [ ] Zustand 패키지 설치

- [ ] `src/lib/store/` 디렉토리 생성

- [ ] `types.ts` - Store 타입 정의

- [ ] `useStore.ts` - 단일 Zustand store 구현

- [ ] `index.ts` - Export 설정

- [ ] 기존 컴포넌트에서 store 사용으로 마이그레이션 (필요시)

  

## 참고사항

  

- Zustand 공식 문서: https://zustand-demo.pmnd.rs/

- Next.js App Router와 Zustand 통합 가이드 참고

- 단일 store 원칙을 엄격히 준수하여 유지보수성 확보