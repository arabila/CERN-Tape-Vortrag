<script setup lang="ts">
// Why the mount times come in pairs (T1/T2, T3/T4, ...): the Diamondback gripper has two bays.
// One cartridge in front of the target = swap in place (bay 2 takes the target, bay 1 puts the
// front one straight back). Two in front = both bays full, they must be carried to another slot.
// Mean times measured with 1 drive (V. Bahyl, CTA Workshop 2026, slide 10 data).
// ⚠️ The dual-gripper explanation of the pairs is our interpretation, see speaker notes.
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'
import { TIER_COLORS } from './diamondbackData'

const MEAN = [19, 24, 42, 47, 61, 67, 83, 87, 104]

// ---------- geometry (SVG units) ----------
const CW = 34, CH = 38, P = 40
const HEADX = 150, DRIVEX = 66
const ROWY = { drive: 58, park: 138, main: 216 } as const
const BAY = { U: -24, D: 24 } as const
const SX = (i: number) => 208 + i * P
type Bay = keyof typeof BAY
type Pt = { x: number; y: number; o: number }
type Frame = { t: number; head: number; carts: Record<string, Pt>; cap: string }

const MAIN_IDS = Array.from({ length: 9 }, (_, i) => `m${i + 1}`)
const PARK_IDS = ['p1', 'p2', 'p3', 'p4']
const SPEED = 0.6 // < 1 = faster

// ---------- build the keyframes of one case (target tier k) ----------
function build(k: number) {
  const main = [...MAIN_IDS], park = [...PARK_IDS]
  const carts: Record<string, Pt> = {}
  const home = () => {
    MAIN_IDS.forEach((id, i) => { carts[id] = { x: SX(i), y: ROWY.main, o: 1 } })
    PARK_IDS.forEach((id, i) => { carts[id] = { x: SX(i), y: ROWY.park, o: 1 } })
  }
  home()
  const held: Partial<Record<Bay, string>> = {}
  let head = ROWY.main - BAY.U
  let t = 0
  const frames: Frame[] = []
  const snap = (dur: number, cap: string) => {
    t += dur * SPEED
    frames.push({ t, head, carts: JSON.parse(JSON.stringify(carts)), cap })
  }
  const restack = (row: string[], y: number) => row.forEach((id, i) => { carts[id] = { x: SX(i), y, o: 1 } })
  const follow = () => { for (const b of ['U', 'D'] as Bay[]) { const id = held[b]; if (id) carts[id] = { x: HEADX, y: head + BAY[b], o: 1 } } }
  const align = (y: number, b: Bay, cap: string) => {
    const target = y - BAY[b]
    if (target === head) return
    const dur = 0.3 + Math.abs(target - head) / 320
    head = target; follow(); snap(dur, cap)
  }
  const pull = (rowName: 'main' | 'park', b: Bay, cap: string) => {
    const row = rowName === 'main' ? main : park
    const id = row.shift()!
    held[b] = id; follow(); restack(row, ROWY[rowName]); snap(0.55, cap)
  }
  const push = (rowName: 'main' | 'park', b: Bay, cap: string) => {
    const row = rowName === 'main' ? main : park
    const id = held[b]!
    delete held[b]; row.unshift(id); restack(row, ROWY[rowName]); snap(0.55, cap)
  }
  const name = (id: string) => `T${id.slice(1)}`

  snap(0.6, `Gesucht: Kassette aus T${k}`)
  const pairs = Math.floor((k - 1) / 2)
  for (let p = 0; p < pairs; p++) {
    const a = main[0], b = main[1]
    pull('main', 'U', `Hand 1 nimmt ${name(a)}`)
    align(ROWY.main, 'D', `Hand 1 nimmt ${name(a)}`)
    pull('main', 'D', `Hand 2 nimmt ${name(b)}: beide Hände voll`)
    align(ROWY.park, 'D', `${name(a)} und ${name(b)} in einen anderen Schacht bringen`)
    push('park', 'D', `${name(a)} und ${name(b)} in einen anderen Schacht bringen`)
    align(ROWY.park, 'U', `${name(a)} und ${name(b)} in einen anderen Schacht bringen`)
    push('park', 'U', `${name(a)} und ${name(b)} in einen anderen Schacht bringen`)
    align(ROWY.main, 'U', 'Zurück zum Schacht')
  }
  let tb: Bay = 'U'
  if (k % 2 === 0) {
    const a = main[0], b = main[1]
    pull('main', 'U', `Hand 1 nimmt ${name(a)}`)
    align(ROWY.main, 'D', `Hand 2 nimmt ${name(b)}`)
    pull('main', 'D', `Hand 2 nimmt ${name(b)}`)
    align(ROWY.main, 'U', `Tausch vor Ort: ${name(a)} sofort zurück`)
    push('main', 'U', `Tausch vor Ort: ${name(a)} sofort zurück`)
    tb = 'D'
  }
  else {
    pull('main', 'U', `Hand 1 nimmt ${name(main[0])}`)
  }
  const target = held[tb]!
  align(ROWY.drive, tb, `${name(target)} zum Laufwerk`)
  delete held[tb]; carts[target] = { x: DRIVEX, y: ROWY.drive, o: 1 }; snap(0.5, `${name(target)} ist im Laufwerk`)
  const work = t
  snap(0.9, `${name(target)} ist im Laufwerk`)
  // soft reset for the next case: the cartridge in the drive fades out, the gripper slides
  // back down and the other cartridges glide back to where they started
  const cap = k < 4 ? `Nächster Fall: T${k + 1}` : 'Von vorn: T1'
  home(); carts[target] = { x: DRIVEX, y: ROWY.drive, o: 0 }; head = ROWY.main - BAY.U; snap(0.9, cap)
  carts[target] = { x: SX(k - 1), y: ROWY.main, o: 0 }; snap(0.02, cap)
  carts[target].o = 1; snap(0.45, cap)
  return { frames, work, total: t, k }
}

const CASES = [1, 2, 3, 4].map(build)
const OFFSETS = CASES.reduce<number[]>((acc, c, i) => [...acc, (acc[i] ?? 0) + c.total], [0])
const CYCLE = OFFSETS.at(-1)!

// ---------- clock ----------
const now = ref(0)
let raf = 0
onMounted(() => {
  const start = performance.now()
  const step = (ms: number) => { now.value = ((ms - start) / 1000) % CYCLE; raf = requestAnimationFrame(step) }
  raf = requestAnimationFrame(step)
})
onBeforeUnmount(() => cancelAnimationFrame(raf))

const ease = (x: number) => x * x * (3 - 2 * x)
const caseIdx = computed(() => Math.max(0, OFFSETS.findIndex((o, i) => now.value >= o && now.value < (OFFSETS[i + 1] ?? Infinity))))
const cur = computed(() => CASES[Math.min(caseIdx.value, CASES.length - 1)])
const local = computed(() => now.value - OFFSETS[caseIdx.value])
const view = computed(() => {
  const f = cur.value.frames
  const t = local.value
  let i = f.findIndex(fr => fr.t > t)
  if (i === -1) i = f.length - 1
  const b = f[i], a = f[i - 1] ?? f[0]
  const span = b.t - (f[i - 1]?.t ?? 0)
  const r = i === 0 ? 1 : ease(Math.min(1, Math.max(0, (t - a.t) / (span || 1))))
  const carts: Record<string, Pt> = {}
  const mix = (u: number, v: number) => u + (v - u) * r
  for (const id in b.carts) { const p = a.carts[id], q = b.carts[id]; carts[id] = { x: mix(p.x, q.x), y: mix(p.y, q.y), o: mix(p.o, q.o) } }
  return { head: a.head + (b.head - a.head) * r, carts, cap: b.cap }
})
const seconds = computed(() => Math.round(MEAN[cur.value.k - 1] * Math.min(1, local.value / cur.value.work)))
const color = (id: string) => id.startsWith('m') ? TIER_COLORS[Number(id.slice(1)) - 1] : '#c3cbd7'

// ---------- chart ----------
const BW = 26, BG = 8, CHART_H = 170, BASE = 196
const barX = (i: number) => 18 + i * (BW + BG) + Math.floor(i / 2) * 8
const barH = (v: number) => (v / 110) * CHART_H
const groups = [[0, 1], [2, 3], [4, 5], [6, 7], [8]]
</script>

<template>
  <div class="gp">
    <div class="panel anim">
      <div class="chips">
        <span v-for="(c, i) in CASES" :key="c.k" class="chip" :class="{ on: i === caseIdx }" :style="{ '--c': TIER_COLORS[c.k - 1] }">T{{ c.k }}</span>
        <span class="clock"><b>{{ seconds }}</b> s <em>gemessenes Mittel</em></span>
      </div>
      <svg viewBox="0 -8 600 304">
        <!-- slots -->
        <g class="slot">
          <rect :x="SX(0) - CW / 2 - 6" :y="ROWY.park - CH / 2 - 5" :width="9 * P + 8" :height="CH + 10" rx="6" />
          <rect :x="SX(0) - CW / 2 - 6" :y="ROWY.main - CH / 2 - 5" :width="9 * P + 8" :height="CH + 10" rx="6" />
          <text :x="SX(0) - CW / 2 - 4" :y="ROWY.park - CH / 2 - 10">anderer Schacht</text>
          <text :x="SX(0) - CW / 2 - 4" :y="ROWY.main - CH / 2 - 10">Schacht mit der gesuchten Kassette</text>
          <text v-for="i in 9" :key="i" :x="SX(i - 1)" :y="ROWY.main + CH / 2 + 18" text-anchor="middle" class="tl">T{{ i }}</text>
        </g>
        <!-- drive -->
        <g class="drive">
          <rect :x="DRIVEX - 32" :y="ROWY.drive - 28" width="64" height="56" rx="8" />
          <text :x="DRIVEX" :y="ROWY.drive + 44" text-anchor="middle">Laufwerk</text>
        </g>
        <!-- rail + gripper -->
        <line :x1="HEADX" :x2="HEADX" y1="0" y2="292" class="rail" />
        <g class="grip" :transform="`translate(${HEADX} ${view.head})`">
          <rect x="-26" y="-50" width="52" height="100" rx="10" />
          <line x1="-26" x2="26" y1="0" y2="0" />
          <text x="-34" y="-20" text-anchor="end">Hand 1</text>
          <text x="-34" y="28" text-anchor="end">Hand 2</text>
        </g>
        <!-- cartridges -->
        <g v-for="(p, id) in view.carts" :key="id" :transform="`translate(${p.x} ${p.y})`" :opacity="p.o" class="cart" :class="{ target: id === `m${cur.k}` }">
          <rect :x="-CW / 2" :y="-CH / 2" :width="CW" :height="CH" rx="5" :style="{ fill: color(String(id)) }" />
          <text v-if="String(id).startsWith('m')" y="5" text-anchor="middle">{{ String(id).slice(1) }}</text>
        </g>
      </svg>
      <div class="cap">{{ view.cap }}</div>
    </div>

    <div class="panel chart">
      <b class="ct">Mittlere Mount-Zeit pro Ebene (1 Laufwerk)</b>
      <svg viewBox="0 0 352 222">
        <g v-for="(g, gi) in groups" :key="gi" class="band">
          <rect :x="barX(g[0]) - 4" y="14" :width="g.length * (BW + BG) - BG + 8" :height="BASE - 10" rx="6" />
        </g>
        <g v-for="(m, i) in MEAN" :key="i" class="bar" :class="{ on: i + 1 === cur.k }">
          <rect :x="barX(i)" :y="BASE - barH(m)" :width="BW" :height="barH(m)" rx="4" :style="{ fill: TIER_COLORS[i] }" />
          <text :x="barX(i) + BW / 2" :y="BASE - barH(m) - 5" text-anchor="middle" class="v">{{ m }}</text>
          <text :x="barX(i) + BW / 2" :y="BASE + 16" text-anchor="middle" class="x">T{{ i + 1 }}</text>
        </g>
      </svg>
      <div class="keys">
        <div><i class="small" /><span><b>+5 s innerhalb eines Paars:</b> eine Kassette davor, Tausch vor Ort mit zwei Händen</span></div>
        <div><i class="big" /><span><b>+18 s zum nächsten Paar:</b> zwei Kassetten davor, beide Hände voll, also einmal wegbringen</span></div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.gp { height: 100%; display: grid; grid-template-columns: 1.45fr 1fr; gap: 16px; }
.panel { display: flex; flex-direction: column; min-height: 0; padding: 12px 16px; border-radius: var(--radius); border: 1px solid var(--line); background: var(--surface); box-shadow: var(--shadow); }
.panel svg { flex: 1; min-height: 0; width: 100%; }

.chips { display: flex; align-items: center; gap: 8px; }
.chip { padding: 3px 11px; border-radius: 999px; font: 700 12.5px 'JetBrains Mono', monospace; color: var(--muted); background: var(--surface-2); transition: background 0.25s, color 0.25s; }
.chip.on { color: #fff; background: var(--c); }
.clock { margin-left: auto; font: 600 13px 'JetBrains Mono', monospace; color: var(--muted); }
.clock b { font-size: 22px; color: var(--ink); }
.clock em { margin-left: 6px; font-style: normal; font-size: 11px; }

.slot rect { fill: var(--surface-2); stroke: #c9d1dc; stroke-width: 1.5; }
.slot text { font: 600 11px 'JetBrains Mono', monospace; fill: var(--muted); }
.slot .tl { font-size: 10.5px; }
.drive rect { fill: #1f2937; }
.drive text { font: 600 11px 'JetBrains Mono', monospace; fill: var(--muted); }
.rail { stroke: #cbd5e1; stroke-width: 3; stroke-dasharray: 2 5; }
.grip rect { fill: rgba(115, 87, 255, 0.08); stroke: var(--violet); stroke-width: 2.5; }
.grip line { stroke: var(--violet); stroke-width: 1.5; stroke-dasharray: 3 3; }
.grip text { font: 700 11px 'JetBrains Mono', monospace; fill: var(--violet); }
.cart rect { stroke: rgba(15, 23, 42, 0.25); stroke-width: 1; }
.cart text { font: 700 13px 'JetBrains Mono', monospace; fill: #fff; }
.cart.target rect { stroke: var(--ink); stroke-width: 2.5; }
.cap { padding: 8px 12px; border-radius: 10px; font-size: 15px; font-weight: 600; color: var(--ink); background: var(--violet-soft); }

.ct { font-size: 14.5px; color: var(--ink); }
.band rect { fill: var(--surface-2); }
.bar { opacity: 0.45; transition: opacity 0.3s; }
.bar.on { opacity: 1; }
.bar .v { font: 700 11.5px 'JetBrains Mono', monospace; fill: var(--ink-2); }
.bar .x { font: 600 11px 'JetBrains Mono', monospace; fill: var(--muted); }
.keys { display: grid; gap: 8px; font-size: 13px; line-height: 1.38; color: var(--ink-2); }
.keys div { display: flex; gap: 10px; align-items: flex-start; }
.keys b { color: var(--ink); }
.keys i { flex: none; margin-top: 6px; height: 4px; border-radius: 2px; background: var(--violet); }
.keys i.small { width: 10px; }
.keys i.big { width: 28px; background: var(--rose); }
</style>
