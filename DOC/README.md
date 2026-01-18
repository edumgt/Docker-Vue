# GH Action -> AWS ECR
```
이 워크플로우에서 쓰는 ${{ secrets.XXX }} 값들은 GitHub 리포지토리(또는 조직) 설정에 “Secrets”로 만들어야 합니다.
워크플로우 파일 안에서 만드는 게 아니라, GitHub UI에서 등록해요.

1) 어디에 만드나? (가장 일반: Repo Secrets)

GitHub에서 해당 리포지토리 들어가기

Settings

왼쪽 메뉴 Secrets and variables → Actions

New repository secret 클릭

아래 이름/값을 각각 추가

이 워크플로우가 요구하는 Secrets 목록

AWS_REGION (예: ap-northeast-2)

AWS_ACCOUNT_ID (예: 123456789012)

ECR_FRONTEND_REPOSITORY (예: my-frontend)

ECR_SIGNALING_REPOSITORY (예: my-signaling)

AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY
```
### IAM 및 액세스키 생성
![alt text](0.png)

### Github 의 설정
![alt text](1.png)
![alt text](2.png)
![alt text](3.png)
![alt text](4.png)

### Action 진행 사항
![alt text](5.png)

### ECR 에 Repo 생성 확인
![alt text](6.png)

### Elastic Beantalk
```
A. 먼저 “프론트용 환경” 1개 만들기 (추천: my-app-frontend)
1) (이미 선택됨) 환경 티어 = 웹 서버 환경

지금처럼 웹 서버 환경 체크 그대로 두세요.

2) 애플리케이션 이름 = my-app

이미 입력하신 my-app 그대로 OK.

3) 아래쪽에 있는 환경 정보에서

환경 이름: my-app-frontend (예시)

도메인: 자동으로 잡히는 값(충돌 나면 다른 걸로)

환경 이름은 나중에 변경이 번거로우니, 프론트/시그널링 구분되게 이름을 추천합니다.

4) 다음(2단계)으로 이동
B. 2단계(서비스 액세스 구성)에서 “Docker 플랫폼” 선택
5) 플랫폼(Platform)

Docker 선택

가능하면 Amazon Linux 2023 running Docker(또는 최신 Docker 플랫폼) 선택

6) 환경 유형(비용 줄이려면)

처음 테스트면 단일 인스턴스(Single instance) 추천

운영이면 로드 밸런싱(Load balanced)

7) 역할(Role) 관련(중요)

여기 단계나 다음 단계에서 역할을 고르게 되는데:

EC2 인스턴스 프로파일 역할(예: aws-elasticbeanstalk-ec2-role)이 있어야 하고

이 역할에 ECR pull 권한이 있어야 합니다.

가장 쉬운 방법: 역할에 AmazonEC2ContainerRegistryReadOnly 정책 붙이기

이거 안 하면 배포할 때 “ECR에서 이미지 pull 실패”가 자주 납니다.
```
![alt text](image.png)
![alt text](image-1.png)
![alt text](image-2.png)

![alt text](image-3.png)


### aws-elasticbeanstalk-service-role 의 경우 environment
### aws-elasticbeanstalk-ec2-role 의 경우 compute
![alt text](image-5.png)

![alt text](image-6.png)

### Role name: aws-elasticbeanstalk-service-role (추천)
![alt text](image-7.png)
![alt text](image-8.png)

### 권한 추가 - AmazonEC2ContainerRegistryReadOnly
![alt text](image-9.png)
![alt text](image-10.png)

### 최종 권한 확인
![alt text](image-16.png)
![alt text](image-11.png)

### 다시 위의 과정
![alt text](image-12.png)
![alt text](image-13.png)
![alt text](image-14.png)

### 이 화면에서 role 불러오는데 시간걸림
![alt text](image-15.png)
![alt text](image-17.png)
![alt text](image-23.png)

### 아래와 같이 compute , service 의 2종류 role 에 대해 혼동이 많으니 상당한 주의가 필요함
![alt text](image-18.png)

### 사용하는 기본 VPC 환경
![alt text](image-19.png)

### 인스탄스 선택 - 기본으로
![alt text](image-20.png)

### 구성 중 - 최대 10분정도 대기
![alt text](image-22.png)

### 최종 화면
![alt text](image-24.png)

### 경고 있을 경우 확인
![alt text](image-25.png)

### role 가장 큰 문제로, 다음 작업이 필요할 수 있음.
방법 1) CloudShell로 서비스 연결 역할 생성 (가장 확실)
AWS 콘솔 상단에서 CloudShell(>_ 아이콘) 열고 아래 한 줄 실행:
```
aws iam create-service-linked-role --aws-service-name elasticbeanstalk.amazonaws.com
```
성공하면:
AWSServiceRoleForElasticBeanstalk 역할이 생성됩니다.

### Cloud Shell 에서
```
~ $ aws iam get-role --role-name AWSServiceRoleForElasticBeanstalk
{
    "Role": {
        "Path": "/aws-service-role/elasticbeanstalk.amazonaws.com/",
        "RoleName": "AWSServiceRoleForElasticBeanstalk",
        "RoleId": "AROARIBXLWVE3GC4XAKO7",
        "Arn": "arn:aws:iam::086015456585:role/aws-service-role/elasticbeanstalk.amazonaws.com/AWSServiceRoleForElasticBeanstalk",
        "CreateDate": "2026-01-17T12:00:25+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "elasticbeanstalk.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        },
        "MaxSessionDuration": 3600,
        "RoleLastUsed": {}
    }
}
~ $ 
```
### 여전히 권한 문제 시 권한 추가
![alt text](image-26.png)

### 배포
![alt text](image-27.png)


### Dockerrun.aws.json 파일 만들기 - 내용의 
```
{
  "AWSEBDockerrunVersion": 1,
  "Image": {
    "Name": "<Account-ID>.dkr.ecr.ap-northeast-2.amazonaws.com/<ECR_FRONTEND_REPO>:<TAG>",
    "Update": "true"
  },
  "Ports": [
    { "ContainerPort": "80" }
  ]
}
```
### 압축
```bash
zip -r frontend-eb.zip Dockerrun.aws.json
```
![alt text](image-28.png)


### S3 반영 확인
![alt text](image-29.png)

### 각자의 설정에 따라 여러 경우가 발생할 수 있음
![alt text](image-30.png)
```
위의 경우는 S3 버킷 이름이 빈 문자열("")로 들어갔습니다.
즉, GitHub Actions에서 EB_S3_BUCKET 값이 설정되지 않았거나 비어있어서, aws s3api create-bucket --bucket "" 같은 형태로 실행된 겁니다.

1) 가장 먼저 확인할 것 (GitHub Actions Secrets)

GitHub Repo → Settings → Secrets and variables → Actions → Secrets

여기에 아래 값이 정확히 존재해야 합니다.

EB_S3_BUCKET ✅ (현재 이게 비어있음)

(추가로 쓰는 값들) AWS_REGION, AWS_ACCOUNT_ID, EB_APPLICATION_NAME, EB_ENVIRONMENT_NAME 등

✅ 해결: EB_S3_BUCKET에 버킷 이름을 넣으세요. 예:

my-app-eb-deploy-086015456585-ap-northeast-2

S3 버킷 이름 규칙:

전부 소문자 권장

공백 금지

3~63자 권장 (S3 실제 규칙)

예: myapp-eb-bucket-086015456585
```
### 위의 경우 
![alt text](image-31.png)

### 오류 사항
![alt text](image-32.png)
```
EB_APPLICATION_NAME: 빈 값
EB_FRONTEND_ENVIRONMENT_NAME: 빈 값
EB_SIGNALING_ENVIRONMENT_NAME: 빈 값
```
![alt text](image-33.png)
```
  AWS_REGION: ${{ secrets.AWS_REGION }}
  AWS_ACCOUNT_ID: ${{ secrets.AWS_ACCOUNT_ID }}
  ECR_FRONTEND_REPOSITORY: ${{ secrets.ECR_FRONTEND_REPOSITORY }}
  ECR_SIGNALING_REPOSITORY: ${{ secrets.ECR_SIGNALING_REPOSITORY }}

  EB_APPLICATION_NAME: ${{ secrets.EB_APPLICATION_NAME }}
  EB_S3_BUCKET: ${{ secrets.EB_S3_BUCKET }}
  EB_FRONTEND_ENVIRONMENT_NAME: ${{ secrets.EB_FRONTEND_ENVIRONMENT_NAME }}
  EB_SIGNALING_ENVIRONMENT_NAME: ${{ secrets.EB_SIGNALING_ENVIRONMENT_NAME }}
```
### my-app-backend 도 설정
![alt text](image-34.png)

### backend용 Dockerrun.aws.json 예시
```
{
  "AWSEBDockerrunVersion": 1,
  "Image": {
    "Name": "086015456585.dkr.ecr.ap-northeast-2.amazonaws.com/my-signaling:latest",
    "Update": "true"
  },
  "Ports": [
    { "ContainerPort": "3001" }
  ]
}
```
### 각 상황의 로그 분석
![alt text](image-35.png)

### json 이름 잘 바꾸면서 저장 업로드 필요
```
kimdy@DESKTOP-CLQV18N:~/Docker-Vue$ zip -r frontend-eb.zip Dockerrun.aws.json
  adding: Dockerrun.aws.json (deflated 19%)
kimdy@DESKTOP-CLQV18N:~/Docker-Vue$ zip -r backend-eb.zip Dockerrun.aws.json
  adding: Dockerrun.aws.json (deflated 19%)
kimdy@DESKTOP-CLQV18N:~/Docker-Vue$ zip -r frontend-eb.zip Dockerrun.aws.json
updating: Dockerrun.aws.json (deflated 19%)
kimdy@DESKTOP-CLQV18N:~/Docker-Vue$ zip -r frontend-eb.zip Dockerrun.aws.json
updating: Dockerrun.aws.json (deflated 19%)
kimdy@DESKTOP-CLQV18N:~/Docker-Vue$ zip -r backend-eb.zip Dockerrun.aws.json
updating: Dockerrun.aws.json (deflated 19%)
kimdy@DESKTOP-CLQV18N:~/Docker-Vue$ 
```


---
---
## Vite + Vue3 + TailwindCSS v3 기반 WebRTC 로컬 테스트 모듈 (영상통화 + 채팅 + 화면공유 + 파일전송)

> 목표: **localhost**에서 “두 브라우저(또는 시크릿 탭)”로 쉽게 P2P WebRTC 기능을 테스트할 수 있는 최소/실전형 예제  
> 구성:  
> - **Vue 클라이언트**: `App.vue` (WebRTC + UI)  
> - **Signaling 서버**: `signal.js` (Node + ws) — offer/answer/ICE 교환용  
> - **TailwindCSS v3**: UI 빠르게 스타일링

---
 
## 1) 폴더 구조 (예시)

```
my-webrtc-app/
 ├─ signal.js                 # WebSocket signaling 서버
 ├─ package.json              # (Vue 프로젝트) 의존성
 ├─ index.html
 ├─ postcss.config.js
 ├─ tailwind.config.js
 ├─ vite.config.js
 └─ src/
    ├─ App.vue                # ✅ 아래의 “버그 수정 완료” 코드
    ├─ main.js
    └─ index.css
```

> `signal.js`는 Vue 프로젝트 루트에 둬도 되고, `server/` 같은 폴더로 분리해도 됩니다.

---

## 2) 설치 (Tailwind 3 버전 고정)

### 2-1) Vite + Vue 프로젝트 생성

```bash
npm create vite@latest my-webrtc-app -- --template vue
cd my-webrtc-app
npm install
```

### 2-2) TailwindCSS v3 고정 설치

```bash
npm install -D tailwindcss@3 postcss@8 autoprefixer@10
npx tailwindcss init -p
```

> 만약 설치가 꼬였으면(무한 설치처럼 보이거나 의존성 충돌) 아래 초기화 후 재시도:
```bash
# Windows PowerShell에서도 동작
rmdir /s /q node_modules 2>nul
del package-lock.json 2>nul
npm cache clean --force
npm install
npm install -D tailwindcss@3 postcss@8 autoprefixer@10 --legacy-peer-deps
```

---

## 3) Tailwind 설정

### 3-1) `tailwind.config.js`

```js
export default {
  content: ["./index.html", "./src/**/*.{vue,js,ts}"],
  theme: { extend: {} },
  plugins: [],
}
```

### 3-2) `src/index.css`

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### 3-3) `src/main.js`

```js
import { createApp } from "vue"
import "./index.css"
import App from "./App.vue"

createApp(App).mount("#app")
```

---

## 4) Signaling 서버 (Node + ws)

### 4-1) 서버 의존성 설치

Vue 프로젝트 루트에서:

```bash
npm install ws
```

### 4-2) `signal.js` (ESM import 버전)

> Node v22에서 기본적으로 잘 동작합니다. (`import` 사용)

```js
// signal.js
import { WebSocketServer } from "ws"

const wss = new WebSocketServer({ port: 3001 })

wss.on("connection", (ws) => {
  ws.on("message", (message) => {
    // 모든 클라이언트에 브로드캐스트
    // (클라이언트에서 sender ID로 자기 자신 메시지는 무시)
    for (const client of wss.clients) {
      if (client.readyState === 1) client.send(message.toString())
    }
  })
})

console.log("✅ Signaling server running on ws://localhost:3001")
```

> 만약 CommonJS(require)로 쓰고 싶다면:
```js
// signal.cjs
const { WebSocketServer } = require("ws")
```

---

## 5) ✅ App.vue (채팅 송수신 버그 수정 + 로컬 테스트 편의 고도화)

아래 코드는 다음을 반영합니다.

- **채팅 안 보이던 버그 수정**
  - `pc.ondatachannel`에서 **chatChannel/fileChannel을 변수에 저장**해야 양쪽에서 전송 가능
  - `sendChat()`는 `readyState === "open"` 확인
- **localhost 테스트 편의**
  - WebSocket은 `onMounted()`에 **1번만 연결**
  - signaling 메시지에 `sender`를 넣고 **자기 자신 메시지 무시**
- **ICE 서버(STUN) 기본값 추가**
  - 로컬에서도 대부분 문제 없지만, 실제 환경에 가까운 테스트에 도움

> ⚠️ 현재 흐름은 **두 브라우저(또는 시크릿 탭)에서 모두 `Call`을 눌러야** 연결이 완료되는 형태입니다.  
> (상대방이 Call을 누르지 않으면 offer를 받아도 처리하지 않습니다. “초대/수락” 모델로 바꾸는 건 다음 단계에서 분리 가능합니다.)

```vue
<template>
  <div class="min-h-screen bg-gray-100 p-6">
    <div class="mx-auto max-w-5xl space-y-4">
      <header class="flex items-center justify-between">
        <div>
          <h1 class="text-xl font-bold">WebRTC Local Test</h1>
          <p class="text-sm text-gray-600">
            영상통화 · 채팅 · 화면공유 · 파일전송 (localhost)
          </p>
        </div>

        <div class="text-xs text-gray-600 space-y-1 text-right">
          <div>
            WS:
            <span :class="wsReady ? 'text-green-700' : 'text-red-700'">
              {{ wsReady ? "connected" : "disconnected" }}
            </span>
          </div>
          <div>
            RTC:
            <span class="text-gray-800">{{ pcState || "-" }}</span>
          </div>
        </div>
      </header>

      <!-- 비디오 -->
      <section class="grid grid-cols-1 gap-4 md:grid-cols-2">
        <div class="rounded-lg border bg-white p-3 shadow">
          <div class="mb-2 flex items-center justify-between">
            <h2 class="font-semibold">Local</h2>
            <span class="text-xs text-gray-500">muted</span>
          </div>
          <video
            ref="localVideo"
            autoplay
            playsinline
            muted
            class="aspect-video w-full rounded bg-black"
          ></video>
        </div>

        <div class="rounded-lg border bg-white p-3 shadow">
          <div class="mb-2 flex items-center justify-between">
            <h2 class="font-semibold">Remote</h2>
            <span class="text-xs text-gray-500">audio on</span>
          </div>
          <video
            ref="remoteVideo"
            autoplay
            playsinline
            class="aspect-video w-full rounded bg-black"
          ></video>
        </div>
      </section>

      <!-- 컨트롤 -->
      <section class="flex flex-wrap items-center gap-2">
        <button
          @click="startCall"
          class="rounded bg-blue-600 px-4 py-2 text-white hover:bg-blue-700 disabled:opacity-50"
          :disabled="calling"
        >
          📞 Call
        </button>

        <button
          @click="shareScreen"
          class="rounded bg-green-600 px-4 py-2 text-white hover:bg-green-700 disabled:opacity-50"
          :disabled="!pc"
        >
          🖥 화면공유
        </button>

        <button
          @click="hangup"
          class="rounded bg-red-600 px-4 py-2 text-white hover:bg-red-700"
        >
          ❌ Hangup
        </button>

        <div class="ml-auto flex items-center gap-2 text-sm text-gray-700">
          <span class="font-semibold">Tip:</span>
          <span>크롬 + 시크릿 탭(또는 엣지) 2개로 테스트</span>
        </div>
      </section>

      <!-- 채팅 + 파일 -->
      <section class="grid grid-cols-1 gap-4 md:grid-cols-2">
        <!-- 채팅 -->
        <div class="rounded-lg border bg-white p-3 shadow">
          <h3 class="mb-2 font-bold">💬 Chat</h3>

          <div ref="chatBox" class="h-44 overflow-y-auto rounded border bg-gray-50 p-2">
            <div v-for="(msg, i) in messages" :key="i" class="text-sm leading-6">
              {{ msg }}
            </div>
          </div>

          <div class="mt-2 flex gap-2">
            <input
              v-model="chatInput"
              @keyup.enter="sendChat"
              placeholder="메시지 입력"
              class="w-full rounded border px-3 py-2 text-sm"
            />
            <button
              @click="sendChat"
              class="shrink-0 rounded bg-gray-900 px-3 py-2 text-sm text-white disabled:opacity-50"
              :disabled="!chatChannelOpen"
            >
              전송
            </button>
          </div>

          <p class="mt-2 text-xs text-gray-500">
            채널 상태: {{ chatChannelOpen ? "open" : "closed" }}
          </p>
        </div>

        <!-- 파일 -->
        <div class="rounded-lg border bg-white p-3 shadow">
          <h3 class="mb-2 font-bold">📂 File Transfer</h3>

          <input type="file" @change="sendFile" class="mb-2 block w-full text-sm" />

          <div class="text-xs text-gray-500">
            채널 상태: {{ fileChannelOpen ? "open" : "closed" }}
          </div>

          <ul class="mt-3 space-y-1">
            <li v-for="(file, i) in receivedFiles" :key="i" class="text-sm">
              <a :href="file.url" :download="file.name" class="text-blue-600 underline">
                {{ file.name }}
              </a>
            </li>
          </ul>

          <p class="mt-2 text-xs text-gray-500">
            참고: 큰 파일은 청크 전송(분할)이 필요합니다. 이 예제는 로컬 소형 파일 테스트용입니다.
          </p>
        </div>
      </section>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from "vue"

const localVideo = ref(null)
const remoteVideo = ref(null)
const chatBox = ref(null)

let localStream = null
let pc = null
let ws = null
let chatChannel = null
let fileChannel = null

const clientId = Math.random().toString(36).slice(2, 10)

// UI 상태
const messages = ref([])
const chatInput = ref("")
const receivedFiles = ref([])
const wsReady = ref(false)
const pcState = ref("")
const calling = ref(false)

// WebRTC 설정 (STUN 추가)
const rtcConfig = {
  iceServers: [{ urls: "stun:stun.l.google.com:19302" }],
}

const chatChannelOpen = computed(() => chatChannel?.readyState === "open")
const fileChannelOpen = computed(() => fileChannel?.readyState === "open")

onMounted(() => {
  initWebSocket()
})

onBeforeUnmount(() => {
  try { ws?.close() } catch {}
  hangup()
})

function initWebSocket() {
  ws = new WebSocket("ws://localhost:3001")

  ws.onopen = () => {
    wsReady.value = true
    console.log("✅ WebSocket Connected")
  }

  ws.onclose = () => {
    wsReady.value = false
    console.log("⚠️ WebSocket Closed")
  }

  ws.onerror = (e) => {
    wsReady.value = false
    console.log("❌ WebSocket Error", e)
  }

  ws.onmessage = async (event) => {
    const data = JSON.parse(event.data)

    // 자기 자신이 보낸 signaling 무시
    if (data.sender === clientId) return

    // 이 예제는 “양쪽 모두 Call” 방식이므로 pc가 없으면 일단 무시
    if (!pc) return

    try {
      if (data.type === "offer") {
        await pc.setRemoteDescription(data)
        const answer = await pc.createAnswer()
        await pc.setLocalDescription(answer)
        ws.send(JSON.stringify({ ...answer, sender: clientId }))
      } else if (data.type === "answer") {
        await pc.setRemoteDescription(data)
      } else if (data.type === "candidate" && data.candidate) {
        await pc.addIceCandidate(data.candidate)
      }
    } catch (err) {
      console.error("❌ signaling 처리 오류:", err)
    }
  }
}

async function startCall() {
  if (!ws || ws.readyState !== 1) {
    alert("WebSocket(signaling)이 연결되지 않았습니다. signal.js 서버를 먼저 실행하세요.")
    return
  }

  calling.value = true
  messages.value.push("시도: Call 시작...")

  try {
    localStream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true })
    localVideo.value.srcObject = localStream

    pc = new RTCPeerConnection(rtcConfig)
    pcState.value = pc.connectionState || ""

    pc.onconnectionstatechange = () => {
      pcState.value = pc.connectionState
    }

    // Media track 추가
    localStream.getTracks().forEach((track) => pc.addTrack(track, localStream))

    pc.ontrack = (event) => {
      remoteVideo.value.srcObject = event.streams[0]
    }

    // Caller: DataChannel 생성
    chatChannel = pc.createDataChannel("chat", { ordered: true })
    fileChannel = pc.createDataChannel("file", { ordered: true })

    setupChatChannel(chatChannel)
    setupFileChannel(fileChannel)

    // Callee: DataChannel 수신
    pc.ondatachannel = (event) => {
      if (event.channel.label === "chat") {
        chatChannel = event.channel
        setupChatChannel(chatChannel)
      }
      if (event.channel.label === "file") {
        fileChannel = event.channel
        setupFileChannel(fileChannel)
      }
    }

    pc.onicecandidate = (event) => {
      if (event.candidate) {
        ws.send(JSON.stringify({ type: "candidate", candidate: event.candidate, sender: clientId }))
      }
    }

    const offer = await pc.createOffer()
    await pc.setLocalDescription(offer)
    ws.send(JSON.stringify({ ...offer, sender: clientId }))

    messages.value.push("Offer 전송 완료. (상대도 Call을 눌러 Answer를 보내야 연결됩니다.)")
  } catch (e) {
    console.error(e)
    alert("Call 시작 실패: 브라우저 권한(카메라/마이크)과 HTTPS/localhost 여부를 확인하세요.")
  } finally {
    calling.value = false
  }
}

function setupChatChannel(channel) {
  channel.onopen = () => {
    messages.value.push("채팅 채널 open")
    scrollChatBottom()
  }

  channel.onmessage = (e) => {
    messages.value.push(`상대방: ${e.data}`)
    scrollChatBottom()
  }

  channel.onclose = () => {
    messages.value.push("채팅 채널 close")
    scrollChatBottom()
  }
}

function setupFileChannel(channel) {
  let receivedBuffer = []
  let fileName = ""

  channel.onmessage = async (event) => {
    // 첫 메시지를 “파일명”으로 받는 단순 프로토콜
    if (typeof event.data === "string") {
      fileName = event.data
      receivedBuffer = []
      return
    }

    // ArrayBuffer(또는 Blob) 수신 처리
    if (event.data instanceof Blob) {
      receivedBuffer.push(await event.data.arrayBuffer())
    } else {
      receivedBuffer.push(event.data)
    }

    // 이 예제는 “한 번에 1개 버퍼”만 보내는 단순 방식이므로,
    // 버퍼가 하나 들어오면 파일로 완성시킵니다.
    const blob = new Blob(receivedBuffer)
    const url = URL.createObjectURL(blob)
    receivedFiles.value.push({ name: fileName || "received.bin", url })

    fileName = ""
    receivedBuffer = []
  }
}

function sendChat() {
  const text = chatInput.value.trim()
  if (!text) return

  if (chatChannel?.readyState === "open") {
    chatChannel.send(text)
    messages.value.push(`나: ${text}`)
    chatInput.value = ""
    scrollChatBottom()
  } else {
    alert("채팅 채널이 아직 열리지 않았습니다. (상대와 연결이 완료되었는지 확인)")
  }
}

function sendFile(event) {
  const file = event.target.files?.[0]
  if (!file) return

  if (!fileChannel || fileChannel.readyState !== "open") {
    alert("파일 채널이 아직 열리지 않았습니다.")
    return
  }

  // 1) 파일명 전송
  fileChannel.send(file.name)

  // 2) 파일 데이터 전송 (로컬 소형 파일 테스트용: 통째로 전송)
  file.arrayBuffer().then((buffer) => {
    fileChannel.send(buffer)
  })

  // 같은 파일 다시 선택 가능하도록 reset
  event.target.value = ""
}

async function shareScreen() {
  if (!pc) {
    alert("먼저 Call로 연결을 시작하세요.")
    return
  }

  const screenStream = await navigator.mediaDevices.getDisplayMedia({ video: true })
  const screenTrack = screenStream.getTracks()[0]

  const sender = pc.getSenders().find((s) => s.track && s.track.kind === "video")
  if (!sender) {
    alert("비디오 sender를 찾지 못했습니다.")
    return
  }

  await sender.replaceTrack(screenTrack)

  // 화면공유 종료 시 카메라로 복귀
  screenTrack.onended = async () => {
    const camTrack = localStream?.getTracks()?.find((t) => t.kind === "video")
    if (camTrack) await sender.replaceTrack(camTrack)
  }
}

function hangup() {
  try { chatChannel?.close() } catch {}
  try { fileChannel?.close() } catch {}
  chatChannel = null
  fileChannel = null

  if (pc) {
    try { pc.close() } catch {}
    pc = null
  }

  if (localStream) {
    localStream.getTracks().forEach((t) => t.stop())
    localStream = null
  }

  if (localVideo.value) localVideo.value.srcObject = null
  if (remoteVideo.value) remoteVideo.value.srcObject = null

  pcState.value = ""
  messages.value.push("Hangup 완료")
  scrollChatBottom()
}

function scrollChatBottom() {
  setTimeout(() => {
    if (chatBox.value) chatBox.value.scrollTop = chatBox.value.scrollHeight
  }, 0)
}
</script>
```

---

## 6) 실행 방법 (localhost)

### 6-1) signaling 서버 실행

프로젝트 루트에서:

```bash
node signal.js
# ✅ Signaling server running on ws://localhost:3001
```

### 6-2) Vue 앱 실행

```bash
npm run dev
# http://localhost:5173
```

### 6-3) 테스트 시나리오

1) 브라우저 2개(예: 크롬 + 엣지, 또는 크롬 + 시크릿 탭)에서  
   `http://localhost:5173` 접속

2) **양쪽 모두** `📞 Call` 버튼 클릭  
   - 한쪽이 offer 전송  
   - 다른 쪽이 answer 전송  
   - ICE candidate 교환되며 연결

3) 연결되면:
- 영상/음성: Local/Remote 영상 표시
- 채팅: 양쪽에서 입력하면 서로 표시
- 파일: 작은 파일 업로드 → 상대에게 링크 생성
- 화면공유: `🖥 화면공유` 클릭 → 상대 화면이 공유 화면으로 바뀜 (종료 시 카메라 복귀)

---

## 7) 자주 겪는 문제/체크리스트

### A) `Cannot find package 'ws' ...`
```bash
npm install ws
```

### B) 카메라/마이크 권한 문제
- 브라우저 주소창 왼쪽의 “자물쇠/권한”에서 카메라/마이크 허용
- 로컬 테스트는 `http://localhost`는 대체로 허용되지만, 실제 배포는 **HTTPS** 필수

### C) NAT/방화벽 환경에서 연결 실패
- 로컬은 대부분 STUN만으로 되지만, 실제 서비스는 **TURN 서버**가 필요할 수 있음

### D) 파일 전송이 큰 파일에서 깨짐
- DataChannel은 큰 파일을 **청크 분할 전송**해야 안정적
- 이 예제는 “로컬 소형 파일” 검증용

---

## 8) 다음 고도화 아이디어 (원하면 확장 가능)

- “초대/수락” 모델: 한쪽만 Call 누르면 상대는 자동으로 offer 처리 + “수락” 버튼 제공
- 룸(room) 개념: WebSocket 서버에서 roomId로 대상 분리 (현재는 전체 브로드캐스트)
- 파일 전송 청크/진행률 표시 (progress bar)
- 카메라/마이크 on/off 토글
- 연결 상태(ICE/DTLS) 상세 표시 및 로그 패널

---
