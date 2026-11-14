<script setup lang="ts">
// v6 shared runtime library: every service gets the same operational features
const services = ['cta-workflow-api', 'cta-admin-api', 'cta-taped', 'cta-maintd', 'cta-rmcd']
const features = [
  { name: 'TOML-Konfiguration', hint: 'ein Format für alles' },
  { name: 'Config-Check', hint: '--config-strict' },
  { name: 'Health-Probes', hint: '/health/ready · /health/live' },
  { name: 'JSON-Logs', hint: 'mit stabilem event_name' },
  { name: 'Telemetrie', hint: 'OpenTelemetry-Metriken' },
  { name: 'Runtime-Snapshot', hint: '/run/cta/<dienst>' },
]
const H = 460
const sy = (i: number, n: number) => 40 + i * ((H - 80) / (n - 1))
</script>

<template>
  <SlideFrame :section="3" title="Ein gemeinsames Fundament für jeden Dienst" kicker="Neu in CTA v6 · Runtime-Bibliothek">
    <svg viewBox="0 0 1150 460" class="hub">
      <!-- Links -->
      <path v-for="(s, i) in services" :key="`l${s}`" :d="`M250 ${sy(i, services.length)} C 380 ${sy(i, services.length)}, 420 230, 500 230`" class="link" :style="{ animationDelay: `${i * 0.25}s` }" />
      <path v-for="(f, i) in features" :key="`r${f.name}`" :d="`M650 230 C 730 230, 770 ${sy(i, features.length)}, 860 ${sy(i, features.length)}`" class="link right" :style="{ animationDelay: `${i * 0.2}s` }" />

      <!-- Services -->
      <g v-for="(s, i) in services" :key="s" :transform="`translate(20 ${sy(i, services.length) - 24})`">
        <g class="svc" v-motion :initial="{ opacity: 0, x: -20 }" :enter="{ opacity: 1, x: 0, transition: { delay: i * 100, duration: 400 } }">
          <rect width="230" height="48" rx="12" />
          <circle cx="24" cy="24" r="5" />
          <text x="40" y="29">{{ s }}</text>
        </g>
      </g>

      <!-- Hub -->
      <g class="core" v-motion :initial="{ opacity: 0, scale: 0.8 }" :enter="{ opacity: 1, scale: 1, transition: { delay: 300, duration: 600 } }">
        <circle cx="575" cy="230" r="112" class="ring" />
        <circle cx="575" cy="230" r="86" class="disc" />
        <text x="575" y="222" text-anchor="middle" class="core-t1">CTA</text>
        <text x="575" y="246" text-anchor="middle" class="core-t2">Runtime-Bibliothek</text>
      </g>

      <!-- Features -->
      <g v-for="(f, i) in features" :key="f.name" :transform="`translate(860 ${sy(i, features.length) - 26})`">
        <g class="feat" v-motion :initial="{ opacity: 0, x: 20 }" :enter="{ opacity: 1, x: 0, transition: { delay: 600 + i * 100, duration: 400 } }">
          <rect width="270" height="52" rx="12" />
          <text x="18" y="23" class="f1">{{ f.name }}</text>
          <text x="18" y="41" class="f2">{{ f.hint }}</text>
        </g>
      </g>
    </svg>
  </SlideFrame>
</template>

<style scoped>
.hub { width: 100%; height: 100%; }
.link {
  fill: none;
  stroke: var(--violet);
  stroke-width: 2;
  stroke-opacity: 0.45;
  stroke-dasharray: 4 10;
  animation: flow 1.2s linear infinite;
}
.link.right { stroke: var(--accent); }
.core { transform-box: fill-box; transform-origin: center; }
.svc rect { fill: var(--surface); stroke: var(--line); }
.svc circle { fill: var(--violet); }
.svc text { font: 600 15px 'JetBrains Mono', monospace; fill: var(--ink); }
.ring { fill: none; stroke: var(--violet); stroke-opacity: 0.25; stroke-width: 2; stroke-dasharray: 6 8; transform-box: fill-box; transform-origin: center; animation: spin 24s linear infinite; }
.disc { fill: var(--violet); filter: drop-shadow(0 16px 30px rgba(115, 87, 255, 0.35)); }
.core-t1 { font: 800 30px 'Inter', sans-serif; fill: #fff; letter-spacing: -0.02em; }
.core-t2 { font: 600 13px 'Inter', sans-serif; fill: #e4ddff; }
.feat rect { fill: var(--surface); stroke: var(--line); }
.f1 { font: 650 15px 'Inter', sans-serif; fill: var(--ink); }
.f2 { font: 500 12px 'JetBrains Mono', monospace; fill: var(--muted); }

@keyframes flow { to { stroke-dashoffset: -28; } }
@keyframes spin { to { transform: rotate(360deg); } }
</style>
