<script setup lang="ts">
// A detailed tape library for the closing slide. A robot (mast + gripper) works through
// a small job list in a loop: fetch a cartridge, scan its barcode, load it into a drive,
// read or write (data flows to/from the disk buffer), unload and put it back.
import { onBeforeUnmount, onMounted, reactive, ref, watch } from 'vue'
import { useIsSlideActive } from '@slidev/client'

// ---- geometry (SVG units) ----
const COLS = [56, 112, 168, 224]
const ROWS = Array.from({ length: 8 }, (_, r) => 96 + r * 40)
const CW = 48, CH = 28 // cartridge
const DRIVE_Y = [104, 196, 288]
const HOME = { x: 278, y: 208 }
const PASTEL = ['#dbe6ff', '#d6f2ec', '#e6e0ff', '#ffeccc', '#e9edf2', '#fde2e2']

type Job = { id: string; volser: string; col: number; row: number; color: string }
const JOBS: Job[] = [
  { id: 'A', volser: 'L90012', col: 1, row: 2, color: '#ffb547' },
  { id: 'B', volser: 'L90037', col: 3, row: 5, color: '#7fe0c6' },
  { id: 'C', volser: 'L90105', col: 0, row: 6, color: '#a99bff' },
]
const EMPTY = new Set(['2-0', '0-3', '3-7', '2-6'])
const jobAt = (c: number, r: number) => JOBS.find(j => j.col === c && j.row === r)

// static cartridges (everything except the three that move)
const fillers = COLS.flatMap((x, c) => ROWS.map((y, r) => ({ key: `${c}-${r}`, x, y, c, r })))
  .filter(s => !EMPTY.has(s.key) && !jobAt(s.c, s.r))
  .map((s, i) => ({ ...s, color: PASTEL[(s.c * 3 + s.r) % PASTEL.length], volser: `L9${String(100 + i * 7).padStart(4, '0')}` }))

// ---- state ----
// Discrete state (texts, LEDs, classes) is reactive. Positions are not: they are written
// straight to the SVG nodes once per frame, all from one clock, so the gripper, the mast
// and the cartridge it carries can never drift apart.
type DriveState = { mode: 'read' | 'write' | null; run: boolean; done: boolean }
const idleDrive = (): DriveState => ({ mode: null, run: false, done: false })
const st = reactive({
  drives: [idleDrive(), idleDrive(), idleDrive()],
  lcd: 'BEREIT',
  scan: false,
  hold: false,
  mounts: 1284,
})

const slotPos = (j: Job) => ({ x: COLS[j.col], y: ROWS[j.row] })
const g = { ...HOME }                                   // gripper
const pos: Record<string, { x: number; y: number }> = {} // moving cartridges
let held: string | null = null                           // cartridge travelling with the gripper

const mastEl = ref<SVGGElement>()
const gripEl = ref<SVGGElement>()
const cartEls: Record<string, SVGGElement | null> = {}
const barEls: (SVGRectElement | null)[] = []

function place() {
  if (held) pos[held] = { x: g.x + 4, y: g.y + 5 }
  mastEl.value?.setAttribute('transform', `translate(${g.x + 25} 0)`)
  gripEl.value?.setAttribute('transform', `translate(${g.x} ${g.y})`)
  for (const j of JOBS) cartEls[j.id]?.setAttribute('transform', `translate(${pos[j.id].x} ${pos[j.id].y})`)
}
const setBar = (d: number, p: number) => { const el = barEls[d]; if (el) el.style.transform = `scaleX(${p})` }

// ---- clock ----
// Every motion and every pause runs through animate(). A new generation (slide left,
// component unmounted) silently stops everything that is still pending.
let gen = 0
const ease = (p: number) => (p < 0.5 ? 4 * p * p * p : 1 - (-2 * p + 2) ** 3 / 2)
function animate(ms: number, step?: (p: number) => void) {
  const my = gen
  return new Promise<void>((resolve) => {
    let elapsed = 0
    let last: number | undefined
    const tick = (now: number) => {
      if (my !== gen) return
      if (last !== undefined) elapsed += Math.min(now - last, 50) // no jumps after a hidden tab
      last = now
      const p = Math.min(elapsed / ms, 1)
      step?.(p)
      if (p < 1) requestAnimationFrame(tick)
      else resolve()
    }
    requestAnimationFrame(tick)
  })
}
const wait = (ms: number) => animate(ms)

// Travel time grows with the distance, so short hops are not sluggish and long ones not rushed.
function moveTo(x: number, y: number) {
  const x0 = g.x, y0 = g.y
  const dist = Math.hypot(x - x0, y - y0)
  if (dist < 0.5) return Promise.resolve()
  return animate(380 + dist * 3.6, (p) => {
    const e = ease(p)
    g.x = x0 + (x - x0) * e
    g.y = y0 + (y - y0) * e
    place()
  })
}
function slideCart(j: Job, x: number, y: number, ms = 520) {
  const { x: x0, y: y0 } = pos[j.id]
  return animate(ms, (p) => {
    const e = ease(p)
    pos[j.id] = { x: x0 + (x - x0) * e, y: y0 + (y - y0) * e }
    place()
  })
}

// ---- job script ----
const slotGrip = (j: Job) => [COLS[j.col] - 4, ROWS[j.row] - 5] as const
const driveGrip = (d: number) => [278, DRIVE_Y[d] + 13] as const
const driveSeat = (d: number) => [338, DRIVE_Y[d] + 18] as const
const finished: Promise<void>[] = []

async function mount(j: Job, d: number, mode: 'read' | 'write', ms: number) {
  st.lcd = `MOUNT ${j.volser} → LW ${d + 1}`
  await moveTo(...slotGrip(j))
  st.scan = true; await wait(450); st.scan = false
  held = j.id; st.hold = true
  await wait(260)
  await moveTo(...driveGrip(d))
  held = null
  await slideCart(j, ...driveSeat(d))
  st.hold = false
  st.mounts++
  st.drives[d] = { mode, run: true, done: false }
  st.lcd = mode === 'read' ? `LW ${d + 1} LIEST · 400 MB/s` : `LW ${d + 1} SCHREIBT · 400 MB/s`
  // the drive works on its own; its bar shows the real progress of this job
  finished[d] = animate(ms, p => setBar(d, p)).then(() => { st.drives[d] = { mode, run: false, done: true } })
  await wait(220)
}
async function unmount(j: Job, d: number) {
  if (!st.drives[d].done) {
    await moveTo(HOME.x, HOME.y) // nothing to do yet: park and wait for the drive
    await finished[d]
    await wait(350)
  }
  st.lcd = `UNLOAD ${j.volser} ← LW ${d + 1}`
  await moveTo(...driveGrip(d))
  st.hold = true
  await slideCart(j, driveGrip(d)[0] + 4, driveGrip(d)[1] + 5)
  held = j.id
  st.drives[d] = idleDrive(); setBar(d, 0)
  await wait(180)
  await moveTo(...slotGrip(j))
  held = null; pos[j.id] = slotPos(j); place()
  st.hold = false
  await wait(320)
}

async function loop() {
  for (;;) {
    await mount(JOBS[0], 0, 'read', 5200)
    await mount(JOBS[1], 1, 'write', 7000)
    await unmount(JOBS[0], 0)
    await mount(JOBS[2], 2, 'read', 4600)
    await unmount(JOBS[1], 1)
    await unmount(JOBS[2], 2)
    st.lcd = 'BEREIT'
    await moveTo(HOME.x, HOME.y)
    await wait(1500)
  }
}

function reset() {
  gen++
  held = null
  Object.assign(g, HOME)
  for (const j of JOBS) pos[j.id] = slotPos(j)
  st.drives = [idleDrive(), idleDrive(), idleDrive()]
  st.lcd = 'BEREIT'; st.scan = false; st.hold = false; st.mounts = 1284
  for (let d = 0; d < 3; d++) setBar(d, 0)
  place()
}
reset()

// Run only while this slide is on screen and always start from the idle state.
const active = useIsSlideActive()
onMounted(() => {
  watch(active, (on) => {
    reset()
    if (!on || window.matchMedia?.('(prefers-reduced-motion: reduce)').matches) return
    wait(1300).then(loop) // let the slide's entrance finish first
  }, { immediate: true })
})
onBeforeUnmount(() => { gen++ })
</script>

<template>
  <div class="library">
  <svg viewBox="0 0 520 450" aria-hidden="true">
    <defs>
      <linearGradient id="tl-shell" x1="0" y1="0" x2="1" y2="1">
        <stop offset="0" stop-color="#2a2f3a" />
        <stop offset="0.55" stop-color="#14171e" />
        <stop offset="1" stop-color="#07080b" />
      </linearGradient>
      <linearGradient id="tl-glass" x1="0" y1="0" x2="1" y2="1">
        <stop offset="0" stop-color="#fff" stop-opacity="0.07" />
        <stop offset="0.4" stop-color="#fff" stop-opacity="0" />
      </linearGradient>
      <linearGradient id="tl-light" x1="0" y1="0" x2="0" y2="1">
        <stop offset="0" stop-color="#2f6bff" stop-opacity="0.16" />
        <stop offset="1" stop-color="#2f6bff" stop-opacity="0" />
      </linearGradient>
      <pattern id="tl-barcode" width="6" height="6" patternUnits="userSpaceOnUse">
        <rect width="1" height="6" fill="#0b1220" />
        <rect x="2" width="2" height="6" fill="#0b1220" />
        <rect x="5" width="0.8" height="6" fill="#0b1220" />
      </pattern>
    </defs>

    <!-- cabinet and header -->
    <rect x="20" y="20" width="480" height="410" rx="24" fill="url(#tl-shell)" stroke="#3a404d" stroke-width="1.5" />
    <rect x="40" y="32" width="440" height="36" rx="9" fill="#f5f6f8" />
    <text x="54" y="54" class="label">CTA · LIBRARY</text>
    <rect x="182" y="38" width="216" height="24" rx="5" fill="#07130f" stroke="#1f3a31" />
    <circle cx="194" cy="50" r="3" class="lcd-dot" />
    <text x="204" y="54" class="lcd">{{ st.lcd }}</text>
    <text x="392" y="54" text-anchor="end" class="lcd dim">#{{ st.mounts.toLocaleString('de-DE') }}</text>
    <text x="468" y="47" text-anchor="end" class="label small">DISK</text>
    <text x="468" y="59" text-anchor="end" class="label small dim">BUFFER</text>

    <!-- bay -->
    <rect x="40" y="80" width="440" height="340" rx="12" fill="#0d1016" />
    <rect x="40" y="80" width="440" height="120" rx="12" fill="url(#tl-light)" />

    <!-- storage slots -->
    <g v-for="(x, c) in COLS" :key="`col${c}`">
      <rect v-for="(y, r) in ROWS" :key="`s${c}-${r}`" :x="x - 3" :y="y - 3" :width="CW + 6" :height="CH + 6" rx="5" class="slot" />
    </g>
    <g v-for="f in fillers" :key="f.key" :transform="`translate(${f.x} ${f.y})`" class="cart">
      <rect :width="CW" :height="CH" rx="4" class="body" />
      <rect x="5" y="5" :width="CW - 10" :height="CH - 10" rx="2" :fill="f.color" />
      <rect x="8" :y="CH - 11" :width="CW - 16" height="5" fill="url(#tl-barcode)" opacity="0.55" />
    </g>

    <!-- drives -->
    <g v-for="(y, d) in DRIVE_Y" :key="`drv${d}`" class="drive" :class="[st.drives[d].mode, { run: st.drives[d].run, done: st.drives[d].done }]">
      <rect x="330" :y="y" width="140" height="64" rx="9" class="case" />
      <rect x="334" :y="y + 14" width="56" height="36" rx="5" class="mouth" />
      <rect x="396" :y="y + 8" width="54" height="30" rx="5" class="window" />
      <g class="reel">
        <circle cx="411" :cy="y + 23" r="10" /><path :d="`M411 ${y + 14} v18 M402 ${y + 23} h18`" />
      </g>
      <g class="reel">
        <circle cx="435" :cy="y + 23" r="10" /><path :d="`M435 ${y + 14} v18 M426 ${y + 23} h18`" />
      </g>
      <circle cx="460" :cy="y + 14" r="3.5" class="led" />
      <rect x="396" :y="y + 43" width="66" height="4" rx="2" class="track" />
      <rect x="396" :y="y + 43" width="66" height="4" rx="2" class="bar" :ref="el => (barEls[d] = el as SVGRectElement | null)" />
      <text x="336" :y="y + 59" class="dtext">LW {{ d + 1 }}<tspan v-if="st.drives[d].run"> · {{ st.drives[d].mode === 'read' ? 'LESEN' : 'SCHREIBEN' }}</tspan><tspan v-else-if="st.drives[d].done"> · FERTIG</tspan></text>
      <!-- data to/from the disk buffer -->
      <path :d="`M470 ${y + 32} H488 V70`" class="cable" />
      <circle v-for="k in 3" :key="k" r="2.6" class="packet" :style="{ offsetPath: `path('M470 ${y + 32} H488 V70')`, animationDelay: `${(k - 1) * 0.33}s` }" />
    </g>

    <!-- I/O station -->
    <rect x="330" y="372" width="140" height="38" rx="8" class="io" />
    <rect v-for="k in 3" :key="`io${k}`" :x="340 + (k - 1) * 30" y="380" width="22" height="22" rx="3" class="io-slot" />
    <text x="436" y="395" text-anchor="middle" class="dtext">I/O</text>

    <!-- robot: rails, mast, gripper -->
    <rect x="44" y="82" width="432" height="4" rx="2" class="rail" />
    <rect x="44" y="414" width="432" height="4" rx="2" class="rail" />
    <g ref="mastEl" class="mast" :transform="`translate(${HOME.x + 25} 0)`">
      <rect x="0" y="84" width="6" height="332" rx="2" />
    </g>

    <!-- moving cartridges (drawn above the mast, below the gripper frame) -->
    <g v-for="m in JOBS" :key="m.id" :ref="el => (cartEls[m.id] = el as SVGGElement | null)" class="cart mover" :transform="`translate(${COLS[m.col]} ${ROWS[m.row]})`">
      <rect :width="CW" :height="CH" rx="4" class="body" />
      <rect x="5" y="5" :width="CW - 10" :height="CH - 10" rx="2" :fill="m.color" />
      <text :x="CW / 2" y="14" text-anchor="middle" class="volser">{{ m.volser }}</text>
      <rect x="8" :y="CH - 11" :width="CW - 16" height="5" fill="url(#tl-barcode)" opacity="0.7" />
    </g>

    <g ref="gripEl" class="gripper" :class="{ scan: st.scan, hold: st.hold }" :transform="`translate(${HOME.x} ${HOME.y})`">
      <rect width="56" height="38" rx="6" class="frame" />
      <rect x="20" y="-9" width="16" height="9" rx="2" class="head" />
      <rect x="54" y="13" width="5" height="12" rx="1.5" class="scanner" />
      <rect x="4" y="21" width="48" height="1.6" class="beamline" />
    </g>

    <!-- front glass -->
    <rect x="40" y="80" width="440" height="340" rx="12" fill="url(#tl-glass)" pointer-events="none" />
  </svg>
  </div>
</template>

<style scoped>
/* The shadow is a static box behind the cabinet. A drop-shadow filter on the SVG would be
   re-rendered on every animation frame. */
.library { position: relative; }
.library::before {
  content: '';
  position: absolute;
  inset: 4.44% 3.85%;
  border-radius: 5% / 5.85%;
  box-shadow: 0 30px 60px -6px rgba(15, 23, 42, 0.4);
}
.library svg { position: relative; display: block; width: 100%; height: auto; }

.label { font: 700 12px 'JetBrains Mono', monospace; letter-spacing: 0.1em; fill: #0b1220; }
.label.small { font-size: 8.5px; letter-spacing: 0.08em; }
.label.dim { fill: #64748b; }
.lcd { font: 600 10px 'JetBrains Mono', monospace; letter-spacing: 0.04em; fill: #4ade80; }
.lcd.dim { fill: #2f7a50; }
.lcd-dot { fill: #4ade80; animation: blink 1.4s ease-in-out infinite; }

.slot { fill: #141922; stroke: #232a36; }
.cart .body { fill: #1c212b; stroke: #313a49; }
.volser { font: 700 7.5px 'JetBrains Mono', monospace; fill: #0b1220; }

.mast rect { fill: #2a3242; }
.rail { fill: #2c3340; }
.gripper .frame { fill: rgba(47, 107, 255, 0.08); stroke: #2f6bff; stroke-width: 2.5; transition: fill 0.2s, stroke 0.2s; }
.gripper.hold .frame { fill: rgba(47, 107, 255, 0.22); stroke: #6f9bff; }
.gripper .head { fill: #2f6bff; }
.gripper .scanner { fill: #e5484d; }
.gripper .beamline { fill: #ff4d4f; opacity: 0; }
.gripper.scan .beamline { animation: scanline 0.45s ease-in-out; }
.gripper.scan .scanner { filter: drop-shadow(0 0 4px #ff4d4f); }
@keyframes scanline { 0% { opacity: 0; transform: translateY(-12px); } 30% { opacity: 1; } 100% { opacity: 0; transform: translateY(10px); } }

.drive .case { fill: #1b1f27; stroke: #3a404d; }
.drive .mouth { fill: #07080b; }
.drive .window { fill: #0b0e13; stroke: #2c3340; }
.drive .reel circle { fill: #1c212b; stroke: #4b5567; stroke-width: 1.5; }
.drive .reel path { stroke: #4b5567; stroke-width: 1.5; }
.drive .reel { transform-box: fill-box; transform-origin: center; }
.drive.run .reel { animation: spin 1s linear infinite; }
.drive.run.write .reel { animation-direction: reverse; }
.drive .led { fill: #3a404d; transition: fill 0.3s; }
.drive.run .led { fill: #00a389; animation: blink 0.5s ease-in-out infinite; }
.drive.done .led { fill: #00a389; }
.drive .track { fill: #232a36; }
.drive .bar { fill: #00a389; transform: scaleX(0); transform-box: fill-box; transform-origin: left center; }
.drive.write .bar { fill: #f2a93b; }
.dtext { font: 600 8.5px 'JetBrains Mono', monospace; letter-spacing: 0.08em; fill: #8090a6; }
.drive .cable { fill: none; stroke: #232a36; stroke-width: 2; }
.drive .packet { fill: #4ade80; opacity: 0; offset-rotate: 0deg; }
.drive.run .packet { animation: flow 1s linear infinite; }
.drive.run.write .packet { fill: #f2a93b; animation-direction: reverse; }
@keyframes flow { 0% { offset-distance: 0%; opacity: 0; } 15% { opacity: 1; } 85% { opacity: 1; } 100% { offset-distance: 100%; opacity: 0; } }

.io { fill: #161b23; stroke: #2c3340; }
.io-slot { fill: #0b0e13; stroke: #2c3340; stroke-dasharray: 2 2; }

@keyframes spin { to { transform: rotate(360deg); } }
@keyframes blink { 50% { opacity: 0.3; } }

@media (prefers-reduced-motion: reduce) {
  .lcd-dot { animation: none; }
}
</style>
