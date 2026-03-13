# 🐾 펫 카드 배틀 게임 상세 PRD 및 개발 명세서

**작성자**: 멍뭉 (전략기획)  
**작성일**: 2026.03.13  
**버전**: v1.0  

---

## 📋 1. 프로젝트 개요 & 목적

### 1.1 프로젝트 개요
- **서비스명**: Pet Card Battle Arena (가칭)
- **장르**: 모바일 턴제 카드 배틀 RPG
- **타겟**: 20-35세 캐주얼 게이머
- **핵심 차별점**: 귀여운 펫 테마 + 전략적 카드 배틀

### 1.2 목적
- 중독성 있는 수집/육성 루프 제공
- 실시간 PvP를 통한 경쟁요소 강화
- 지속적인 컨텐츠 업데이트로 장기간 이용 유도

---

## 🎯 2. 핵심 기능 목록 (우선순위별)

### Phase 1 (MVP - 4주)
1. **사용자 인증/계정 관리**
2. **기본 카드 수집 시스템**
3. **카드 강화 시스템**
4. **AI 배틀 시스템**
5. **기본 UI/UX**

### Phase 2 (확장 - 2주)
1. **실시간 PvP 매칭**
2. **길드 시스템**
3. **일일 미션**
4. **랭킹 시스템**

### Phase 3 (고도화 - 2주)
1. **고급 훈련 시스템**
2. **이벤트 던전**
3. **카드 트레이딩**
4. **푸시 알림**

---

## 📱 3. 화면별 기능 명세

### 3.1 메인 화면
- **기능**: 전체 메뉴 접근, 알림 확인
- **구성**: 상단 리소스바, 중앙 메뉴 버튼, 하단 네비게이션
- **API**: `/api/user/dashboard`

### 3.2 카드 수집 (뽑기)
- **기능**: 가챠 시스템, 확률 표시, 애니메이션
- **구성**: 뽑기 버튼, 확률표, 보유 재화 표시
- **API**: `/api/gacha/draw`, `/api/gacha/rates`

### 3.3 카드 관리
- **기능**: 카드 목록, 상세정보, 강화, 훈련
- **구성**: 카드 그리드, 필터링, 정렬 옵션
- **API**: `/api/cards/inventory`, `/api/cards/enhance`

### 3.4 배틀
- **기능**: 턴제 배틀, 스킬 사용, 결과 처리
- **구성**: 배틀필드, 카드 UI, 타이머
- **API**: `/api/battle/start`, `/api/battle/action`

### 3.5 상점
- **기능**: 재화 구매, 패키지 상품
- **구성**: 상품 목록, 결제 인터페이스
- **API**: `/api/shop/products`, `/api/purchase`

---

## 🎴 4. 카드 시스템 상세 설계

### 4.1 카드 등급 & 확률표

| 등급 | 이름 | 뽑기 확률 | 기본 능력치 범위 | 성장률 |
|------|------|-----------|------------------|--------|
| 1★ | Common | 50% | 100-150 | 1.0x |
| 2★ | Rare | 30% | 150-250 | 1.2x |
| 3★ | Epic | 15% | 250-400 | 1.5x |
| 4★ | Legendary | 4% | 400-600 | 2.0x |
| 5★ | Mythic | 1% | 600-1000 | 3.0x |

### 4.2 능력치 계산 공식

```javascript
// 기본 능력치 계산
baseStat = rarityMultiplier * (baseValue + random(0, variance))

// 레벨업 능력치
levelStat = baseStat + (level - 1) * growthRate * rarityBonus

// 강화 능력치
enhancedStat = levelStat * (1 + enhanceLevel * 0.1)

// 최종 능력치
finalStat = enhancedStat * equipmentBonus * buffMultiplier
```

### 4.3 카드 능력치 종류
- **HP**: 생명력 (기본 1000-3000)
- **ATK**: 공격력 (기본 100-500)
- **DEF**: 방어력 (기본 50-300)
- **SPD**: 속도 (기본 50-200, 턴 순서 결정)
- **CRIT**: 치명타율 (기본 5%-25%)

---

## ⚡ 5. 강화/훈련 시스템 상세

### 5.1 강화 시스템

| 강화 단계 | 필요 재료 | 성공률 | 능력치 증가 | 실패 시 |
|-----------|-----------|--------|-------------|---------|
| +1 ~ +5 | 강화석 x1 | 100% | 5% | - |
| +6 ~ +10 | 강화석 x2 | 90% | 7% | 단계 유지 |
| +11 ~ +15 | 강화석 x3 | 70% | 10% | 1단계 하락 |
| +16 ~ +20 | 강화석 x5 | 50% | 15% | 2단계 하락 |

### 5.2 훈련 시스템
```javascript
// 훈련 효과 계산
trainingBonus = {
  combat: { atk: +10, def: +5 },    // 전투 훈련
  agility: { spd: +15, crit: +3 },  // 민첩 훈련
  endurance: { hp: +50, def: +8 }   // 지구력 훈련
}

// 훈련 소요 시간
trainingTime = baseTime * (1 + cardLevel * 0.1)
```

---

## ⚔️ 6. 배틀 알고리즘 로직

### 6.1 턴 순서 결정
```javascript
function calculateTurnOrder(team1, team2) {
  const allCards = [...team1, ...team2];
  return allCards.sort((a, b) => {
    const speedA = a.stats.spd + random(-10, 10);
    const speedB = b.stats.spd + random(-10, 10);
    return speedB - speedA;
  });
}
```

### 6.2 대미지 계산
```javascript
function calculateDamage(attacker, defender, skill) {
  const baseDamage = attacker.stats.atk * skill.damageMultiplier;
  const defense = defender.stats.def;
  const critChance = attacker.stats.crit / 100;
  
  let finalDamage = Math.max(1, baseDamage - defense);
  
  // 치명타 체크
  if (Math.random() < critChance) {
    finalDamage *= 1.5;
  }
  
  // 속성 상성 체크
  finalDamage *= getElementalMultiplier(attacker.element, defender.element);
  
  return Math.floor(finalDamage);
}
```

### 6.3 속성 상성표

| 공격\방어 | 불 | 물 | 풀 | 전기 | 어둠 |
|----------|----|----|----|----- |------|
| 불 | 1.0 | 0.5 | 2.0 | 1.0 | 1.0 |
| 물 | 2.0 | 1.0 | 0.5 | 0.5 | 1.0 |
| 풀 | 0.5 | 2.0 | 1.0 | 1.0 | 0.5 |
| 전기 | 1.0 | 2.0 | 1.0 | 1.0 | 0.5 |
| 어둠 | 1.0 | 1.0 | 2.0 | 2.0 | 1.0 |

---

## 🎲 7. 뽑기 시스템 확률 테이블

### 7.1 일반 뽑기 (젬 300개)
```json
{
  "rates": {
    "1star": { "rate": 0.50, "pity": null },
    "2star": { "rate": 0.30, "pity": null },
    "3star": { "rate": 0.15, "pity": 10 },
    "4star": { "rate": 0.04, "pity": 90 },
    "5star": { "rate": 0.01, "pity": 180 }
  },
  "guaranteedRates": {
    "10pull": { "minRarity": 3, "rate": 1.0 },
    "pitySystem": true
  }
}
```

### 7.2 프리미엄 뽑기 (젬 1500개)
- 4성 이상 확률 2배
- 5성 카드 최소 1장 보장 (10연차)

### 7.3 천장 시스템
- 180회 뽑기 시 5성 확정
- 90회 뽑기 시 4성 이상 확정
- 10회 뽑기 시 3성 이상 확정

---

## 🎮 8. 매칭 시스템 세부 로직

### 8.1 매칭 알고리즘
```javascript
function findMatch(player) {
  const playerRating = player.rating;
  const searchRange = Math.min(200, player.waitTime * 10);
  
  const candidates = getPlayersInRange(
    playerRating - searchRange,
    playerRating + searchRange
  );
  
  return candidates
    .filter(p => Math.abs(p.rating - playerRating) < searchRange)
    .sort((a, b) => Math.abs(a.rating - playerRating) - Math.abs(b.rating - playerRating))[0];
}
```

### 8.2 레이팅 시스템
- **초기 레이팅**: 1000점
- **승리**: +25~35점 (상대 레이팅 차이에 따라)
- **패배**: -15~25점
- **연승 보너스**: 3연승 이상 시 추가 +5점

### 8.3 매칭 시간 제한
- 최대 대기시간: 60초
- 60초 초과 시 AI 매칭

---

## 🎨 9. UI/UX 와이어프레임 설계

### 9.1 전체 레이아웃 구조
```
┌─────────────────────────┐
│     상단 리소스바        │
├─────────────────────────┤
│                         │
│      메인 컨텐츠         │
│                         │
├─────────────────────────┤
│   하단 네비게이션        │
└─────────────────────────┘
```

### 9.2 주요 화면별 컴포넌트

**메인 화면**
- HeaderBar (젬, 골드, 레벨)
- MenuGrid (4x2 버튼 배치)
- NotificationBanner
- BottomNavigation

**카드 목록**
- FilterTabs (등급별, 속성별)
- CardGrid (3xN 그리드)
- SortOptions
- SearchBar

**배틀 화면**
- BattleField (3D 공간)
- PlayerCards (하단 덱)
- TimerBar
- SkillButtons

### 9.3 디자인 가이드라인
- **색상**: 파스텔톤 중심, 등급별 구분색
- **폰트**: 둥근고딕, 가독성 중시
- **애니메이션**: 부드러운 전환, 60fps 목표
- **반응형**: iPhone SE ~ iPhone Pro Max 대응

---

## 🗄️ 10. 데이터베이스 스키마 설계

### 10.1 사용자 관련 테이블

```sql
-- 사용자 기본정보
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    level INT DEFAULT 1,
    exp BIGINT DEFAULT 0,
    gems INT DEFAULT 1000,
    gold BIGINT DEFAULT 10000,
    rating INT DEFAULT 1000,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_username (username),
    INDEX idx_email (email),
    INDEX idx_rating (rating)
);

-- 사용자 통계
CREATE TABLE user_stats (
    user_id BIGINT PRIMARY KEY,
    total_battles INT DEFAULT 0,
    wins INT DEFAULT 0,
    losses INT DEFAULT 0,
    win_streak INT DEFAULT 0,
    max_win_streak INT DEFAULT 0,
    total_cards_collected INT DEFAULT 0,
    highest_rating INT DEFAULT 1000,
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### 10.2 카드 시스템 테이블

```sql
-- 카드 마스터 데이터
CREATE TABLE card_master (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    rarity INT NOT NULL,
    element ENUM('fire', 'water', 'grass', 'electric', 'dark') NOT NULL,
    base_hp INT NOT NULL,
    base_