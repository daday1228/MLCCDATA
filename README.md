# GreenSheet Lab — 초기 모델 v0.1

MLCC 그린시트 데이터 플랫폼. KRICT · Korea University · Amotech 협력 프로젝트용.

## 파일 구성

```
greensheet-lab/
├── index.html    # 메인 페이지 (6개 탭)
└── admin.html    # 원료 ID 관리 페이지
```

두 파일을 같은 디렉터리에 두면 됩니다. 외부 의존성은 모두 CDN(Pretendard, Fraunces, JetBrains Mono, Plotly.js)에서 자동 로드되므로 별도 설치는 필요 없습니다.

## 배포 방법

### Netlify
1. 두 파일을 ZIP으로 묶거나 GitHub 저장소에 push
2. Netlify에서 새 사이트 생성 → drag-and-drop 또는 GitHub 연동
3. 빌드 설정 불필요 (정적 파일)
4. 배포 후 `<도메인>/admin.html` 로 관리 페이지 접근

### 로컬 테스트
파일 시스템에서 바로 열어도 동작합니다. 또는:
```bash
python -m http.server 8000
# → http://localhost:8000
```

## 탭 구조

| # | 탭명 | 설명 |
|---|------|------|
| 01 | 입도 데이터 입력 | 원료 / DLS / SEM 데이터 입력 |
| 02 | 입도 데이터 목록 | 저장 레코드 조회·삭제·CSV 출력 |
| 03 | 물성 데이터 입력 | 3-roll mill 공정 변수 + 물성 측정 |
| 04 | 물성 데이터 목록 | 저장 레코드 조회·삭제·CSV 출력 |
| 05 | 시각화 | 입도 비교 차트 3종 + 물성 차트 3종 |
| 06 | 현재 완성도 | ML 모델 metrics.json 업로드 → 성능 표시 |

## 식별자 ID 체계

- 입도 레코드: `PSD-{YYYYMMDD}-{원료ID}-{seq2}`
- 물성 레코드: `MECH-{YYYYMMDD}-{원료ID}-{seq2}`

같은 날짜·같은 원료에 대해 여러 번 측정 시 seq가 자동 증가합니다.

## 데이터 저장 방식

브라우저 `localStorage` 사용 (별도 서버 불필요).

| 키 | 내용 |
|----|------|
| `mlcc.materials` | 등록된 원료 ID 목록 |
| `mlcc.psd_records` | 입도 분석 레코드 |
| `mlcc.mech_records` | 물성 분석 레코드 |
| `mlcc.metrics` | 업로드된 ML metrics.json |

**주의사항**:
- localStorage는 도메인·브라우저별로 분리 저장됩니다. 다른 PC에서 작업하려면 [전체 백업] → [복원] 기능 사용.
- DLS/SEM 이미지가 base64로 저장되어 용량을 차지합니다. 브라우저당 약 5~10MB 한계가 있으니, 정기적으로 [전체 백업]을 받아 JSON으로 보관하실 것을 권장드립니다.
- 시크릿/프라이빗 창에서는 종료 시 데이터가 사라집니다.

## metrics.json 스키마

현재 완성도 탭의 [스키마 예시 다운로드] 버튼으로 받으실 수 있습니다. 핵심 구조:

```json
{
  "metadata": {
    "trained_at": "ISO 8601 timestamp",
    "model_type": "모델 종류 (예: RandomForest, XGBoost)",
    "data_count": 50
  },
  "targets": {
    "uniformity":   { "label": "필름 두께 균일도", "r2": 0.85, "mae": 0.12, "rmse": 0.18,
                      "predictions": [{"actual": ..., "predicted": ...}],
                      "feature_importance": {"roll_gap_um": 0.42, ...} },
    "roughness":    { ... },
    "shrinkage":    { ... }
  },
  "learning_curve": [
    {"n_samples": 10, "r2_avg": 0.42},
    ...
  ]
}
```

학습 스크립트(scikit-learn 등)에서 이 포맷으로 export하시면 그대로 업로드 가능합니다.

## 시각화 — 물성 텍스트 처리

물성(필름 균일도 / 표면조도 / 수축 거동)이 현재 자유 텍스트 입력이므로, 시각화 탭의 산점도(차트 5)는 **텍스트에서 첫 번째 숫자를 자동 추출**해 사용합니다. 즉:

- `"Ra 12.5 nm, 측정 위치 5개점 평균"` → `12.5` 추출
- `"균일함"` → 추출 실패 → 차트에 표시 안 됨

추후 정형화 시 별도 수치 필드 추가를 권장드립니다.

## 추후 확장 검토 항목

1. 물성 입력 정형화 (텍스트 → 구조화된 수치 + 메모 분리)
2. 다중 사용자 협업 (Firebase / Supabase 백엔드 연동)
3. 이미지 압축 또는 IndexedDB 이관 (용량 한계 회피)
4. Q값 시계열 추적 (같은 원료의 분급 단계별 변화)
5. ML 모델 학습 트리거를 웹앱에서 직접 실행 (별도 API 서버 연동)

---

문의·수정 요청은 다음 작업에서 이어갑니다.
