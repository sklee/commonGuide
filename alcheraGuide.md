# 비대면 실명확인 시스템 프론트 Alchera 웹 모듈 연동 개발 가이드

<br />

# TOC

1. [디렉토리 구조](#directory-structure)
2. [ALCHERA-SDK 웹모듈 적용](#alchera-sdk)
3. [OCR](#ocr)
4. [FACE](#face)
5. [업데이트 이력](#업데이트-이력)
6. [참고](#참고)

   <br />

# directory-structure

### 디렉토리 구조 설명

```
├─SMN-NRV-WEB
│ ├─public
│ │ ├─sdk
│ │ │ ├─css
│ │ │ │ └─sdk.css
│ │ │ ├─encrypt
│ │ │ │ ├─useb_aes256.js
│ │ │ │ └─useb_aes256.wasm
│ │ │ ├─face
│ │ │ │ ├─face_lendmark_68_model-shard1.bin
│ │ │ │ ├─face_lendmark_68_model-weights_manifest.json
│ │ │ │ ├─face-api.min.js
│ │ │ │ ├─face.js
│ │ │ │ ├─lena.js
│ │ │ │ ├─tiny_face_detector_model-shard1.bin
│ │ │ │ ├─tiny_face_detector_model-weights_manifest.json
│ │ │ │ └─utils.js
│ │ │ └─ocr
│ │ │ │ ├─helpers --> ocr.js 에서 사용되는 유틸 스크립트 입니다.
│ │ │ │ ├─lib
│ │ │ │ ├─ocr.js --> ocr 인식을 수행하는 라이브러리 wrapper script 입니다.
│ │ │ │ ├─quram_simd.data
│ │ │ │ ├─quram_simd.js
│ │ │ │ ├─quram_simd.wasm
│ │ │ │ ├─quram.data
│ │ │ │ ├─quram.js
│ │ │ │ └─quram.wasm
│ │ │ ├─index.js --> ocr 모듈의 카메라를 활성화 시키는 모듈
│ │ ├─index.html --> sdk 모듈을 연동하는 상위 페이지
│ │ └─globals.js
│ └─src
│ │ └─sdk
│ │ │ └─index.js
```

| 파일           | 설명                                                                | 비고           |
| -------------- | ------------------------------------------------------------------- | -------------- |
| route/index.js | 업무 진입 시 ocr sdk 로드 로직(preloading)                          | load 시점 수정 |
| index.html     | 모듈 연동하는 상위                                                  | 변경 불필요    |
| ocr.js         | 웹어셈블리 용 SDK js                                                | 변경 불필요    |
| quram.js       | 웹어셈블리 바이너리와 데이터를 사용할 수 있도록 wrapping 된 js 파일 | 변경 불필요    |
| quram.wasm     | 웹어셈블리 바이너리 파일                                            | 변경 불필요    |
| quram.data     | 웹어셈블리 데이터 파일                                              | 변경 불필요    |
| helpers 폴더   | SDK js 파일에서 사용되는 유틸리티 js 파일 폴더                      | 변경 불필요    |

<br />

# alchera-sdk

### 1. sdk 웹모듈 로드

```javascript
//public\index.html
<script type="text/javascript" src="%PUBLIC_URL%/sdk/index.js"></script>
```

```javascript
//public\sdk\index.js
document.write('<script type="module" src="/global.js"></script>')
document.write('<script type="text/javascript" src="/sdk/face/face-api.min.js"></script>')
document.write('<script type="text/javascript" src="/sdk/encrypt/useb_aes256.js"></script>')
document.write('<script type="text/javascript" src="/sdk/ocr/lib/lodash.min.js"></script>')
document.write('<link href="/sdk/css/sdk.css" rel="stylesheet" />')
```

- public\index.html (최상위 파일)에서 sdk/index.js를 가져온다.
- ocr/face 웹 어셈블리 용 SDK js 파일을 로드한다.
  - global.js
  - sdk/face/face-api.min.js
  - sdk/encrypt/useb_aes256.js
  - sdk/ocr/lib/lodash.min.js
  - sdk/css/sdk.css
    <br>

### 2. UseBOCR/UsebFACE 객체 선언

```javascript
//public\global.js
window.UseBOCR = UseBOCR
window.UsebFACE = UsebFACE
window.Utils = Utils
```

- `window` 객체에 전역으로 ocr/face 클래스 객체(UseBOCR, UsebFACE) 추가하여 관련 함수를 사용한다.

### 3. preloading

```javascript
//모듈 활성화 및 Preloading
const isScriptLoaded = (src) => {
  return document.querySelector(`script[src*="${src}"]`) !== null
}

useEffect(() => {
  if (
    (location.pathname.includes('gdnc') ||
      location.pathname.includes('comn') ||
      location.pathname.includes('idcr') ||
      location.pathname.includes('seco') ||
      location.pathname.includes('face')) &&
    isWebBrowser(wooriBridge.getUserAgentInfo('nma-app-ver')) &&
    !isScriptLoaded(scriptSrc)
  ) {
    const ocrConf = settingCtx.ocr.config
    sdk.ocrInit(ocrConf)
  }
}, [location.pathname])
```

- sdk.ocrInit 을 통해 모듈을 활성화 시키고 관련 웹 어셈블리 파일`(42MB)`을 로드한다.
- routes\index.js에 위 로직을 추가하여 업무 진입 시 파일 로드 여부를 확인하고 로드한다.(preloading)

### 4. 다국어

- ocr 관련 다국어: utils.js
- face 관련 다국어: face-validation.js

<br>

# ocr

### 1. OcrPage

```javascript
//ocr 모듈 활성화
await ocrSdk.startOCR(settings.ocrType, ocrResultCallBack, ocrResultCallBack, progress)
```

- 카메라 스캔을 시작하고 촬영 후 callback을 호출 합니다.

```javascript
//ocr 추출 후 이미지 업로드
const { ocr_result, ocr_origin_image, ocr_face_image, ocr_masking_image } = result.review_result
```

- ocr 추출 결과
  - name
  - jumin
  - issued_date
  - region
  - driver_type
  - ...
  - ocr_origin_image --> 원본 이미지
  - ocr_face_image --> 얼굴 크롭 이미지
  - ocr_masking_image --> 신분증 크롭+마스킹 이미지

```javascript
await imgUpload('ocrOriginImage', ocr_origin_image)
await imgUpload('ocrFaceImage', ocr_face_image)
await imgUpload('ocrMaskingImage', ocr_masking_image)
```

```javascript
const imgUpload = async (type, data) => {
  const reqData = {
    type: type, // 사진 종류
    imgData: data, // 데이터
  }
  await callApi({
    showLoadingBar: false,
    url: '/nrv_svc/smn/nrv/idcr/saveWebOcrImg',
    data: reqData,
...
```

- ocr 추출된 결과 이미지를 한번에 보낼 경우 데이터가 유실되는 이슈가 발생하여 각각 따로 전송합니다.

```javascript
reqData = {
  OCR_CARD_CD: ocrCardCd, // 신분증 종류
  ID_TRUTH: ocr_result.id_truth ? ocr_result.id_truth : '', // 사본판별
  FD_CONFIDENCE: ocr_result?.fd_confidence ? ocr_result?.fd_confidence : '0.0', // fd_confidence
  NAME: ocr_result.result_scan_type === '3' ? ocr_result.name_kor : ocr_result.name, // 이름
  JUMIN_NUM: ocr_result?.jumin ? ocr_result?.jumin : '', // 주민등록번호
...
```

```javascript
callApi({
  showLoadingBar: false,
  url: '/nrv_svc/smn/nrv/idcr/idCardWebOcrImgSave',
  data: reqData,
```

- 신분증 촬영 후 추출된 정보를 전송한다.

<br>

### 2. 촬영 옵션

```javascript
//Setting.jsx
const initialState = {
  face: {
    settings: {
      encrypt: false,
    },
  },
  ocr: {
    use: true,
    config: {
      ocrServerBaseUrl: env.ENV_MI_OCR_SERVER_BASE_URL,
      resourceBaseUrl: env.ENV_OCR_RESOURCE_BASE_URL,
      licenseKey: env.ENV_OCR_LICENSE_KEY,
      ssaRetryType: 'ENSEMBLE',
      ssaRetryPivot: 0.5,
      useIDNumberValidation: false,
    },
    settings: {
      ocrMode: 'wasm',
      ocrType: 'idcard-ssa',
      inProgress: '',
      customUI: null,
      uiPosition: 'top',
      usePreviewUI: true,
    },
    ...
  },
}

//Setting.jsx
const setOcrSetting = (newSsaRetryPivot, newUseIDNumberValidation) => {
  setState({
    ...state,
    ocr: {
      ...state.ocr,
      config: {
        ...state.ocr.config,
        ssaRetryPivot: newSsaRetryPivot,
        useIDNumberValidation: newUseIDNumberValidation,
      },
    },
  })
}
```

- setOcrSetting와 같이 set 함수를 통해 컨텍스트 Setting의 옵션 값을 설정한다.
- Context Api 를 사용하여 전역으로 ocr/face 관련 옵션 값을 관리한다.

```javascript
//NRVIDVRM00.jsx
    <OcrPage
    ssaRetryPivot={idcrJsonData.idcrCntlBasScre} // 진위확인 기준점수
    useIDNumberValidation={idcrJsonData.luhnChckYn} // 룬체크(유효성 검사)
    ...
  />
```

```javascript
//OcrPage.jsx
  export default function OcrPage({
    useIDNumberValidation,
    ssaRetryPivot,
  }) {

  setOcrSetting(ssaRetryPivot, useIDNumberValidation === 'Y' ? true : false)
```

- NRVIDCRM00 페이지에서 param 세팅을 통해서 룬체크(유효성체크) 또는 사본판별(진위확인) 점수 option에 대한 적용이 가능하다.
- 옵션 값에 대한 정의는 `UI Simulator 가이드 및 활용 방법` 참고 바랍니다.

<br>

### 3. 암호화

```javascript
  //ocr.js
  useEncryptValueMode: false,
  useEncryptOverallMode: false,
  useEncryptMode: true, // 암호화 적용 (pii)
  ...
```

- useEncryptMode를 활성화 할 경우 ocr 추출 결과로 나오는 default 값(OCR type에 따라 value값 다름)을 암호화하여 추출한다.
- useEncryptMode를 비활성화 하고 useEncryptValueMode: false, useEncryptOverallMode: true로 할 경우 포괄 암호화하여 출력한다.
- useEncryptMode를 비활성화 하고 useEncryptValueMode: true, useEncryptOverallMode: false로 할 경우 OCR type 별 출력 키 목록에 들어가 있는 항목을 암호화하여 출력한다.
  - ocrResultIdcardKeylist: [] // 주민증/면허증 평문 결과 출력 키 목록
  - encryptedOcrResultIdcardKeylist: [] // 주민증/면허증 암호화 결과 출력 키 목록
  - ocrResultPassportKeylist: [] // 여권 평문 결과 출력 키 목록
  - encryptedOcrResultPassportKeylist: [] // 여권 암호화 결과 출력 키 목록
  - ocrResultAlienKeylist: [] // 외국인등록증 평문 결과 출력 키 목록
  - encryptedOcrResultAlienKeylist: [] // 외국인등록증 암호화 결과 출력 키 목록

<br>

### 4. 로딩바

```javascript
//NRVIDCRM00.jsx
{
  idcrExtLoadingBar && <NRVIDCRM03 setFlag={extStep} />
}
```

- 업무에서 idcrExtLoadingBar에 따라 풀팝업 로딩 페이지를 띄운다.

```javascript
//OcrPage.jsx
useEffect(() => {
  window.setExtStep_Y = function () {
    setIdcrExtLoadingBar(true)
  }
  window.setExtStep_N = function () {
    setIdcrExtLoadingBar(false)
  }
}, [])
```

- 풀팝업 로딩 페이지 호출하는 함수를 전역으로 등록하여 OCR 추출 결과에 따라 호출여부를 결정

```javascript
//utils.js
switch (data.inProgress) {
  case ocr.IN_PROGRESS.OCR_SUCCESS:
    window.setExtStep_Y()
    result.message = `${cardTypeString} 인식이 완료 되었습니다.`
    break
  case ocr.IN_PROGRESS.OCR_SUCCESS_WITH_SSA:
    window.setExtStep_Y()
    result.message = `${cardTypeString} 인식 및 사본(도용) 여부 판별이 완료되었습니다.`
    break
  case ocr.IN_PROGRESS.OCR_FAILED:
    window.setExtStep_N()
    result.message = `${cardTypeString} 인식에 실패하였습니다. 다시 시도해주세요.`
    break
  ...
```

<br>

# face

### 1. facePage

```javascript
//face 모듈 활성화
sdk.faceInit()
```

```javascript
await CAMERA.open(cameraRef.current)

SCANNER.scan((queue) => {
  imageCtx.handlers.setLivenessQueue(queue)
  wooriBridge.setLoadingBar('NRV1000')
  setDone(true)
  setTimeout(() => CAMERA.stop(), 0)
})
```

- 카메라 스캔을 시작하고 촬영 후 추출된 이미지(베스트샷)를 imageCtx 컨텍스트 컴포넌트에 handlers.setLivenessQueue을 통해 set 한다.

```javascript
//face 추출 후 이미지 업로드
callApi({
  showLoadingBar: true,
  url: '/nrv_svc/smn/nrv/facecert/faceCertForgery',
  data: {
    faceImgFile1: livenessImages[0].split(',')[1],
  },
  ...
```

### 2. 수정

```javascript
if (userAgent.indexOf('iphone') > -1) {
  let mode = videoElement.clientWidth > videoElement.clientHeight ? 'landscape' : 'portrait'
  if (mode === 'portrait') {
    this.resolution.width = Math.min(settings.width, settings.height)
    this.resolution.height = Math.max(settings.width, settings.height)
  } else {
    this.resolution.width = Math.max(settings.width, settings.height)
    this.resolution.height = Math.min(settings.width, settings.height)
  }
} else {
  this.resolution.width = settings.width
  this.resolution.height = settings.height
}
```

1. iphone 일 때 가로 세로가 역으로 계산되어 안면인식이 안되는 오류 발생
2. 어셈블리 파일 교체가 어려워 iphone일 경우 다시 역으로 계산하는 로직 추가

<br>

# [업데이트 이력]

### OCR wasm SDK

| 날짜      | 업데이트 내용                                               | 구분          |
| --------- | ----------------------------------------------------------- | ------------- |
| 04월 29일 | 통합 신분증으로 인신 되도록 수정(idcard-ssa)                | Web Module    |
|           | 신형여권 인식 오래걸리는 현상 수정                          |               |
|           | 외국인등록증 타입 구분 기능 추가                            |               |
|           | 외국인등록증 뒷면 인식 기능 추가                            |               |
|           | 발급일자 인식 완료시 넘어가도록 수정                        |               |
|           | 최신 SSA(사본탐지) 모델 변경                                |               |
|           | 신형 영주증 뒷면 인식 되도록 수정                           |               |
|           | 개인정보식별부호 뒷자리 유효성 검증 기능 수정               |               |
| 04월 30일 | 마스킹 이미지 메모리 이슈 관련 암호화 모듈 추가             | Server Module |
| 05월 02일 | quram.wasm과 quram.js, quram.data의 버전 호환 이슈로 재배포 | Web Module    |
| 05월 10일 | 디버깅 옵션(뒷자리 유효성검증) 추가                         | Web Module    |
|           | web 자원정리 이슈 수정                                      |               |
|           | 취약점 수정                                                 |               |
|           | Preloading 구현                                             |               |
| 05월 17일 | 위변조 신분증 식별 오류 개선 및 버전 호환 이슈 해결         | Web Module    |

<br>

# 참고

### 1. useB.WASM 연동 가이드

- reg-tech-useb.notion.site/useB-WASM-6a9bf1a301614cd49f7f0ebd6b1fcc22
- 신분증 OCR 연동 개발 가이드

### 2. ocr Demo Page

- ocr-demo.useb.co.kr
- UI Simulator 가이드
