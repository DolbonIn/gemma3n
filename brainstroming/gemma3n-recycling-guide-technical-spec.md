# 분리수거 즉석 가이드 - 기술 명세서

## 프로젝트 개요
- **프로젝트명**: RecycleAI - 분리수거 즉석 가이드
- **목표**: 2주 내 해커톤 MVP 완성
- **핵심 기능**: 쓰레기 촬영 → AI 분석 → 분리수거 방법 즉시 안내

## 기술 스택 확정

### 모델 및 추론
- **모델**: Gemma3n 8B (INT8 양자화)
- **추론 프레임워크**: TensorFlow Lite
- **목표 성능**: 
  - 모델 크기: ~2GB
  - 추론 시간: <300ms
  - 메모리 사용량: <3GB

### 개발 환경
- **프론트엔드**: Flutter 3.x
- **백엔드**: 오프라인 전용 (서버 없음)
- **데이터베이스**: SQLite
- **이미지 처리**: Flutter 내장 카메라 플러그인

### 타겟 플랫폼
- Android 7.0+ (API 24+)
- iOS 12.0+ (향후 고려)
- 최소 RAM: 4GB

## 핵심 기술 구현 상세

### 1. 모델 최적화 전략

```python
# Gemma3n INT8 양자화 예시
import tensorflow as tf

converter = tf.lite.TFLiteConverter.from_saved_model(gemma3n_path)
converter.optimizations = [tf.lite.Optimize.DEFAULT]
converter.representative_dataset = representative_dataset_gen
converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8]
converter.inference_input_type = tf.uint8
converter.inference_output_type = tf.uint8
tflite_model = converter.convert()
```

### 2. 분류 카테고리 (7개)
1. **플라스틱** - 페트병, 플라스틱 용기
2. **종이** - 신문, 박스, 종이컵
3. **유리** - 유리병, 유리 용기
4. **캔/금속** - 알루미늄캔, 철캔
5. **비닐** - 비닐봉지, 포장재
6. **음식물** - 음식물 쓰레기
7. **일반쓰레기** - 기타 분리수거 불가

### 3. 데이터베이스 스키마

```sql
-- 제품 정보 테이블
CREATE TABLE products (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    category TEXT NOT NULL,
    subcategory TEXT,
    image_hash TEXT,
    disposal_method TEXT NOT NULL,
    special_notes TEXT
);

-- 분리수거 규칙 테이블
CREATE TABLE recycling_rules (
    id INTEGER PRIMARY KEY,
    category TEXT NOT NULL,
    rule_description TEXT NOT NULL,
    do_items TEXT,
    dont_items TEXT
);

-- 인덱스 생성
CREATE INDEX idx_image_hash ON products(image_hash);
CREATE INDEX idx_category ON products(category);
```

### 4. UI/UX 플로우

```
[카메라 뷰]
    ↓ (촬영 버튼 탭)
[이미지 캡처 & 전처리]
    ↓ (300ms 이내)
[AI 분석 결과]
    - 카테고리 아이콘 (크게)
    - 분리수거 방법 (명확한 텍스트)
    - 특별 주의사항 (있을 경우)
    ↓ (추가 정보 버튼)
[상세 가이드]
    - 분해 방법
    - 세척 필요 여부
    - 지역별 차이점
```

## 리스크 대응 전략

### 1. 조명 조건 문제 (Medium Risk)
**문제**: 다양한 조명에서 인식률 저하
**해결책**:
- 데이터 증강: 밝기, 대비, 색온도 변형
- 전처리: 히스토그램 균등화, 화이트 밸런스 조정
- UI 가이드: "밝은 곳에서 촬영하세요" 안내

### 2. 복합 재질 처리 (Medium Risk)
**문제**: 테이크아웃 용기 등 복합 재질
**해결책**:
- DB에 복합 재질 분해 가이드 저장
- "플라스틱 컵 + 종이 라벨" → 각각 분리 안내
- 주요 1,000개 제품에 대한 사전 매핑

### 3. 지역별 규칙 차이 (Low Risk)
**문제**: 지자체별 분리수거 규칙 상이
**해결책**:
- MVP는 서울시 기준으로 통일
- 앱 내 명시: "서울시 기준, 지역별 확인 필요"
- v2.0에서 지역 선택 기능 추가 계획

## 2주 개발 일정

### Week 1: 기술 구현
| 일차 | 작업 내용 | 담당 | 완료 기준 |
|------|----------|------|----------|
| Day 1-2 | Flutter 환경 구축, TFLite 통합 | 개발자 A | 샘플 이미지 추론 성공 |
| Day 3-4 | Gemma3n 양자화 및 최적화 | ML 엔지니어 | <300ms 추론 달성 |
| Day 5-6 | SQLite DB 구축 (1,000개 제품) | 개발자 B | CRUD 테스트 완료 |
| Day 7 | 카메라 통합 및 기본 UI | 개발자 A | 실시간 프리뷰 작동 |

### Week 2: 완성도 및 데모
| 일차 | 작업 내용 | 담당 | 완료 기준 |
|------|----------|------|----------|
| Day 8-9 | UI/UX 개선 및 애니메이션 | 디자이너+개발자 | 사용성 테스트 |
| Day 10 | 엣지 케이스 처리 | 전체 | 10가지 시나리오 통과 |
| Day 11 | 성능 최적화 | ML 엔지니어 | 저사양 폰 테스트 |
| Day 12 | 데모 시나리오 작성 | PM | 3분 스크립트 완성 |
| Day 13 | 영상 촬영 및 편집 | 전체 | 초안 완성 |
| Day 14 | 최종 테스트 및 제출 | 전체 | 제출 완료 |

## 성공 지표 (KPI)

### 기술적 지표
- 추론 시간: <300ms (고사양), <500ms (중사양)
- 모델 정확도: 85%+ (1,000개 제품 대상)
- 앱 크기: <500MB
- 메모리 사용량: <3GB
- 오프라인 작동률: 100%

### 사용자 경험 지표
- 첫 결과까지 시간: <2초
- 3번 이내 탭으로 원하는 정보 도달
- 오류 발생률: <5%

## 데이터 수집 계획

### 우선순위 제품 (TOP 100)
1. 페트병 (다양한 브랜드)
2. 테이크아웃 커피컵
3. 배달 용기 (플라스틱/종이)
4. 과자 봉지
5. 우유팩/음료팩
6. 캔 음료
7. 유리병
8. 스티로폼 박스
9. 비닐봉지
10. 음식물 쓰레기

### 이미지 수집 전략
- 제품당 50장 (다양한 각도/조명)
- 총 50,000장 목표
- 크라우드소싱 + 팀 직접 촬영

## 차별화 포인트

### 1. 진정한 오프라인
- 인터넷 연결 불필요
- 언제 어디서나 사용 가능

### 2. 한국 특화
- 한국 분리수거 체계 최적화
- 한국 제품 DB

### 3. 즉각적 피드백
- 1초 내 결과 제공
- 명확한 시각적 안내

## 향후 확장 계획 (Post-MVP)

### v2.0 (3개월 후)
- 지역별 규칙 선택
- 5,000개 제품으로 확장
- 사용자 피드백 반영

### v3.0 (6개월 후)
- 바코드 스캔 통합
- 커뮤니티 기능
- 분리수거 통계/게이미피케이션

---

## 결론

분리수거 즉석 가이드는 2주 내 구현 가능한 최적의 해커톤 프로젝트입니다. 
- 기술적 리스크가 관리 가능한 수준
- 명확한 사용자 가치
- 극적인 Before/After 데모 가능
- Gemma3n의 장점 최대 활용

**다음 단계**: Day 1 - Flutter 프로젝트 생성 및 TensorFlow Lite 통합 시작