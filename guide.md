# 비대면 실명확인 시스템 재구축 프론트 프로젝트

<br />

# TOC

1. [디렉토리 구조](#directory-structure)
2. [프로젝트 실행 가이드](#project-guide)
3. [명명규칙](#rule)
4. [Custom Hooks](#custom-hooks)
   - [통신](#1.-통신-:-useapi)
   - [사용자/인증 정보](#2.사용자/인증-:-useauth)
   - [UI 제어](#3.-ui-제어)
   - [OS 확인](#4.-os-확인)
   - [페이지이동](#5.-페이지-이동)
5. [공통 유틸 가이드](#common-utils)
6. [고객 행동 로그 수집](#cdp)
7. [유량제어](#nefunnel)

<br />

# directory-structure

### 1. 디렉토리 구조

```
├─SMN-NRV-WEB
│ ├─public
│ │ ├─images
│ │ ├─js
│ │ │ ├─cdp
│ │ │ └─netfunnel
│ │ ├─sdk
│ │ │ ├─css
│ │ │ ├─encrypt
│ │ │ ├─face
│ │ │ └─ocr
│ │ └─transkey_mobile
│ └─src
│ │ ├─common
│ │ │ ├─alchera
│ │ │ ├─api
│ │ │ ├─components
│ │ │ ├─context
│ │ │ ├─hooks
│ │ │ ├─layout
│ │ │ ├─routes
│ │ │ ├─store
│ │ │ ├─styles
│ │ │ └─utils
│ │ ├─pages
│ │ │ ├─face
│ │ │ ├─nrv
│ │ │ ├─ocr
│ │ │ └─test
│ │ └─sdk
```

### 2. 디렉토리 설명

- `/SMN-NRV-WEB/public/js/cdp` : 디지털로그 수집을 위한 스크립트 파일
- `/SMN-NRV-WEB/public/js/netfunnel` : 유량제어 스크립트 파일
- `/SMN-NRV-WEB/public/sdk` : Alchera ocr/face 관련 어셈블리/css/인코딩 관련 파일 모음
- `/SMN-NRV-WEB/src/common/components` : 재사용 또는 유틸리티 컴포넌트 모음
- `/SMN-NRV-WEB/src/common/context` : 공통으로 사용되는 컨텍스트(Context)를 관리
- `/SMN-NRV-WEB/src/common/hooks` : 재사용 가능한 커스텀 훅 또는 로직을 관리
- `/SMN-NRV-WEB/src/common/layout` : 공통 Header 또는 세션 재로드 관련 파일 모음
- `/SMN-NRV-WEB/src/common/routes` : 라우팅 설정 또는 라우팅 관련 로직 분리 파일
- `/SMN-NRV-WEB/src/common/store` : 전역상태관리(Redux) 관련 설정 파일 모음
- `/SMN-NRV-WEB/src/common/utils` : 공통 유틸리티 함수(날짜,문자열처리 등) 파일 모음
- `/SMN-NRV-WEB/src/pages/nrv` : 라우트와 연결된 각 페이지 컴포넌트와 비즈니스 로직이 정의되있는 파일 모음

<br />

# Project Guide

### 1. React 구동 방법

- Visual Studio code 에서 smn-nrv-web.git을 git clone 받는다.
- clone 받은 폴더 위치에서 Terminal을 열고 다음을 실행한다.

```bash
npm install
npm run start:local
```

### 2. 외부 라이브러리 추가

1. 추가된 라이브러리 파일을 가져와 \node_modules에 수동으로 추가
2. 추가된 라이브러리 `package.json`의 `dependencies:{}` 목록을 프로젝트 package.json의 dependencies:{}에 추가한다.

```properties
# npm insatll 시 설치되는 라이브러리 목록
"dependencies": {
  "axios": "^0.21.4",
  "buffer": "^6.0.3",
  "csstype": "3.1.3",
  "dom-helpers": "5.2.1",
  "i18next": "^22.4.15",
  ...
},
```

3. package-json.lock 을 제거 후 npm install 한다.
4. npm install 진행 후 `'not found module'` 발생 시 \node_modules\ .bin에 라이브러리 버전이 추가되었는지 확인한다.

### 3. `.env.local`

```properties
# RUNNING MODE (LOCAL, DEV, PROD)
REACT_APP_SERVER_RUNNING_MODE=LOCAL

# SERVER BIZ
REACT_APP_BIZ_DIS=SMN

# URL
REACT_APP_SERVER_BASE_URL=http://localhost:8080
REACT_APP_API_PROXY_ADDR=http://localhost:8080
```

로컬에서 BFW와 FFW를 실행하는 경우 CROS Policy 설정을 위해 `REACT_APP_SERVER_BASE_URL` 와 `REACT_APP_API_PROXY_ADDR` 값을 기재한다. \
(개발, 테스트, 운영환경에서는 동일한 도메인을 사용하므로 별도 설정 필요없음.)

### 4. Proxy 설정

`setupProxy.js` 파일에 특정 Context로 시작하는 경우 Proxy 설정

```javascript
const { createProxyMiddleware } = require('http-proxy-middleware')

/**
 * @description
 * 특정 Context로 시작하는 주소로 요청을 보낼 경우
 * Target Base URL 기준으로 Proxy를 설정
 */
module.exports = function (app) {
  app.use(
    ['/nrv_svc/smn/nrv', '/nrv_svc/transkeyServlet'],
    createProxyMiddleware({
      target: process.env.REACT_APP_API_PROXY_ADDR, // Target Base URL
      changeOrigin: true,
    })
  )
}
```

### 4. 프로젝트 실행

```bash
NPM SCRIPTS > start:local
  또는
npm run start:local
```

# Rule

| 구분             | 값      |
| ---------------- | ------- |
| AA L3코드        | SMN     |
| 업무구분\_대분류 | NRV     |
| context_path     | nrv_svc |

## 1. 폴더구조 및 화면명

각 화면의 페이지들은 `/src/pages/nrv` 하위에 있으며, 각 폴더는 `업무구분ID(메뉴)` 단위로 구분된다.\
업무구분ID는 우리은행 메타에 따른다.

| 화면명 | [업무구분_대분류]/[업무구분ID][화면유형][화면세부일련번호] |
| ------ | ---------------------------------------------------------- |

## 2. URL 규칙

| 구조 | http://[domain]/[context_path]/web/[AA L3 코드]/[업무구분_대분류]/[업무구분_중분류]/[화면ID] |
| ---- | -------------------------------------------------------------------------------------------- |
| 예시 | http://[domain]/nrv_svc/web/smn/nrv/idcr/NRVIDCRM00                                          |

# Custom Hooks

## 1. 통신 : useApi

---

- BXM 과의 통신을 위한 Hook 으로 2개의 함수를 제공한다.

```javascript
const { callApi, uploadFile } = useApi()
```

### a. callApi(option)

- `axios`를 이용하여 BXM 과 통신하며, 직접 axios를 이용하여 통신하는 것은 지양하도록 한다.
- Content-Type : `application/json;charset=UTF-8`

#### option

```javascript
{
  showLoadingBar: true,     // API 요청시 로딩바 노출 여부 (default: false)
  url: '/tpg/smp/smp/ccs/dbSample',
  data: { },                // Response Body 로 전달될 요청 데이터
  timeout : 70000,          // 타임아웃 (default : 70000)
  success : (response, reqData) => {}, // 성공 콜백
  error : (err) => {}       // 에러 콜백
}
```

#### Example

```javascript
callApi({
  showLoadingBar: true,
  url: '/tpg/smp/smp/ccs/dbSample',
  data: {
    DNI_REQ_KEY_ID: 'gBowoGe4PeGOlYdK',
  },
  success: (response) => {
    /* 성공시 동작 기재 */
  },
  error: (err) => {
    const { messageId, message } = err

    Dialog.alert({
      title: '통신 실패',
      message: `[${messageId}] ${message}`,
      actionBtn: '확인',
      cancelBtn: '취소',
      actionBtnOnClick: () => {
        Dialog.close()
      },
      cancelBtnOnClick: () => {
        Dialog.close()
      },
    })
  },
})
```

#### 에러 콜백함수 데이터 예시

```json
{
  "timestamp": "2023-12-29 14:25:51.674",
  "path": "/tpg/smp/edu/usr/userinfo/login",
  "status": 500,
  "reason": "Internal Server Error",
  "exception": "wfw.woori.online.exception.app.WFApplicationException",
  "messageId": "EDUE0001",
  "message": "잘못된 입력 정보입니다 [아이디]",
  "addMessage": null
}
```

## 2. 웹뷰(WebView) 초기 호출 시 App 정보 세팅, 언어팩 설정

---

앱에서 WebView를 통해 접근 시 첫 랜더링 페이지에서 거래번호를 갖고 startNrvLnkd를 호출하여 비대면채널ID, 비대면실명확인식별ID, 진입URL, 외부URL, 언어팩 등의 정보를 받아 값을 세팅한다.
각 항목은 `context`로 전역상태로 관리되며 각 컴포넌트들에서 사용가능하도록 Hook 형태로 제공되된다.
언어 설정은 전달받은 언어코드로 언어팩 정보를 요청하여 전달받은 스크립트 값을 addResource 하여 세팅한다.

- useAppInfo

상세 사용은 `InitPageTest.jsx` 또는 `/init-test` 페이지 참고

### a. 앱정보세팅 : useAppInfo

```javascript
const { appInfo, setAppInfo, updateAppInfo } = useAppInfo()
```

AppInfo는 setAppInfo로 값을 세팅하고 appInfo로 값을 조회한다.

App정보 세팅하기1(추가)

```javascript
updateAppInfo({
  REACT_APP_LANG_REQ_KEY_ID: response.LANG_CD,
})
```

App정보 세팅하기2

```javascript
setAppInfo((prev) => ({
  ...prev,
  REACT_APP_LANG_REQ_KEY_ID: response.LANG_CD,
  REACT_APP_THEME_COL_REQ_KEY_ID: response.THEME_CD,
}))
```

app정보 조회하기

```javascript
appInfo.REACT_APP_LANG_REQ_KEY_ID
```

### b. 언어팩세팅 : useTranslation

```javascript
const { t } = useTranslation()
```

언어팩 정보는 초기 페이지 진입 시 세팅하고 사용할 컴포넌트에서 `t('언어코드', 'default')`와 같이 사용한다.

문구 변환하기

```javascript
<Header title={t('CODE', '비대면 실명확인')} />
```

### c. 테마색 세팅

초기 사용자가 설정한 테마 컬러를 context에서 가져와 사용자가 설정한 값으로 사용되는 모든 컴포넌트에 적용한다.
ex) Button, Header

## 3. UI 제어

---

팝업(모달), 다이얼로그(alert, confirm), 토스트 메시지 상태는 `redux` 로 전역관리되며, \
각 컴포넌트들에서 사용 가능하도록 Hook 형태로 제공된다.

- useDialog
- useDimmed
- useLoadingBar
  - 통신 요청중일 경우 로딩바 노출 : `callApi` 를 통해 호출되므로 별도로 사용 X
- usePopup
- useToast

상세 사용은 `UITest.jsx` 또는 `/ui-test` 페이지 참고

### a. 다이얼로그(알림창) : useDialog

```javascript
const { Dialog } = useDialog()
```

Dialog는 3가지 함수를 제공한다.

- alert : 버튼 1개(e.g. 확인)가 제공되는 alert 창
- confirm : 버튼 2개(e.g. 확인 / 취소)가 제공되는 confirm 창
- close : 알림창 닫기

Alert 창 띄우기

```javascript
Dialog.alert({
  title: 'Alert',
  message: '안녕하세요 :D', /* 필수 */
  actionBtn: '확인',
  actionBtnOnClick: () => { /* 필수 */
    Dialog.close()
  },
})

또는

const callbackFunc = () => {...}

Dialog.alert({
  title: 'Alert',
  message: '안녕하세요 :D',       /* 필수 */
  actionBtn: '확인',
  actionBtnOnClick: callbackFunc, /* 필수 */
})
```

Confirm 창 띄우기

```javascript
Dialog.confirm({
  title: 'Confirm' /* 필수 */,
  message: '정말로 삭제하시겠습니까?' /* 필수 */,
  actionBtn: '확인',
  actionBtnOnClick: () => {
    /* 필수 */
    Dialog.close()
  },
  cancelBtnOnClick: () => {
    /* 필수 */
    Dialog.close()
  },
})
```

알림창 닫기

```javascript
Dialog.close()
```

### b. 팝업(모달) : usePopup

```javascript
const { openPopup, closePopup, getParam } = usePopup()
```

상세 사용은 `UITest.jsx` 또는 `/ui-test` 페이지 참고

#### Option

```javascript
{
  "id": "Popup1",         // 고유 ID
  "message":<Popup1 />,   // 팝업으로 보여줄 컴포넌트
  "param": {},            // 팝업으로 전달할 데이터
  "callback": (res) => {} // 팝업이 닫힌 후 실행될 콜백함수
}
```

#### Example - 팝업 열기

- 팝업을 닫을 때 `closePopup` 에서 함께 넘긴 데이터는 `callback` 함수에서 `res.result` 객체에 담겨

```javascript
openPopup({
  id: 'Popup1',
  message: <Popup1 />,
  param: {
    from: '메인페이지로부터',
  },
  callback: (res) => {
    if (res.result?.code) {
      Dialog.alert({
        message: `${res.result.code}`,
        actionBtnOnClick: () => {
          Dialog.close()
        },
      })
    }
  },
})
```

#### Example - 팝업 닫기

- 팝업을 열 때 `openPopup` 에서 지정한 `param` 은 `getParam()` 으로 접근 가능하다.
- `closePopup` 함수 호출 시, 함께 넘기는 데이터는 result 객체에 담겨서 `callback` 함수로 전달된다.

```javascript
// 팝업 : Popup1.jsx
const { openPopup, closePopup, getParam } = usePopup()

useEffect(() => {
  console.log('팝업 파라미터 : ', getParam())
}, [getParam])

<Button
  onClick={() => {
    closePopup({ code: '000', msg: '팝업 1 이 닫힙니다.' })
  }}>
  팝업 1 닫기
</Button>
```

### C. 토스트 : useToast

```javascript
const { title, buttonText, onClick, timer, ToastMsg } = useToast()
```

상세 사용은 `UITest.jsx` 또는 `/ui-test` 페이지 참고

#### Option

```javascript
{
  show: true,          // 해당 값의 유무에 따라 토스트 메시지 노출 여부 결정
  title: '안내 메시지', // 토스트 메시지에 들어갈 텍스트
  buttonText: '',      // 버튼에 들어갈 텍스트, buttonText,onClick 값 여부에 따라 하단에 버튼 생성 결정
  timer: 2000,         // 토스트 메시지 노출 시간(ms)
  onClick: null, // 팝업이 닫힌 후 실행될 함수
}
```

#### Example - 토스트 띄우기

- 토스트가 띄워지고 timer(ms) 초가 지난 후 hide()가 실행되며 종료된다.

```javascript
const { ToastMsg } = useToast()

const toastTest = () => {
  ToastMsg.show({
    title: '토스트 메시지',
    timer: 500,
    toastCallback: () => {
      console.log('toastCallback')
    },
  })
}

<FlexBox>
  <Button onClick={toastTest}>Toast Test/Button>
</FlexBox>
```

### C. Header

```javascript
import { useAppInfo } from '../../common/context'

const { appInfo, headerInfo, updateHandler, updateMessage, updateTitle, updateMultiLangYn, updateHeaderInfo } =
  useAppInfo()

const [headerInfo, setHeaderInfo] = useState({
  backHandler: undefined, // Back 버튼 callback
  message: '', // ClosePopup Message
  messageTitle: '', // ClosePopup Title
  title: '', // Header타이틀
  multiLangYn: 'N', // 다국어버튼 여부
})
```

뒤로가기(Button), Title, 다국어(Button), 닫기(Button)는 `context api`를 통해 전역으로 관리되며,
각 컴포넌트들에서 사용 가능하도록 Hook 형태로 제공된다.

- updateHandler: 뒤로가기 이벤트
- updateMessage: 중도이탈 시 팝업 Message
- updateTitle: 중도이탈 시 팝업 Title
- updateMultiLangYn: 다국어 버튼 여부(진입 페이지에서 multiLangYn값이 Y일 때 다국어 버튼이 생성된다)
- updateHeaderInfo: 위 함수 기능들이 종합되어있는 함수

backHandler에 따라 Header 버튼 생성

- null이 아닌 경우 Back 버튼이 생성
- none이 경우 닫기 버튼 안보임

prgAvlDcntStr

- 중도이탈 시 팝업 Message 내용에 포함될 추가서류 다음에 제출 시 진행 가능 일수

#### Option

```javascript
updateHeaderInfo(testFunc, 'aa', 'bb', 'cc', 'Y')
// testFunc: 뒤로가기 실행되는 callback 이벤트
// 2번째 인자: 중도이탈 시 팝업 Message
// 3번째 인자: 중도이탈 시 팝업 Title
// 4번째 인자: Header Title
// 5번째 인자:
```

```javascript
updateHandler(testFunc)
// 뒤로가기 실행되는 callback 이벤트만 수정
```

```javascript
updateMessage('cc')
//중도이탈 시 팝업 Message만 수정
```

```javascript
updateMultiLangYn('Y')
//다국어 버튼 여부
```

### C. 다른 방법으로 인증하기(BottomSheet)

`NRVCOMNP00` 팝업은 `다른 방법으로 인증하기` 관련 BottomSheet이다.
openBottomSheet 호출 시 param으로 전달한 crtfType값에 따라 선택 리스트가 달라진다.

- crtfType: A : 계좌로 인증하는 페이지에서 호출 시
- crtfType: B : 영상통화로 인증하는 페이지에서 호출 시
- crtfType: C : 안면인식으로 인증하는 페이지에서 호출 시

```javascript
import { useBottomSheet } from '../../../common/hooks/useBottomSheet'

const { openBottomSheet, changeParam } = useBottomSheet()

openBottomSheet({
  id: 'Crtf LisT',
  title: `${t('CODE', `인증방법 선택`)}`,
  message: <NRVCOMNP00 />,
  block: false,
  param: {
    crtfType: 'C',
  },
})
```

## 5. OS 확인

- 안드로이드 : android
- 아이폰/아이패드 등 : ios
- 그외 : other

```javascript
import { useCheckOs } from './useCheckOs'
const os = useCheckOs()
```

## 6. 페이지 이동

**페이지를 이동**하는 경우 ( ASIS : callWidget, callMenu, movePage 등 ) `wooriRouter` 의 `movePage` 함수를 이용하고 직접 `useNavigate` 를 사용하여 페이지를 이동하지 않도록 한다.\

```javascript
const wooriRouter = useRouter()

wooriRouter.movePage('/seco/NRVSECOM00')
OR
wooriRouter.movePage('/seco/NRVSECOM00', { param1: true })
```

`wooriRouter` 의 `move` 함수를 이용하는 경우 ( `move`는 location.href와 같은 기능이다 )

- 비대면 실명확인 서비스 첫 진입 또는 이어하기 경우 ( `movePage` 사용시 Native JSession에 set이 안되는 이슈가 있음 )
- 상단 바 닫기 버튼을 눌러 서비스 중도이탈 시

```javascript
Dialog.confirm({
  title: '알림',
  message: '발급받으신 요청키로 비대면 실명확인을 시작하시겠습니까?',
  actionBtn: '확인',
  actionBtnOnClick: () => {
    var port = window.location.port
    if (port !== '') {
      port = ':' + port
    }
    var hostname = window.location.protocol + '//' + window.location.hostname + port
    wooriRouter.move(hostname + `/nrv_svc/web/smn/nrv/comn/NRVCOMNM00?DNI_ACNM_CNF_REQ_KEY_ID=${result}`)
    Dialog.close()
  },
})
```

`wooriRouter` 의 `closeNrvWithoutConfirm` 함수는 비대면 실명인증 종료이동되는 페이지가 유입채널 별로 다르게 호출되야 할 경우 사용한다.

```javascript
// 0: [비대면센터] WON뱅크, A:WON개인뱅킹, B:WON기업뱅킹, E:뉴WON개인뱅킹
if ('0' === lnkdChnlId || 'A' === lnkdChnlId || 'B' === lnkdChnlId || 'E' === lnkdChnlId) {
  // 업무별(0: WON뱅킹, 3: 원터치, 4: 위비뱅크, A: WON뱅크(D), B: WON기업(D)) 비대면 센터 닫기 함수로 이동
  wooriBridge.closeNrv()
}
// C:[인앱] 모바일웹, 7:[비대면센터] 모바일웹
else if ('C' === lnkdChnlId || '7' === lnkdChnlId) {
  // redirectUrl: 첫 진입 페이지 or 이어하기 페이지
  wooriRouter.move(redirectUrl)
}
```

`wooriRouter` 에서 제공하는 함수 목록

- movePage
- move
- replace
- delayReplace
- closeNrvWithoutConfirm
- closeWbnfs

```javascript
const wooriRouter = {
  movePage : (url, param) => {...}

  move: (url) => {
    window.location.href = url
  },

  /**
   * 외부 URL로 이동한다.
   * (ASIS : movePage 중 외부 URL로 이동시키는 경우)
   *
   * @param {*} url
   */
  replace : (url) => {...}

  /**
   * N ms후에 외부 URL로 이동한다.
   *
   * @param {*} url
   * @param {*} time (ms단위)
   */
  delayReplace : (url, time = 250) => {...}

  /**
   * 확인 과정 없이 바로 비대면 실명인증 종료
   */
  closeNrvWithoutConfirm : () => {...}

  /**
   * 비대면 실명인증 종료
   */
  closeWbnfs: () => {...}
}
```

<br />

---

<br />

## 6. WEB -> APP 연동 인터페이스 정의

---

Native와 연동 시 앱 연동 공통 함수가 정의되어있는 `useBridge`의 `wooriBridge` 함수를 이용한다.

wooriBridge에 정의한 앱 연동 공통 함수에서는 전달받은 함수명으로 callback을 브라우저 전역함수로 생성하고 액션코드와 param을 네이티브에 전달한다.
네이티브 기능 호출 후 전역으로 등록된 콜백함수를 찾아 실행한다.

useBridge

```javascript
const useBridge = () => {
  const wooriBridge = {
    appKiFaceCall_nonFace: function (_param, callBack) {
      var funcName = 'kiFaceCallCallback' // 콜백 함수명
      this.actionPush(
        this.makeActionJson(
          'NRV1001', // 액션코드
          '',        //
          funcName,  // 콜백 함수명
          _param,    // param
        ),
        callBack     // app 연동 이후 실행될 콜백 함수
      )
    },
    ...
  }
}
```

앱 연동 공통 함수를 호출하기 위해 param과 callback 함수를 인자로 전달한다.

```javascript
const wooriBridge = useBridge()

// 콜백함수
const kiFaceCallCallback = (data) => {
    data = decodeURI(data)
    var rtnData = JSON.parse(data)
    if (rtnData.RETURN_CODE === '0000') {
    ...
  }
}

var _param = {
  NF_KEY: nfKey,
  NF_ENC_KEY: seedKey,
  NF_ID_IMAGE_REQUEST_URL: hostname_idFace,
  NF_LINE_IMG_REQUEST_URL: hostname_live,
  NF_COMPARE_YN: compareYn,
  NF_STD_SCORE: stdScore,
}
// param과 실행될 콜백 메서드를 인자로 보낸다.
wooriBridge.appKiFaceCall_nonFace(_param, kiFaceCallCallback)
```

<br />

---

<br />

# Common Utils

## 공통 유틸 목록

- nefunnel.js
- cdp
- SHA256.js
- Base64.js
- StringUtils.js
- DateUtils.js
- yup/index.js
- func/index.js
- public\js\cmn\netfunnel\netfunnel.js

<br>

# cdp

고객 행동 로그를 수집 하는 스크립트이다. /js/cmn/cdp/script.js 파일 존재 유무에 따라 로드 함수가 실행된다.
휴대폰 본인확인 약관 동의 Bottom Sheet와 신분증 정보 입력 화면 제외하고 ntm.js 에서 고객 행동관련 로그를 자동으로 수집한다.

```javascript
useEffect(() => {
  const isScriptLoaded = (src) => {
    return document.querySelector(`script[src*="${src}"]`) !== null
  }

  const loadCDPScript = async () => {
    const scriptSrc = '/js/cmn/cdp/script.js'

    const channelId =
      appInfo.lnkdChnlId === '7' ? process.env.REACT_APP_CHANNEL_ID_M_WEB : process.env.REACT_APP_CHANNEL_ID_APP

    // Load CDP Script
    if (!isScriptLoaded(scriptSrc)) {
      await LoadJSFile(
        '/js/cmn/cdp/script.js?n_cid=' + channelId + '&logging_url=' + process.env.REACT_APP_CDP_DOMAIN,
        'cdp-logging'
      )
    }
  }
  if (location.pathname !== '/comn/NRVCOMNM00') {
    // script load
    loadCDPScript()
  }
}, [appInfo.lnkdChnlId, location.pathname])
```

<br>

```javascript
//휴대폰 본인확인 약관 동의 Bottom Sheet
if (window.Ntm) {
  window.Ntm.Event.fireUserDefined('cdp_cert', {
    INPUT_STEP: 'BOTS',
    CDP_PAGE: 'NFCOM0055',
  })
} else {
  window.addEventListener('NTMREADY', function () {
    window.Ntm.Event.fireUserDefined('cdp_cert', {
      INPUT_STEP: 'BOTS',
      CDP_PAGE: 'NFCOM0055',
    })
  })
}
```

```javascript
//신분증 정보 입력 화면
if (window.Ntm) {
  window.Ntm.Event.fireUserDefined('cdp_cert', {
    INPUT_STEP: 'CHECK',
    CDP_PAGE: 'NFCOM0055',
  })
} else {
  window.addEventListener('NTMREADY', function () {
    window.Ntm.Event.fireUserDefined('cdp_cert', {
      INPUT_STEP: 'CHECK',
      CDP_PAGE: 'NFCOM0055',
    })
  })
}
```

```javascript
//public\index.html
<div id="root" cdp_page="NFCOM0055"></div> //현업 요청 코드
```

<br>

# nefunnel

nefunnel은 서비스에 대한 트래픽을 제어하고 성능향상 과 보안을 위한 솔루션이다.
유량제어가 필요한 컴포넌트에서 nefunnel script 파일을 로드하여 필요한 설정들에 사용한다. ex) 신분증촬영

#### Example - 파일 로드하기

```javascript
// 파일로드
import LoadJSFile from '../../common/utils/loadJSFile'

const onLoadNetFunnelfile = async () => {
  const newScripts = await LoadJSFile('/js/cmn/netfunnel/netfunnel.js')
  if (newScripts) {
    newScripts.onload = function () {
      window.NetFunnel_Action()
    }
  }
}
```

#### Example - netfunnel 스크립트 사용하기

```javascript
// 사용 방법
import LoadJSFile from '../../common/utils/loadJSFile'

newScripts.onload = function () {
  window.NetFunnel_Action(
    { service_id: '${netfunnelServiceId}', action_id: '${netfunnelActionIdApp}' },
    {
      success: function (ev, ret) {
        var msg =
          'success:' +
          ',code=' +
          ret.code +
          ',key=' +
          ret.data.key +
          ',ip=' +
          ret.data.ip +
          ',port=' +
          ret.data.port +
          ',nnext=' +
          ret.data.nnext +
          ',nwait=' +
          ret.data.nwait +
          ',tps=' +
          ret.data.tps +
          ',ttl=' +
          ret.data.ttl
        //console.debug(msg);
        // callAppCamera()
      },
      stop: function (ev, ret) {
        // hideLoading()
        completeNetFunnel()
      },
      error: function (ev, ret) {
        // hideLoading()
        completeNetFunnel()
      },
    }
  )
}

const completeNetFunnel = () => {
  window.NetFunnel_Complete()
  if (true) {
    // if ('Y' == appCameraYn) {
    if ('Y' == '${netfunnelAppYn}') {
      window.NetFunnel_Complete()
      console.log()
    }
  } else {
    if ('Y' == '${netfunnelYn}') {
      window.NetFunnel_Complete()
    }
  }
}
```
