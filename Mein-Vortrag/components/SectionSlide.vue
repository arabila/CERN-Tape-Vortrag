<script setup lang="ts">
// Divider between the four parts of the talk
defineProps<{ num: number; title: string; subtitle: string }>()
</script>

<template>
  <div class="section">
    <div class="dots-bg" />
    <div class="blob" style="width: 520px; height: 520px; right: -120px; top: -160px; background: #c9d8ff" />
    <div class="blob" style="width: 380px; height: 380px; right: 260px; bottom: -220px; background: #c6efe6" />

    <div class="num" v-motion :initial="{ opacity: 0, x: -30 }" :enter="{ opacity: 1, x: 0, transition: { duration: 700 } }">
      {{ String(num).padStart(2, '0') }}
    </div>
    <div class="text">
      <div class="kicker" v-motion :initial="{ opacity: 0, y: 10 }" :enter="{ opacity: 1, y: 0, transition: { delay: 150, duration: 500 } }">
        Teil {{ num }} von 5
      </div>
      <h1 v-motion :initial="{ opacity: 0, y: 18 }" :enter="{ opacity: 1, y: 0, transition: { delay: 250, duration: 600 } }">{{ title }}</h1>
      <p v-motion :initial="{ opacity: 0, y: 12 }" :enter="{ opacity: 1, y: 0, transition: { delay: 400, duration: 600 } }">{{ subtitle }}</p>
      <div class="progress">
        <span v-for="k in 5" :key="k" :class="{ done: k < num, on: k === num }" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.section {
  position: absolute;
  inset: 0;
  display: grid;
  grid-template-columns: auto 1fr;
  align-items: center;
  gap: 56px;
  padding: 0 96px;
  overflow: hidden;
  background: var(--bg);
}
.num {
  position: relative;
  font-size: 260px;
  font-weight: 800;
  line-height: 1;
  letter-spacing: -0.06em;
  color: #e6edff;
  -webkit-text-stroke: 3px var(--accent);
}
.text { position: relative; }
h1 {
  margin: 16px 0 0;
  font-size: 68px;
  font-weight: 800;
  line-height: 1.02;
  letter-spacing: -0.04em;
  color: var(--ink);
}
p { margin-top: 18px; max-width: 34ch; font-size: 22px; line-height: 1.45; color: var(--muted); opacity: 1; }
.progress { display: flex; gap: 8px; margin-top: 40px; }
.progress span { width: 44px; height: 5px; border-radius: 3px; background: var(--line); }
.progress span.done { background: #b9cbff; }
.progress span.on { background: var(--accent); width: 72px; }
</style>
