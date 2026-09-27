# ppg-test

> ⚠️ **실험 단계 (Experimental)** — 연구·검증용 프로토타입입니다. 측정·판정 결과는 의료적 진단이 아니며, 화면과 동작은 예고 없이 바뀔 수 있습니다.

**PILLYMON PPG의 측정 페이지(프론트엔드)** 입니다.
스마트폰 후면 카메라에 손가락을 대면 30초 동안 PPG(광용적맥파, 심박 생체신호) 파형을 기록하고, 이를 처리 서버로 보내 HRV 기반 스트레스 판정을 받습니다.

- 메인 레포: [qkrvlfg123/keyboard-mouse-1st-modeling](https://github.com/qkrvlfg123/keyboard-mouse-1st-modeling)
- 처리 서버: [qkrvlfg123/pillymon-ppg-server](https://github.com/qkrvlfg123/pillymon-ppg-server)
- 배포 주소: https://pillymon-ppg.vercel.app

## 스택

| 구분 | 사용 기술 |
| --- | --- |
| 페이지 | 단일 `index.html` (HTML + CSS + 바닐라 JavaScript, 빌드 도구·외부 라이브러리 없음) |
| 카메라 | `navigator.mediaDevices.getUserMedia` (후면 카메라, 30fps), `MediaStreamTrack` torch 제약으로 플래시 제어 |
| 신호 추출 | Canvas 2D `getImageData`로 프레임 중앙 영역의 평균 적색값 → PPG 파형, `requestAnimationFrame` 루프 |
| 로컬 처리 | 30fps 균일 리샘플, 1차 IIR 밴드패스(0.7–3.5 Hz), 적응형 피크 검출, RR 아티팩트 보정 → 빠른 심박수 표시 |
| 서버 통신 | `fetch` — `POST /measure` (원시 파형 전송), `GET /health` (연결 상태) |
| 상태 저장 | `localStorage` (서버 주소, 사용자 ID) |
| 배포 | Vercel (정적 호스팅, `.vercelignore`로 `index.html`만 업로드) |

## 동작 흐름

1. **ID 입력:** 영문 이니셜 2–4자와 숫자 4자리(예: `KCS1234`)를 입력하고, 확인 창에서 한 번 더 확인합니다.
2. **카메라와 플래시:** 후면 카메라를 켜고, 기기가 지원하면 플래시를 켭니다.
3. **안정화:** 손가락 접촉이 3초 동안 유지되면 측정을 시작합니다. 접촉이 끊기면 안정화를 처음부터 다시 합니다.
4. **측정(30초):** 호흡 유도 원(들숨·날숨 각 4초)과 탭 리듬 게임으로 사용자가 가만히 있도록 돕습니다.
5. **로컬 계산:** 측정이 끝나면 폰에서 심박수만 빠르게 계산해 보여줍니다.
6. **서버 판정:** 원시 파형을 처리 서버로 보내고, 서버가 계산한 HRV·스트레스 점수와 등급(저부하/중간/과부하)을 표시합니다.

## 서버 주소 설정

- 기본값은 `https://pillymon-ppg-server.onrender.com`입니다.
- QR 코드 등으로 `?server=<주소>&user=<ID>` 파라미터를 붙여 들어오면 그 값이 우선 적용되고 저장됩니다.
- 예전에 저장된 ngrok 임시 주소는 무시합니다. 로컬 서버와 ngrok으로 테스트하던 시절을 위해 요청에는 `ngrok-skip-browser-warning` 헤더가 그대로 붙어 있습니다.

## 로컬에서 열기

카메라 API는 보안 컨텍스트(HTTPS 또는 `localhost`)에서만 동작합니다.

```bash
python -m http.server 8000
# http://localhost:8000 접속 (폰에서 테스트하려면 HTTPS 배포 주소 사용)
```

## 알려진 한계

- 서버 판정은 현재 인구 평균 기준의 콜드스타트 방식만 동작합니다. 이 페이지가 `personal_calm_hrv`(평상시 측정 기록)를 보내지 않아, 개인 기준 판정으로 넘어가지 않습니다.
- Render 무료 요금제 특성상, 한동안 요청이 없으면 첫 측정의 서버 응답이 30초~1분 늦을 수 있습니다.
