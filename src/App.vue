<template>
  <div id="app" class="stage">
    <div class="bg-gradient"></div>
    <div class="stars"></div>
    <div class="orbs">
      <span class="orb o1"></span>
      <span class="orb o2"></span>
      <span class="orb o3"></span>
    </div>
    <div class="rings">
      <span class="ring r1"></span>
      <span class="ring r2"></span>
      <span class="ring r3"></span>
    </div>
    <div class="content">
      <h1 class="title">박종현씨 </h1>
      <p class="subtitle">뭐하고있으세요</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
#app.stage {
  position: relative;
  height: 100vh;
  width: 100%;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #050510;
  color: #fff;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.bg-gradient {
  position: absolute;
  inset: -20vmin;
  background:
    radial-gradient(40vmin 40vmin at 20% 25%, rgba(255, 0, 153, 0.45), transparent 60%),
    radial-gradient(35vmin 35vmin at 80% 30%, rgba(0, 200, 255, 0.40), transparent 60%),
    radial-gradient(45vmin 45vmin at 30% 80%, rgba(255, 255, 0, 0.30), transparent 60%),
    radial-gradient(50vmin 50vmin at 75% 75%, rgba(124, 0, 255, 0.35), transparent 60%),
    linear-gradient(120deg, #0b0b1d 0%, #0a0320 50%, #000 100%);
  filter: saturate(125%);
  animation: hue 16s linear infinite;
}

.stars {
  position: absolute;
  inset: 0;
  background-image:
    radial-gradient(2px 2px at 10% 20%, rgba(255,255,255,0.8), transparent 60%),
    radial-gradient(1px 1px at 30% 80%, rgba(255,255,255,0.7), transparent 60%),
    radial-gradient(1.5px 1.5px at 60% 10%, rgba(255,255,255,0.9), transparent 60%),
    radial-gradient(1.25px 1.25px at 80% 60%, rgba(255,255,255,0.85), transparent 60%),
    radial-gradient(1px 1px at 50% 50%, rgba(255,255,255,0.9), transparent 60%),
    radial-gradient(2px 2px at 15% 65%, rgba(255,255,255,0.75), transparent 60%),
    radial-gradient(1.25px 1.25px at 85% 40%, rgba(255,255,255,0.8), transparent 60%);
  animation: twinkle 3.5s ease-in-out infinite;
  opacity: 0.9;
}

.orbs {
  position: absolute;
  inset: 0;
  pointer-events: none;
}
.orb {
  position: absolute;
  width: 45vmin;
  height: 45vmin;
  border-radius: 50%;
  filter: blur(25px) saturate(140%);
  mix-blend-mode: screen;
  animation: float 10s ease-in-out infinite;
}
.orb.o1 {
  left: -10vmin;
  top: 5vmin;
  background: radial-gradient(circle at 30% 30%, rgba(255, 90, 160, 0.8), rgba(255, 90, 160, 0.05));
  animation-duration: 13s;
}
.orb.o2 {
  right: -8vmin;
  top: 35vmin;
  background: radial-gradient(circle at 70% 30%, rgba(0, 210, 255, 0.8), rgba(0, 210, 255, 0.05));
  animation-duration: 11s;
  animation-delay: -2s;
}
.orb.o3 {
  left: 30vmin;
  bottom: -10vmin;
  background: radial-gradient(circle at 50% 50%, rgba(220, 255, 0, 0.8), rgba(220, 255, 0, 0.05));
  animation-duration: 15s;
  animation-delay: -4s;
}

.rings { position: absolute; inset: 0; pointer-events: none; }
.ring {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  border-radius: 50%;
  border: 2px solid rgba(255,255,255,0.18);
  box-shadow:
    0 0 25px rgba(255,255,255,0.08),
    inset 0 0 60px rgba(255,255,255,0.06);
  backdrop-filter: blur(1px);
}
.ring.r1 { width: 70vmin; height: 70vmin; animation: spin 24s linear infinite; }
.ring.r2 { width: 52vmin; height: 52vmin; animation: spinReverse 18s linear infinite; }
.ring.r3 { width: 34vmin; height: 34vmin; animation: spin 12s linear infinite; }

.content {
  position: relative;
  z-index: 2;
  text-align: center;
}

.title {
  font-size: clamp(54px, 12vmin, 160px);
  line-height: 1.05;
  margin: 0;
  letter-spacing: 2px;
  background: linear-gradient( -45deg,
    #ff9a9e 0%, #fad0c4 20%, #fbc2eb 40%, #a6c0fe 60%, #42e695 80%, #f68084 100%);
  background-size: 300% 300%;
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  text-shadow: 0 0 20px rgba(255,255,255,0.25), 0 0 40px rgba(255,255,255,0.15);
  animation: gradientShift 8s ease-in-out infinite, pulse 2.2s ease-in-out infinite;
}

.subtitle {
  margin-top: 1rem;
  font-size: clamp(14px, 2.2vmin, 22px);
  letter-spacing: 0.4em;
  text-transform: uppercase;
  opacity: 0.85;
  color: rgba(255,255,255,0.9);
  filter: drop-shadow(0 2px 8px rgba(0,0,0,0.35));
}

@keyframes hue {
  0% { filter: hue-rotate(0deg) saturate(120%); }
  50% { filter: hue-rotate(180deg) saturate(135%); }
  100% { filter: hue-rotate(360deg) saturate(120%); }
}

@keyframes float {
  0% { transform: translateY(0) scale(1); }
  50% { transform: translateY(-4vmin) scale(1.05); }
  100% { transform: translateY(0) scale(1); }
}

@keyframes spin { to { transform: translate(-50%, -50%) rotate(360deg); } }
@keyframes spinReverse { to { transform: translate(-50%, -50%) rotate(-360deg); } }

@keyframes gradientShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

@keyframes pulse {
  0%, 100% { text-shadow: 0 0 14px rgba(255,255,255,0.25), 0 0 36px rgba(255,255,255,0.14); }
  50% { text-shadow: 0 0 26px rgba(255,255,255,0.45), 0 0 64px rgba(255,255,255,0.28); }
}

@keyframes twinkle {
  0%, 100% { opacity: 0.75; }
  50% { opacity: 1; }
}

@media (prefers-reduced-motion: reduce) {
  .bg-gradient, .stars, .orb, .ring, .title { animation: none !important; }
}
</style>
