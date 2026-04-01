<template>
  <div class="app-shell">
    <div class="screen-glow"></div>
    <main class="card">
      <header class="header">
        <span class="badge">DEMO ONLY</span>
        <h1>링크 송금받기 데모</h1>
        <p>카카오톡처럼 링크는 공유되지만 실제 송금은 일어나지 않습니다.</p>
      </header>

      <section v-if="isPayPage" class="panel">
        <div v-if="payState === 'not-found'" class="status error">
          <h2>유효하지 않은 링크</h2>
          <p>링크가 없거나 만료되었습니다. 새 링크를 요청해주세요.</p>
        </div>

        <div v-else-if="payState === 'done'" class="status done">
          <h2>수락 완료</h2>
          <p>가짜 송금 수락이 처리되었습니다.</p>
          <button class="btn secondary" @click="goToCreate">새 링크 만들기</button>
        </div>

        <div v-else class="pay-info">
          <h2>{{ payRequest.receiverName }}님에게 보낼 링크</h2>
          <p class="amount">{{ formatAmount(payRequest.amount) }}원</p>
          <p class="message">메시지: {{ payRequest.message || '없음' }}</p>
          <p class="meta">만료: {{ formatDate(payRequest.expiresAt) }}</p>
          <button class="btn primary" @click="mockAcceptPayment">가짜 송금 수락</button>
          <button class="btn secondary" @click="goToCreate">나도 링크 만들기</button>
        </div>
      </section>

      <section v-else class="panel">
        <h2>송금 링크 생성</h2>
        <form class="form" @submit.prevent="createPaymentLink">
          <label>
            받는 사람 이름
            <input v-model.trim="form.receiverName" type="text" maxlength="20" required />
          </label>
          <label>
            금액 (원)
            <input
              v-model.number="form.amount"
              type="number"
              min="1000"
              step="1000"
              required
            />
          </label>
          <label>
            메시지
            <input v-model.trim="form.message" type="text" maxlength="60" placeholder="예: 커피값 고마워!" />
          </label>
          <label>
            만료 시간 (시간)
            <input v-model.number="form.expireHours" type="number" min="1" max="168" required />
          </label>
          <button class="btn primary" type="submit">링크 만들기</button>
        </form>

        <div v-if="latestLink" class="result">
          <p>생성된 링크</p>
          <a :href="latestLink" class="link">{{ latestLink }}</a>
          <div class="actions">
            <button class="btn secondary" @click="copyLink">링크 복사</button>
            <button class="btn secondary" @click="shareBySystem">공유하기</button>
            <button class="btn kakao" @click="sendFoolMessage">카카오톡으로 공유하기</button>
          </div>
          <p v-if="kakaoError" class="kakao-error">{{ kakaoError }}</p>
        </div>

        <div v-if="history.length" class="history">
          <h3>최근 생성 링크</h3>
          <ul>
            <li v-for="item in history" :key="item.token">
              <a :href="buildPayUrl(item.token)">{{ item.receiverName }} · {{ formatAmount(item.amount) }}원</a>
              <span>{{ item.status }}</span>
            </li>
          </ul>
        </div>
      </section>
    </main>
  </div>
</template>

<script>
const STORAGE_KEY = 'mock-payment-links-v1'
const KAKAO_JS_KEY = process.env.KAKAO_JS_KEY || ''
const KAKAO_TEMPLATE_ID = Number(process.env.KAKAO_TEMPLATE_ID || 0)
const KAKAO_SDK_URL = 'https://developers.kakao.com/sdk/js/kakao.js'

function randomHex(size) {
  if (window.crypto && window.crypto.getRandomValues) {
    const bytes = new Uint8Array(size)
    window.crypto.getRandomValues(bytes)
    return Array.from(bytes, (value) => value.toString(16).padStart(2, '0')).join('')
  }
  return Math.random().toString(16).slice(2) + Date.now().toString(16)
}

function parsePathToken() {
  const parts = window.location.pathname.split('/').filter(Boolean)
  if (parts.length === 2 && parts[0] === 'pay') return parts[1]
  const queryToken = new URLSearchParams(window.location.search).get('token')
  return queryToken || ''
}

export default {
  name: 'App',
  data() {
    return {
      form: {
        receiverName: '',
        amount: 10000,
        message: '',
        expireHours: 24
      },
      latestLink: '',
      history: [],
      isPayPage: false,
      payState: 'active',
      payRequest: null,
      kakaoReady: false,
      kakaoError: ''
    }
  },
  async mounted() {
    this.history = this.loadLinks()
    const token = parsePathToken()
    this.isPayPage = Boolean(token)
    if (this.isPayPage) this.loadPayRequest(token)
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
        if (!window.Kakao) throw new Error('SDK 로드 실패')
        if (!window.Kakao.isInitialized()) {
          window.Kakao.init(KAKAO_JS_KEY)
        }
        this.kakaoReady = true
      } catch (error) {
        this.kakaoError = '카카오 SDK 초기화에 실패했습니다. 도메인/앱키 설정을 확인해주세요.'
      }
    },
    loadLinks() {
      try {
        return JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')
      } catch (error) {
        return []
      }
    },
    saveLinks(links) {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(links))
      this.history = links
    },
    buildPayUrl(token) {
      return `${window.location.origin}/pay/${token}`
    },
    createPaymentLink() {
      if (!this.form.receiverName || !this.form.amount || this.form.amount < 1000) return
      const token = randomHex(16)
      const expiresAt = Date.now() + this.form.expireHours * 60 * 60 * 1000
      const newItem = {
        token,
        receiverName: this.form.receiverName,
        amount: Number(this.form.amount),
        message: this.form.message,
        expiresAt,
        status: 'PENDING',
        createdAt: Date.now()
      }
      const nextLinks = [newItem, ...this.loadLinks()].slice(0, 20)
      this.saveLinks(nextLinks)
      this.latestLink = this.buildPayUrl(token)
    },
    loadPayRequest(token) {
      const target = this.loadLinks().find((item) => item.token === token)
      if (!target) {
        this.payState = 'not-found'
        return
      }
      if (Date.now() > target.expiresAt) {
        this.payState = 'not-found'
        return
      }
      if (target.status === 'DONE') {
        this.payState = 'done'
      } else {
        this.payState = 'active'
      }
      this.payRequest = target
    },
    mockAcceptPayment() {
      if (!this.payRequest) return
      const links = this.loadLinks().map((item) => {
        if (item.token !== this.payRequest.token) return item
        return { ...item, status: 'DONE' }
      })
      this.saveLinks(links)
      this.payRequest = { ...this.payRequest, status: 'DONE' }
      this.payState = 'done'
    },
    async copyLink() {
      if (!this.latestLink) return
      await navigator.clipboard.writeText(this.latestLink)
      window.alert('링크를 복사했습니다. 카카오톡에 붙여넣어 공유하세요.')
    },
    async shareBySystem() {
      if (!this.latestLink) return
      if (navigator.share) {
        await navigator.share({
          title: '송금 요청 링크 (데모)',
          text: '실제 송금은 되지 않는 데모 링크입니다.',
          url: this.latestLink
        })
        return
      }
      await this.copyLink()
    },
    async sendFoolMessage() {
      this.kakaoError = ''
    
      if (!this.kakaoReady) {
        await this.ensureKakaoSdk()
      }
      if (!window.Kakao || !window.Kakao.Link) {
        this.kakaoError = '카카오 SDK를 찾을 수 없습니다.'
        return
      }
      try {
        window.Kakao.Link.sendCustom({
          templateId: KAKAO_TEMPLATE_ID
        })
      } catch (error) {
        this.kakaoError = '카카오톡 공유 실패: 템플릿 ID 또는 도메인 설정을 확인해주세요.'
      }
    },
    goToCreate() {
      window.location.href = window.location.origin
    },
    formatAmount(value) {
      return new Intl.NumberFormat('ko-KR').format(value || 0)
    },
    formatDate(value) {
      return new Intl.DateTimeFormat('ko-KR', {
        year: 'numeric',
        month: '2-digit',
        day: '2-digit',
        hour: '2-digit',
        minute: '2-digit'
      }).format(value)
    }
  }
}
</script>

<style>
:root {
  --bg: #f5efe5;
  --ink: #2f2721;
  --ink-soft: #685d54;
  --line: #d6c9bb;
  --card: rgba(255, 251, 245, 0.92);
  --accent: #ff7f3e;
  --accent-deep: #d55814;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
}

.app-shell {
  min-height: 100vh;
  padding: 24px;
  background:
    radial-gradient(circle at 10% 15%, #ffd6a9 0%, transparent 40%),
    radial-gradient(circle at 85% 85%, #ffc2c2 0%, transparent 35%),
    linear-gradient(160deg, #f6f1e8 0%, #fbe8d3 45%, #f5eee1 100%);
  display: grid;
  place-items: center;
  position: relative;
  overflow: hidden;
  font-family: "Noto Sans KR", "Apple SD Gothic Neo", sans-serif;
  color: var(--ink);
}

.screen-glow {
  position: absolute;
  width: min(60vw, 620px);
  height: min(60vw, 620px);
  border-radius: 999px;
  background: rgba(255, 127, 62, 0.23);
  filter: blur(90px);
  animation: drift 8s ease-in-out infinite;
}

.card {
  width: min(100%, 760px);
  background: var(--card);
  border: 1px solid var(--line);
  border-radius: 24px;
  box-shadow: 0 20px 60px rgba(103, 83, 62, 0.15);
  padding: 30px;
  position: relative;
  z-index: 1;
  animation: rise 0.55s ease;
}

.header h1 {
  margin: 10px 0 8px;
  font-size: clamp(26px, 4vw, 38px);
}

.header p {
  margin: 0;
  color: var(--ink-soft);
}

.badge {
  display: inline-block;
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.06em;
  color: #fff;
  background: var(--accent-deep);
  padding: 6px 10px;
  border-radius: 999px;
}

.panel {
  margin-top: 24px;
}

.form {
  display: grid;
  gap: 14px;
}

label {
  display: grid;
  gap: 8px;
  font-weight: 600;
}

input {
  width: 100%;
  padding: 12px 14px;
  border: 1px solid var(--line);
  border-radius: 12px;
  font-size: 15px;
  background: #fff;
}

input:focus {
  outline: 2px solid rgba(255, 127, 62, 0.3);
  border-color: var(--accent);
}

.btn {
  border: 0;
  border-radius: 12px;
  padding: 12px 16px;
  font-weight: 700;
  cursor: pointer;
  font-size: 15px;
}

.btn.primary {
  background: var(--accent);
  color: #fff;
}

.btn.primary:hover {
  background: #f16f2d;
}

.btn.secondary {
  background: #efe4d7;
  color: var(--ink);
}

.btn.kakao {
  background: #fee500;
  color: #191919;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.result {
  margin-top: 20px;
  border: 1px dashed #d4b89d;
  border-radius: 14px;
  padding: 14px;
  background: #fff6ec;
}

.result p {
  margin: 0 0 8px;
  font-size: 14px;
  color: var(--ink-soft);
}

.kakao-error {
  margin: 12px 0 0;
  color: #b62600 !important;
  font-weight: 600;
}

.link {
  display: block;
  margin-bottom: 12px;
  color: #204d8a;
  word-break: break-all;
}

.history {
  margin-top: 22px;
}

.history ul {
  list-style: none;
  margin: 10px 0 0;
  padding: 0;
  display: grid;
  gap: 10px;
}

.history li {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  padding: 10px 12px;
  border: 1px solid var(--line);
  border-radius: 12px;
  background: #fffcf7;
}

.history span {
  font-size: 13px;
  font-weight: 700;
  color: var(--ink-soft);
}

.status {
  padding: 22px;
  border-radius: 14px;
  border: 1px solid var(--line);
}

.status.error {
  background: #fff2ef;
  border-color: #f5c3bb;
}

.status.done {
  background: #eef9ef;
  border-color: #bde3bf;
}

.status h2 {
  margin-top: 0;
}

.status p {
  color: var(--ink-soft);
}

.pay-info {
  border: 1px solid var(--line);
  border-radius: 14px;
  padding: 20px;
  background: #fffdf9;
  display: grid;
  gap: 12px;
}

.pay-info h2 {
  margin: 0;
}

.amount {
  margin: 0;
  font-size: clamp(32px, 5vw, 46px);
  font-weight: 800;
  color: #b94a12;
}

.message,
.meta {
  margin: 0;
  color: var(--ink-soft);
}

@keyframes rise {
  from {
    opacity: 0;
    transform: translateY(14px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes drift {
  0%,
  100% {
    transform: translate(-10px, 0);
  }
  50% {
    transform: translate(18px, -8px);
  }
}

@media (max-width: 640px) {
  .app-shell {
    padding: 14px;
  }

  .card {
    padding: 20px;
    border-radius: 18px;
  }
}
</style>
