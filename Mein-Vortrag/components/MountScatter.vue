<script setup lang="ts">
// One scatter panel of the Diamondback mount tests: x = time per cartridge, y = library row,
// colour = tier the cartridge came from. Data: diamondbackData.ts (measured at CERN).
import { computed } from 'vue'
import { DB, TIER_COLORS } from './diamondbackData'

const props = withDefaults(defineProps<{
  set: string
  xMax: number
  xStep?: number
  title?: string
  sub?: string
  accent?: string
  big?: boolean
  markMax?: boolean
}>(), { xStep: 20, accent: 'var(--ink)' })

const W = props.big ? 600 : 380, H = props.big ? 400 : 340
const L = 38, R = W - 12, T = 10, B = H - 36
const ROWS = 44
const x = (s: number) => L + (Math.min(s, props.xMax) / props.xMax) * (R - L)
const y = (row: number) => T + ((row - 1) / (ROWS - 1)) * (B - T)

const tiers = computed(() => DB[props.set] ?? [])
const all = computed(() => tiers.value.flat().map(p => p[0]).sort((a, b) => a - b))
const median = computed(() => all.value[Math.floor(all.value.length / 2)] ?? 0)
const max = computed(() => all.value.at(-1) ?? 0)
// Arrow + label pointing at the slowest mount (the outlier mentioned in the slide text)
const outlier = computed(() => {
  let best: [number, number] | null = null
  for (const pts of tiers.value) for (const p of pts) if (!best || p[0] > best[0]) best = p
  if (!best) return null
  const px = x(best[0]), py = y(best[1])
  const below = best[1] < ROWS / 2
  const lw = 128, lh = 22
  const lx = Math.max(L + 4, Math.min(px - lw - 26, R - lw))
  const ly = Math.max(T + 2, Math.min(below ? py + 30 : py - 30 - lh, B - lh - 2))
  // arrow from label edge to just outside the ring around the point
  const sx = lx + lw, sy = ly + lh / 2
  const d = Math.hypot(px - sx, py - sy) || 1
  const ex = px - ((px - sx) / d) * 11, ey = py - ((py - sy) / d) * 11
  return { px, py, lx, ly, lw, lh, sx, sy, ex, ey }
})
const ticks = computed(() => Array.from({ length: Math.floor(props.xMax / props.xStep) + 1 }, (_, i) => i * props.xStep))
</script>

<template>
  <div class="panel" :class="{ big }" :style="{ '--a': accent }">
    <header v-if="title">
      <b>{{ title }}</b>
      <span v-if="sub">{{ sub }}</span>
    </header>
    <svg :viewBox="`0 0 ${W} ${H}`">
      <g class="axes">
        <line v-for="t in ticks" :key="t" :x1="x(t)" :x2="x(t)" :y1="T" :y2="B" />
        <text v-for="t in ticks" :key="`l${t}`" :x="x(t)" :y="B + 18" text-anchor="middle">{{ t }}</text>
        <text :x="R" :y="B + 32" text-anchor="end" class="unit">Sekunden</text>
        <text x="2" :y="T + 8" class="unit">oben</text>
        <text x="2" :y="B" class="unit">unten</text>
      </g>
      <line :x1="x(median)" :x2="x(median)" :y1="T" :y2="B" class="median" />
      <g v-for="(pts, ti) in tiers" :key="`${set}-${ti}`" class="tier" :style="{ animationDelay: `${ti * 70}ms` }">
        <circle v-for="(p, i) in pts" :key="i" :cx="x(p[0])" :cy="y(p[1])" :r="big ? 4.2 : 3.4" :fill="TIER_COLORS[ti]" />
      </g>
      <g v-if="markMax && outlier" class="outlier">
        <circle :cx="outlier.px" :cy="outlier.py" r="8" class="ring" />
        <line :x1="outlier.sx" :y1="outlier.sy" :x2="outlier.ex" :y2="outlier.ey" :marker-end="`url(#oa-${set})`" />
        <rect :x="outlier.lx" :y="outlier.ly" :width="outlier.lw" :height="outlier.lh" rx="11" />
        <text :x="outlier.lx + outlier.lw / 2" :y="outlier.ly + 15" text-anchor="middle">Ausreißer {{ max }} s</text>
      </g>
      <defs>
        <marker :id="`oa-${set}`" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
          <path d="M0 0 L10 5 L0 10 z" style="fill: var(--rose)" />
        </marker>
      </defs>
      <g class="median-tag" :transform="`translate(${Math.min(x(median) + 6, R - 108)}, ${T + 4})`">
        <rect width="106" height="22" rx="11" />
        <text x="53" y="15" text-anchor="middle">Median {{ median }} s</text>
      </g>
    </svg>
    <footer>
      <span>Median <b>{{ median }} s</b></span>
      <span>Langsamste <b>{{ max }} s</b></span>
      <span>{{ all.length }} Mounts</span>
    </footer>
  </div>
</template>

<style scoped>
.panel { display: flex; flex-direction: column; min-height: 0; min-width: 0; padding: 12px 14px 10px; border-radius: var(--radius); border: 1px solid var(--line); background: var(--surface); box-shadow: var(--shadow); }
header { display: flex; align-items: baseline; justify-content: space-between; gap: 10px; padding-bottom: 4px; }
header b { font-size: 16px; color: var(--a); }
header span { font: 600 11.5px 'JetBrains Mono', monospace; color: var(--muted); }
svg { flex: 1; min-height: 0; width: 100%; }
.axes line { stroke: var(--line); }
.axes text { font: 500 13px 'JetBrains Mono', monospace; fill: var(--muted); }
.axes .unit { font-size: 11px; fill: #94a3b8; }
.median { stroke: var(--ink-2); stroke-width: 1.5; stroke-dasharray: 5 4; }
.median-tag rect { fill: var(--ink); }
.median-tag text { font: 700 11.5px 'JetBrains Mono', monospace; fill: #fff; }
.outlier { opacity: 0; animation: fadein 0.5s ease 0.9s forwards; }
.outlier .ring { fill: none; stroke: var(--rose); stroke-width: 2.5; }
.outlier line { stroke: var(--rose); stroke-width: 2.5; }
.outlier rect { fill: var(--rose); }
.outlier text { font: 700 11.5px 'JetBrains Mono', monospace; fill: #fff; }
@keyframes fadein { to { opacity: 1; } }
.tier { opacity: 0; animation: pop 0.45s ease forwards; }
.tier circle { fill-opacity: 0.85; }
@keyframes pop { from { opacity: 0; transform: translateX(-10px); } to { opacity: 1; transform: none; } }
footer { display: flex; gap: 16px; padding-top: 6px; border-top: 1px solid var(--line); font-size: 12.5px; color: var(--muted); }
footer b { color: var(--ink); }
.big header b { font-size: 18px; }
</style>
