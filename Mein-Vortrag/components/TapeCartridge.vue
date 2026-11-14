<script setup lang="ts">
// Data bits travelling along the tape between the two reels
const bits = Array.from({ length: 14 }, (_, i) => ({
  delay: -(i * 0.42),
  text: i % 3 === 1 ? '0' : '1',
}))
</script>

<template>
  <svg viewBox="0 0 520 380" class="cartridge" aria-hidden="true">
    <defs>
      <linearGradient id="tc-shell" x1="0" y1="0" x2="1" y2="1">
        <stop offset="0" stop-color="#2a2f3a" />
        <stop offset="0.55" stop-color="#14171e" />
        <stop offset="1" stop-color="#07080b" />
      </linearGradient>
      <radialGradient id="tc-reel" cx="0.5" cy="0.5" r="0.5">
        <stop offset="0" stop-color="#3a2c20" />
        <stop offset="0.6" stop-color="#5b4431" />
        <stop offset="1" stop-color="#7a5c41" />
      </radialGradient>
      <path id="tc-tape" d="M150 250 C 200 320, 320 320, 370 250" />
    </defs>

    <!-- Cartridge shell -->
    <rect x="20" y="30" width="480" height="320" rx="28" fill="url(#tc-shell)" stroke="#3a404d" stroke-width="1.5" />
    <rect x="21" y="31" width="478" height="160" rx="27" fill="#ffffff" opacity="0.04" />
    <rect x="44" y="54" width="432" height="44" rx="10" fill="#f5f6f8" />
    <text x="64" y="82" class="label">CTA · 2026</text>
    <text x="456" y="82" text-anchor="end" class="label dim">LTO-10 · 40 TB</text>

    <!-- Reels -->
    <g v-for="(cx, i) in [150, 370]" :key="i" :transform="`translate(${cx} 210)`">
      <circle r="88" fill="url(#tc-reel)" />
      <g class="spin" :style="{ animationDuration: i ? '3.2s' : '2.6s' }">
        <circle r="34" fill="#1b1f27" stroke="#2f6bff" stroke-width="2" />
        <rect v-for="a in [0, 60, 120]" :key="a" x="-3" y="-34" width="6" height="68" rx="3"
              fill="#2f6bff" fill-opacity="0.35" :transform="`rotate(${a})`" />
        <circle r="9" fill="#2f6bff" />
      </g>
    </g>

    <!-- Tape and moving bits -->
    <use href="#tc-tape" fill="none" stroke="#6b4f35" stroke-width="7" />
    <text v-for="(b, i) in bits" :key="i" class="bit">
      <textPath href="#tc-tape" startOffset="0%">
        <animate attributeName="startOffset" from="0%" to="100%" dur="5.8s" :begin="`${b.delay}s`" repeatCount="indefinite" />
        {{ b.text }}
      </textPath>
    </text>

    <!-- Read/write head -->
    <rect x="236" y="298" width="48" height="22" rx="5" fill="#2a2f3a" stroke="#3a404d" />
    <circle cx="260" cy="309" r="3.5" fill="#00a389" class="blink" />
  </svg>
</template>

<style scoped>
.cartridge {
  width: 100%;
  height: auto;
  filter: drop-shadow(0 30px 50px rgba(15, 23, 42, 0.35));
}
.label { font: 600 14px 'JetBrains Mono', monospace; letter-spacing: 0.12em; fill: #0b1220; }
.label.dim { fill: #64748b; }
.spin { animation: spin linear infinite; transform-origin: 0 0; }
.bit { font: 700 13px 'JetBrains Mono', monospace; fill: #7fa6ff; dominant-baseline: middle; }
.blink { animation: blink 1.4s ease-in-out infinite; }

@keyframes spin { to { transform: rotate(360deg); } }
@keyframes blink { 50% { opacity: 0.2; } }

@media (prefers-reduced-motion: reduce) {
  .spin, .blink { animation: none; }
}
</style>
