<template>
  <div class="app-shell">
    <main class="card">
      <h1>카카오톡 공유</h1>
      <p>카카오 메시지 템플릿으로 바로 공유합니다.</p>
      <button class="btn-kakao" @click="sendKakaoMessage">카카오톡으로 공유하기</button>
      <p v-if="kakaoError" class="kakao-error">{{ kakaoError }}</p>
    </main>
  </div>
</template>

<script>
const KAKAO_JS_KEY = process.env.KAKAO_JS_KEY || ''
const KAKAO_TEMPLATE_ID = Number(process.env.KAKAO_TEMPLATE_ID || 0)
const KAKAO_SDK_URL = 'https://developers.kakao.com/sdk/js/kakao.js'

export default {
  name: 'App',
  data() {
    return {
      kakaoReady: false,
      kakaoError: ''
    }
  },
  async mounted() {
    await this.ensureKakaoSdk()
  },
  methods: {
    async ensureKakaoSdk() {
      if (window.Kakao && window.Kakao.isInitialized && window.Kakao.isInitialized()) {
        this.kakaoReady = true
        return
      }

      try {
        await new Promise((resolve, reject) => {
          const existing = document.querySelector(`script[src="${KAKAO_SDK_URL}"]`)
          if (existing) {
            existing.addEventListener('load', resolve, { once: true })
            existing.addEventListener('error', reject, { once: true })
            if (window.Kakao) resolve()
            return
          }

          const script = document.createElement('script')
          script.src = KAKAO_SDK_URL
          script.onload = resolve
          script.onerror = reject
          document.head.appendChild(script)
        })

        if (!window.Kakao) throw new Error('SDK load failed')
        if (!window.Kakao.isInitialized()) {
          window.Kakao.init(KAKAO_JS_KEY)
        }

        this.kakaoReady = true
      } catch (error) {
        this.kakaoError = '카카오 SDK 초기화에 실패했습니다. 도메인/앱키 설정을 확인해주세요.'
      }
    },
    async sendKakaoMessage() {
      this.kakaoError = ''

      if (!this.kakaoReady) {
        await this.ensureKakaoSdk()
      }

      if (!window.Kakao || !window.Kakao.Link) {
        this.kakaoError = '카카오 SDK를 찾을 수 없습니다.'
        return
      }

      if (!KAKAO_TEMPLATE_ID) {
        this.kakaoError = 'KAKAO_TEMPLATE_ID 환경 변수를 확인해주세요.'
        return
      }

      try {
        window.Kakao.Link.sendCustom({
          templateId: KAKAO_TEMPLATE_ID
        })
      } catch (error) {
        this.kakaoError = '카카오톡 공유 실패: 템플릿 ID 또는 도메인 설정을 확인해주세요.'
      }
    }
  }
}
</script>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
}

.app-shell {
  min-height: 100vh;
  display: grid;
  place-items: center;
  padding: 20px;
  background: #f5f5f5;
  font-family: 'Noto Sans KR', 'Apple SD Gothic Neo', sans-serif;
}

.card {
  width: min(100%, 420px);
  background: #fff;
  border: 1px solid #e5e5e5;
  border-radius: 12px;
  padding: 24px;
  text-align: center;
}

h1 {
  margin: 0 0 8px;
  font-size: 24px;
}

p {
  margin: 0 0 16px;
  color: #555;
}

.btn-kakao {
  width: 100%;
  border: 0;
  border-radius: 10px;
  padding: 12px 14px;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
  background: #fee500;
  color: #191919;
}

.kakao-error {
  margin-top: 12px;
  color: #b62600;
  font-weight: 600;
}
</style>
