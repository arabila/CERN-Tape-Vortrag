<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'

// Data on tape at CERN (PB), from the workshop welcome talk
const years = [
  { y: '2015', v: 100, note: 'Start Run-2' },
  { y: '2018', v: 340, note: 'Ende Run-2' },
  { y: '2022', v: 420, note: 'Start Run-3' },
  { y: '2023', v: 688 },
  { y: '2024', v: 946 },
  { y: '2025', v: 1250 },
  { y: '2026', v: 1480, note: 'Ende Run-3' },
]
const MAX = 1600

const stats = [
  { value: 1480, unit: 'PB', label: 'Daten auf Band am CERN', sub: '3,5× so viel wie zu Beginn von Run-3' },
  { value: 1, unit: 'EB+', label: 'in Run-3 neu archiviert', sub: 'Design-Annahme: 150 PB pro Jahr' },
  { value: 62, unit: 'PB', label: 'Rekordmonat Oktober 2025', sub: 'zum Vergleich: 4,6 PB im Nov. 2012' },
]

// Count-up animation for the big numbers and growing bars
const progress = ref(0)
onMounted(() => {
  const start = performance.now()
  const step = (now: number) => {
    // The first frame's timestamp can lie slightly before `start`, so clamp at 0 as well
    progress.value = Math.min(1, Math.max(0, (now - start) / 1600))
    if (progress.value < 1)
      requestAnimationFrame(step)
  }
  requestAnimationFrame(step)
})
const eased = computed(() => 1 - (1 - progress.value) ** 3)
const fmt = (n: number) => Math.round(n).toLocaleString('de-DE')
</script>

<template>
  <SlideFrame :section="1" title="Ein Exabyte in einem Run" kicker="CTA heute · Software und Dienst am CERN">
    <div class="layout">
      <div class="left">
        <p class="intro">
          CTA ist die Tape-Software des CERN und zugleich der Dienst, der damit betrieben wird.
          Davor sitzt ein Disk-System: <b>EOS</b> oder <b>dCache</b>.
        </p>
        <div class="stats">
          <div v-for="(s, i) in stats" :key="s.label" class="card stat" v-motion :initial="{ opacity: 0, x: -20 }" :enter="{ opacity: 1, x: 0, transition: { delay: 100 + i * 150, duration: 500 } }">
            <div class="value"><span>{{ fmt(s.value * eased) }}</span><small>{{ s.unit }}</small></div>
            <div>
              <div class="label">{{ s.label }}</div>
              <div class="sub">{{ s.sub }}</div>
            </div>
          </div>
        </div>
      </div>

      <div class="card chart">
        <div class="chart-head">
          <span class="chart-title">Daten auf Band am CERN</span>
          <span class="mono legend">Petabyte</span>
        </div>
        <svg viewBox="0 0 560 330" class="bars">
          <line v-for="g in [400, 800, 1200, 1600]" :key="g" x1="40" :x2="550" :y1="300 - g / MAX * 270" :y2="300 - g / MAX * 270" class="grid" />
          <text v-for="g in [400, 800, 1200, 1600]" :key="`t${g}`" x="32" :y="304 - g / MAX * 270" text-anchor="end" class="axis">{{ g }}</text>
          <rect x="184" y="30" width="358" height="270" rx="10" class="run3" />
          <text x="363" y="50" text-anchor="middle" class="run3-label">RUN-3</text>
          <g v-for="(d, i) in years" :key="d.y" :transform="`translate(${52 + i * 72} 0)`">
            <rect x="8" :y="300 - d.v / MAX * 270 * eased" width="44" :height="d.v / MAX * 270 * eased" rx="8"
                  class="bar" :class="{ last: i === years.length - 1 }" />
            <text x="30" :y="292 - d.v / MAX * 270 * eased" text-anchor="middle" class="val" :opacity="eased">{{ d.v }}</text>
            <text x="30" y="320" text-anchor="middle" class="axis">{{ d.y }}</text>
          </g>
        </svg>
      </div>
    </div>
  </SlideFrame>
</template>

<style scoped>
.layout { height: 100%; display: grid; grid-template-columns: 1fr 1.15fr; gap: 28px; align-items: stretch; }
.left { display: flex; flex-direction: column; gap: 16px; }
.intro { margin: 0; font-size: 18px; line-height: 1.5; color: var(--ink-2); opacity: 1; }
.intro b { color: var(--accent); }
.stats { flex: 1; display: grid; gap: 12px; }
.stat { display: flex; align-items: center; gap: 20px; padding: 14px 20px; }
.value { min-width: 150px; font-size: 46px; font-weight: 800; letter-spacing: -0.04em; color: var(--ink); font-variant-numeric: tabular-nums; }
.value small { margin-left: 4px; font-size: 18px; font-weight: 700; color: var(--accent); }
.label { font-size: 16px; font-weight: 650; color: var(--ink); }
.sub { margin-top: 3px; font-size: 13px; color: var(--muted); }

.chart { display: flex; flex-direction: column; padding: 18px 20px 10px; }
.chart-head { display: flex; justify-content: space-between; align-items: baseline; }
.chart-title { font-size: 15px; font-weight: 650; color: var(--ink); }
.legend { font-size: 12px; color: var(--muted); }
.bars { flex: 1; width: 100%; min-height: 0; }
.grid { stroke: var(--line); stroke-dasharray: 3 4; }
.axis { font: 500 12px 'JetBrains Mono', monospace; fill: var(--muted); }
.run3 { fill: var(--accent-soft); opacity: 0.6; }
.run3-label { font: 700 11px 'JetBrains Mono', monospace; letter-spacing: 0.12em; fill: var(--accent); }
.bar { fill: #b9cbff; }
.bar.last { fill: var(--accent); }
.val { font: 700 12px 'Inter', sans-serif; fill: var(--ink-2); }
</style>
