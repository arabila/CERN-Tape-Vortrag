<script setup lang="ts">
// How the IBM Diamondback stores cartridges: up to 9 per slot, one behind the other,
// pushed to the front by a spring. Only the front one (T1) can be taken out.
// Based on "IBM Slot Tiering explained" and the mount tests (V. Bahyl, CTA Workshop 2026).
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue'
import { useIsSlideActive } from '@slidev/client'

const props = defineProps<{ step: number }>()

type Row = 'A' | 'B'
type Spot = Row | 'D'

// ---- geometry (SVG units) ----
const CW = 56, CH = 46 // cartridge
const X0 = 232, P = 64 // first tier, pitch
const WALL = 864
const ROW = { A: 150, B: 300 } as const
const GRIP_Y = { A: ROW.A - 8, B: ROW.B - 8, D: 104 } as const
const DRIVE = { x: 82, y: 34 }
const BAY = { single: 82, left: 44, right: 112 } // where a cartridge sits inside the gripper
const tiers = Array.from({ length: 9 }, (_, i) => i)
// spring drawn as a zigzag on a 0..1 axis, stretched to the free length of the slot
const COIL = ['0,0', ...Array.from({ length: 16 }, (_, i) => `${((i + 0.5) / 16).toFixed(4)},${i % 2 ? 13 : -13}`), '1,0'].join(' ')
const ids = ['a1', 'a2', 'a3', 'a4', 'a5', 'a6', 'a7', 'a8', 'a9', 'b1', 'b2', 'b3', 'b4', 'b5']
const A_ALL = ids.slice(0, 9), B_ALL = ids.slice(9)

// ---- state ----
// Where every cartridge logically is (model) and where it is drawn (pos, gy). Positions are
// written straight to the SVG nodes once per frame from a single clock, so cartridges,
// gripper and springs can never drift apart and nothing depends on CSS transitions or timers.
const model = { A: [] as string[], B: [] as string[], held: [] as string[], drive: null as string | null }
const pos: Record<string, { x: number; y: number }> = {}
let gy: number = GRIP_Y.A

const status = ref('')
const busy = ref(false)      // cartridge loaded in the drive
const gripping = ref(false)  // gripper holds at least one cartridge
const clockOn = ref(false)

const cartEls: Record<string, SVGGElement | null> = {}
const springEls: Record<string, SVGGElement | null> = {}
const gripEl = ref<SVGGElement>()
const clockEl = ref<HTMLElement>()

function place() {
  for (const id of ids) cartEls[id]?.setAttribute('transform', `translate(${pos[id].x} ${pos[id].y})`)
  gripEl.value?.setAttribute('transform', `translate(36 ${gy})`)
  for (const row of ['A', 'B'] as const) {
    const lastId = model[row].at(-1)
    const from = lastId ? pos[lastId].x + CW : X0 - 8
    springEls[row]?.setAttribute('transform', `translate(${from} ${ROW[row] + CH / 2}) scale(${Math.max(WALL - 4 - from, 1)} 1)`)
  }
}

// ---- the end state of every click (also used when jumping backwards or re-entering the slide) ----
const END: { A: string[]; B: string[]; drive: string | null; grip: Spot; status: string; clock: number | null }[] = [
  { A: A_ALL, B: B_ALL, drive: null, grip: 'A', status: 'Nur T1 ist greifbar', clock: null },
  { A: A_ALL.slice(1), B: B_ALL, drive: 'a1', grip: 'D', status: 'A1 ist im Laufwerk', clock: 19 },
  { A: A_ALL, B: B_ALL, drive: null, grip: 'A', status: 'A1 liegt wieder auf T1', clock: null },
  { A: A_ALL.slice(3), B: ['a1', 'a2', ...B_ALL], drive: 'a3', grip: 'D', status: 'A3 ist im Laufwerk', clock: 42 },
]
const clampStep = (k: number) => Math.min(Math.max(k, 0), END.length - 1)

function setClock(v: number | null) {
  clockOn.value = v !== null
  if (clockEl.value && v !== null) clockEl.value.textContent = String(Math.round(v))
}
function layout(k: number) {
  gen++ // stops everything that is still running
  const e = END[clampStep(k)]
  model.A = [...e.A]; model.B = [...e.B]; model.held = []; model.drive = e.drive
  for (const row of ['A', 'B'] as const) model[row].forEach((id, i) => { pos[id] = { x: X0 + i * P, y: ROW[row] } })
  if (e.drive) pos[e.drive] = { ...DRIVE }
  gy = GRIP_Y[e.grip]
  status.value = e.status; busy.value = !!e.drive; gripping.value = false
  setClock(e.clock)
  place()
}

// ---- clock ----
let gen = 0
let seqMs = 0                                // animated time since the click
let clock: { to: number; est: number } | null = null
const inOut = (p: number) => (p < 0.5 ? 4 * p * p * p : 1 - (-2 * p + 2) ** 3 / 2)
const out = (p: number) => 1 - (1 - p) ** 3
const spring = (p: number) => 1 + 2.2 * (p - 1) ** 3 + 1.2 * (p - 1) ** 2 // slight overshoot, then settles

function animate(ms: number, frame?: (p: number) => void) {
  const my = gen
  return new Promise<void>((resolve) => {
    let elapsed = 0
    let last: number | undefined
    const tick = (now: number) => {
      if (my !== gen) return // superseded: never resolves, so the rest of the old script is dropped
      if (last !== undefined) {
        const dt = Math.min(now - last, 120) // no jumps after a hidden tab, no slow motion on a slow machine
        elapsed += dt; seqMs += dt
      }
      last = now
      const p = Math.min(elapsed / ms, 1)
      frame?.(p)
      if (clock) setClock(clock.to * Math.min(seqMs / clock.est, 0.97))
      if (p < 1) requestAnimationFrame(tick)
      else resolve()
    }
    requestAnimationFrame(tick)
  })
}
const wait = (ms: number) => animate(ms)

// ---- moves ----
function slide(id: string, x: number, y: number, ms: number, ease = inOut) {
  const { x: x0, y: y0 } = pos[id]
  return animate(ms, (p) => { const e = ease(p); pos[id] = { x: x0 + (x - x0) * e, y: y0 + (y - y0) * e }; place() })
}
// all cartridges of a row move to the place their index gives them
function settle(row: Row, ms: number, ease: (p: number) => number) {
  const from = model[row].map(id => pos[id].x)
  return animate(ms, (p) => { const e = ease(p); model[row].forEach((id, i) => { pos[id].x = from[i] + (X0 + i * P - from[i]) * e }); place() })
}
// take the front cartridge out; the spring pushes the rest forward
async function pull(row: Row, bayX: number) {
  const id = model[row].shift()!
  model.held.push(id); gripping.value = true
  await slide(id, bayX, pos[id].y, 440)
  await settle(row, 460, spring)
}
// push the last gripped cartridge back in; it shoves the others against the spring
async function push(row: Row) {
  const id = model.held.pop()!
  await slide(id, X0 - P, pos[id].y, 300)
  model[row].unshift(id)
  await settle(row, 380, out)
  gripping.value = model.held.length > 0
}
function gripTo(spot: Spot) {
  const from = gy, to = GRIP_Y[spot]
  return animate(300 + Math.abs(to - from) * 1.6, (p) => {
    gy = from + (to - from) * inOut(p)
    for (const id of model.held) pos[id].y = gy + 8
    place()
  })
}
async function load() {
  const id = model.held.pop()!
  await slide(id, DRIVE.x, DRIVE.y, 420)
  model.drive = id; gripping.value = false; busy.value = true
}
async function unload() {
  const id = model.drive!
  model.drive = null; model.held.push(id); busy.value = false; gripping.value = true
  await slide(id, BAY.single, gy + 8, 420)
}

// ---- one script per click ----
const scripts: (() => Promise<void>)[] = [
  async () => {},
  async () => { // mount from T1
    clock = { to: 19, est: 2000 }
    status.value = 'Greifer zieht A1 aus T1'
    await pull('A', BAY.single)
    status.value = 'A1 fährt zum Laufwerk'
    await gripTo('D')
    await load()
  },
  async () => { // dismount: the cartridge goes back to a T1 position
    status.value = 'Laufwerk gibt A1 frei'
    await unload()
    status.value = 'Greifer fährt zurück zu Slot 1'
    await gripTo('A')
    status.value = 'A1 schiebt die anderen nach hinten'
    await push('A')
  },
  async () => { // mount from T3: the two cartridges in front have to be moved away first
    clock = { to: 42, est: 7300 }
    status.value = 'A3 liegt hinter A1 und A2'
    await wait(650)
    status.value = 'Greifer räumt A1 und A2 aus'
    await pull('A', BAY.left)
    await pull('A', BAY.right)
    status.value = 'Umlagern in Slot 2'
    await gripTo('B')
    await push('B')
    await push('B')
    status.value = 'Zurück zu Slot 1: A3 liegt jetzt vorn'
    await gripTo('A')
    await pull('A', BAY.single)
    status.value = 'A3 fährt zum Laufwerk'
    await gripTo('D')
    await load()
  },
]

async function play(k: number) {
  layout(k - 1) // always start from the clean end state of the previous click
  const my = gen
  seqMs = 0; clock = null
  setClock(END[k].clock === null ? null : 0)
  await wait(120)
  await scripts[k]()
  if (my !== gen) return
  clock = null
  status.value = END[k].status
  setClock(END[k].clock)
}

const reduced = () => window.matchMedia?.('(prefers-reduced-motion: reduce)').matches
watch(() => props.step, (n, o) => {
  if (n === o + 1 && n < scripts.length && !reduced()) play(n)
  else layout(n)
})
// leaving the slide stops a running animation and leaves the finished picture behind
const active = useIsSlideActive()
watch(active, (on) => { if (!on) layout(props.step) })
layout(props.step)
onMounted(() => layout(props.step))
onBeforeUnmount(() => { gen++ })

const target = computed(() => props.step >= 3 ? 'a3' : props.step >= 1 ? 'a1' : '')
const parked = computed(() => props.step >= 3 ? ['a1', 'a2'] : [])

const steps = [
  { t: 'Aufbau', d: 'Bis zu 9 Kassetten hintereinander, eine Feder drückt sie nach vorn. Greifbar ist nur T1.', chip: '9 Tiers' },
  { t: 'Mount aus T1', d: 'Der Greifer zieht die vorderste Kassette heraus, der Rest rutscht nach.', chip: 'Ø 19 s' },
  { t: 'Zurück nach vorn', d: 'Nach dem Entladen kommt die Kassette auf einen T1-Platz, die anderen rücken nach hinten.', chip: 'T1' },
  { t: 'Mount aus der Tiefe', d: 'Erst müssen die Kassetten davor in einen anderen Slot umgelagert werden. T9: Ø 104 s.', chip: 'T3 Ø 42 s' },
]
</script>

<template>
  <SlideFrame :section="5" title="So funktioniert die Diamondback" kicker="Diamondback · Kassetten in 9 Ebenen">
    <div class="wrap">
      <div class="card stage">
        <svg viewBox="0 0 900 400">
          <!-- drive -->
          <g class="drive" :class="{ busy }">
            <rect x="30" y="20" width="160" height="72" rx="12" />
            <text x="110" y="14" text-anchor="middle" class="lbl">Laufwerk</text>
            <circle cx="170" cy="38" r="5" class="led" />
          </g>
          <!-- accessor rail + gripper -->
          <rect x="30" y="100" width="160" height="290" rx="12" class="rail" />
          <text x="110" y="384" text-anchor="middle" class="lbl">Greifer</text>
          <g ref="gripEl" class="gripper" :class="{ hold: gripping }">
            <rect width="148" height="62" rx="10" />
            <path d="M148 12 h10 M148 50 h10" class="claw" />
          </g>

          <!-- two slots -->
          <g v-for="row in (['A', 'B'] as const)" :key="row">
            <text :x="X0 - 8" :y="ROW[row] - 20" class="lbl">{{ row === 'A' ? 'Slot 1' : 'Slot 2' }}</text>
            <rect :x="X0 - 10" :y="ROW[row] - 10" :width="WALL - X0 + 14" :height="CH + 20" rx="10" class="slot" />
            <line :x1="WALL" :x2="WALL" :y1="ROW[row] - 10" :y2="ROW[row] + CH + 10" class="wall" />
            <text v-for="i in tiers" :key="i" :x="X0 + i * P + CW / 2" :y="ROW[row] + CH + 28" text-anchor="middle" class="tier" :class="{ front: i === 0 }">T{{ i + 1 }}</text>
          </g>
          <text :x="X0 - 8" y="114" class="hint">← vorne</text>
          <text :x="WALL" y="114" text-anchor="end" class="hint">Feder →</text>

          <!-- springs -->
          <g v-for="row in (['A', 'B'] as const)" :key="row" :ref="el => (springEls[row] = el as SVGGElement | null)" class="spring">
            <polyline :points="COIL" vector-effect="non-scaling-stroke" />
          </g>

          <!-- cartridges -->
          <g
            v-for="id in ids" :key="id" :ref="el => (cartEls[id] = el as SVGGElement | null)" class="cart"
            :class="{ target: id === target, parked: parked.includes(id) }"
          >
            <rect :width="CW" :height="CH" rx="6" />
            <rect x="6" y="30" :width="CW - 12" height="10" rx="2" class="tag" />
            <text :x="CW / 2" y="22" text-anchor="middle">{{ id.toUpperCase() }}</text>
          </g>
        </svg>

        <!-- what is happening right now, and the measured mount time it stands for -->
        <div class="now">
          <span class="what">{{ status }}</span>
          <span v-show="clockOn" class="clock mono"><b ref="clockEl">0</b> s<i>Ø Mount-Zeit</i></span>
        </div>
      </div>

      <ol class="steps">
        <li v-for="(s, i) in steps" :key="s.t" :class="{ on: i === Math.min(step, 3), done: i < step }">
          <span class="n">{{ i + 1 }}</span>
          <div>
            <div class="head"><b>{{ s.t }}</b><span class="chip">{{ s.chip }}</span></div>
            <p>{{ s.d }}</p>
          </div>
        </li>
      </ol>
    </div>
  </SlideFrame>
</template>

<style scoped>
.wrap { height: 100%; display: grid; grid-template-columns: 1fr 330px; gap: 16px; }
.stage { position: relative; display: flex; align-items: center; padding: 10px 14px; min-height: 0; }
.now { position: absolute; top: 14px; right: 18px; display: flex; align-items: center; gap: 10px; }
.what { padding: 7px 13px; border-radius: 999px; font-size: 14px; font-weight: 600; color: var(--ink); background: var(--surface-2); }
.clock { display: inline-flex; align-items: baseline; gap: 4px; padding: 5px 13px; border-radius: 999px; font-size: 13px; font-weight: 600; color: #1d4fd8; background: var(--accent-soft); }
.clock b { min-width: 2ch; font-size: 19px; font-weight: 700; text-align: right; font-variant-numeric: tabular-nums; }
.clock i { margin-left: 6px; font-style: normal; font-size: 11px; color: var(--muted); }
.stage svg { width: 100%; height: 100%; }

.lbl { font: 600 13px 'JetBrains Mono', monospace; fill: var(--muted); }
.hint { font: 600 12px 'JetBrains Mono', monospace; fill: #94a3b8; }
.tier { font: 600 12px 'JetBrains Mono', monospace; fill: #94a3b8; }
.tier.front { fill: var(--accent); }
.drive rect { fill: var(--surface-2); stroke: #c3cbd7; stroke-width: 1.5; transition: fill 0.4s, stroke 0.4s; }
.drive.busy rect { fill: var(--teal-soft); stroke: var(--teal); }
.drive .led { fill: #c3cbd7; transition: fill 0.3s; }
.drive.busy .led { fill: var(--teal); animation: led 0.9s ease-in-out infinite; }
@keyframes led { 50% { opacity: 0.35; } }
.rail { fill: #f6f8fb; stroke: #dfe4eb; stroke-dasharray: 5 5; }
.gripper rect { fill: #fff; stroke: var(--ink-2); stroke-width: 2; transition: fill 0.25s, stroke 0.25s; }
.gripper .claw { fill: none; stroke: var(--ink-2); stroke-width: 3; stroke-linecap: round; transition: stroke 0.25s; }
.gripper.hold rect { fill: var(--accent-soft); stroke: var(--accent); }
.gripper.hold .claw { stroke: var(--accent); }
.slot { fill: #f6f8fb; stroke: #dfe4eb; }
.wall { stroke: var(--ink-2); stroke-width: 4; stroke-linecap: round; }
.spring polyline { fill: none; stroke: #94a3b8; stroke-width: 2; stroke-linejoin: round; }

.cart rect { fill: #fff; stroke: #b6c0ce; stroke-width: 1.5; transition: fill 0.3s, stroke 0.3s; }
.cart .tag { fill: #e2e6ec; stroke: none; }
.cart text { font: 700 13px 'JetBrains Mono', monospace; fill: var(--ink-2); }
.cart.target rect:first-child { fill: var(--accent-soft); stroke: var(--accent); stroke-width: 2.5; }
.cart.target .tag { fill: var(--accent); }
.cart.target text { fill: #1d4fd8; }
.cart.parked rect:first-child { fill: var(--amber-soft); stroke: var(--amber); stroke-width: 2; }
.cart.parked .tag { fill: var(--amber); }

.steps { margin: 0; padding: 0; list-style: none; display: flex; flex-direction: column; gap: 8px; min-height: 0; }
.steps li > div { flex: 1; }
.head { display: flex; align-items: center; justify-content: space-between; gap: 8px; }
.steps li { display: flex; gap: 12px; padding: 10px 13px; border-radius: 14px; border: 1px solid var(--line); background: var(--surface); opacity: 0.45; transition: opacity 0.4s, border-color 0.4s, box-shadow 0.4s; }
.steps li.done { opacity: 0.75; }
.steps li.on { opacity: 1; border-color: var(--accent); box-shadow: 0 0 0 2px var(--accent-soft); }
.n { flex: none; display: grid; place-items: center; width: 26px; height: 26px; border-radius: 8px; font: 700 13px 'JetBrains Mono', monospace; color: var(--accent); background: var(--accent-soft); }
.steps b { font-size: 15.5px; color: var(--ink); }
.steps p { margin: 3px 0 0; font-size: 13px; line-height: 1.38; color: var(--ink-2); opacity: 1; }
.chip { display: inline-block; padding: 2px 8px; border-radius: 999px; font: 600 11.5px 'JetBrains Mono', monospace; color: #1d4fd8; background: var(--accent-soft); }
</style>
