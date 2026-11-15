<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue'
import { useIsSlideActive } from '@slidev/client'

// HL-LHC challenges and the LS3 development plan from the CTA Roadmap talk
const numbers = [
  { big: '2 EB', text: 'pro Jahr archivieren' },
  { big: '300 GB/s', text: 'Durchsatz (5 × 60 GB/s)' },
  { big: '8 → 12 EB', text: 'Rohdaten in Run-4 → Run-5' },
  { big: '2–3 Wochen', text: 'bis 150 PB neue Bänder voll sind' },
]
const plans = [
  { title: 'Schnellere Auftragsverteilung', text: 'Neuer Scheduler: Die Tape-Server holen sich ihre Aufträge, ohne sich gegenseitig zu blockieren' },
  { title: 'Leere Bänder automatisch nachfüllen', text: 'CTA füllt die Band-Pools selbst auf und nimmt Bänder, für die gerade ein Laufwerk frei ist' },
  { title: 'Datensätze zusammen speichern', text: 'Rucio (ATLAS, CMS) meldet, welche Dateien zu einem Datensatz gehören. CTA schreibt sie gemeinsam auf Band, damit das Zurücklesen schneller geht' },
]

// Drive efficiency gauge: measured 45 % (CMS recall test), goal 80 %
const MEASURED = 45
const GOAL = 80
const shown = ref(0)
// Half-circle gauge from 180° to 0°
const R = 110
const arc = (pct: number) => {
  const a = Math.PI * (1 - pct / 100)
  return `M ${130 - R} 140 A ${R} ${R} 0 0 1 ${130 + R * Math.cos(a)} ${140 - R * Math.sin(a)}`
}
const goalPt = computed(() => {
  const a = Math.PI * (1 - GOAL / 100)
  return { x: 130 + R * Math.cos(a), y: 140 - R * Math.sin(a) }
})

// Schematic time line of one drive: grey = mounting / winding, green = actually reading.
// Only the reading share (45 % vs 80 %) comes from the source; the split into blocks is illustrative.
type Seg = { w: number; read: boolean }
const lanes: { label: string; pct: number; segs: Seg[] }[] = [
  { label: 'Heute', pct: MEASURED, segs: [{ w: 12, read: false }, { w: 9, read: true }, { w: 9, read: false }, { w: 14, read: true }, { w: 13, read: false }, { w: 10, read: true }, { w: 21, read: false }, { w: 12, read: true }] },
  { label: 'Ziel', pct: GOAL, segs: [{ w: 8, read: false }, { w: 38, read: true }, { w: 12, read: false }, { w: 42, read: true }] },
]
// Playhead sweeps once across both lanes, revealing the blocks
const sweep = ref(0)

// Slidev mounts slides ahead of time, so onMounted would finish the animation before
// the slide is visible. Start it whenever the slide becomes active, reset when it leaves.
const active = useIsSlideActive()
let raf = 0
const run = () => {
  cancelAnimationFrame(raf)
  shown.value = 0
  sweep.value = 0
  const t0 = performance.now()
  const step = (now: number) => {
    const p = Math.min(1, Math.max(0, (now - t0 - 500) / 1400))
    shown.value = MEASURED * (1 - (1 - p) ** 3)
    sweep.value = Math.min(100, Math.max(0, (now - t0 - 900) / 28))
    if (p < 1 || sweep.value < 100)
      raf = requestAnimationFrame(step)
  }
  raf = requestAnimationFrame(step)
}
onMounted(() => watch(active, (on) => {
  if (on)
    run()
  else {
    cancelAnimationFrame(raf)
    shown.value = 0
    sweep.value = 0
  }
}, { immediate: true }))
onBeforeUnmount(() => cancelAnimationFrame(raf))
const segStyle = (segs: Seg[], i: number) => {
  const left = segs.slice(0, i).reduce((s, x) => s + x.w, 0)
  const vis = Math.min(segs[i].w, Math.max(0, sweep.value - left))
  return { left: `${left}%`, width: `${vis}%` }
}
</script>

<template>
  <SlideFrame :section="4" title="Die Herausforderung: Run-4 und HL-LHC" kicker="Roadmap · Long Shutdown 3 (2027–2029)">
    <div class="layout">
      <div class="numbers">
        <div v-for="(n, i) in numbers" :key="n.big" class="card num" v-motion :initial="{ opacity: 0, y: 14 }" :enter="{ opacity: 1, y: 0, transition: { delay: i * 100, duration: 450 } }">
          <b>{{ n.big }}</b><span>{{ n.text }}</span>
        </div>
        <div class="note">Die Kapazität wächst schneller als der Durchsatz: In der LTO-Roadmap kommen pro Generation +66 % dazu statt +100 %.</div>
      </div>

      <div class="card gauge">
        <div class="g-title">Drive Efficiency beim Recall</div>
        <div class="g-sub">Recall = Daten vom Band zurücklesen. Wie viel der Laufwerkszeit wird wirklich gelesen?</div>
        <svg viewBox="0 0 260 150">
          <path :d="arc(100)" class="track" />
          <path :d="arc(shown)" class="value" />
          <circle :cx="goalPt.x" :cy="goalPt.y" r="7" class="goal" />
          <text x="130" y="128" text-anchor="middle" class="pct">{{ Math.round(shown) }} %</text>
          <text :x="goalPt.x + 12" :y="goalPt.y - 6" class="goal-t">Ziel 80 %</text>
        </svg>

        <div class="formula">
          <span class="f-chip">Mount-Effizienz</span><i>×</i><span class="f-chip">Transfer-Effizienz</span>
        </div>

        <div class="lanes">
          <div v-for="l in lanes" :key="l.label" class="lane">
            <span class="l-label">{{ l.label }}</span>
            <div class="bar">
              <i v-for="(s, i) in l.segs" :key="i" class="seg" :class="{ read: s.read }" :style="segStyle(l.segs, i)" />
              <b class="head" :style="{ left: `${sweep}%` }" :class="{ gone: sweep >= 100 }" />
            </div>
            <span class="l-pct" :class="{ goal: l.pct === GOAL }">{{ l.pct }} %</span>
          </div>
          <div class="legend">
            <span><i class="sw" />Mounten, Spulen, Warten</span>
            <span><i class="sw read" />Lesen</span>
          </div>
        </div>

        <p>Gemessen bei CMS-Recall-Tests. Ziel: Was zusammen gelesen wird, liegt zusammen auf Band.</p>
      </div>

      <div class="plans">
        <div class="p-head">Plan für LS3</div>
        <div v-for="(p, i) in plans" :key="p.title" class="card plan" v-motion :initial="{ opacity: 0, x: 20 }" :enter="{ opacity: 1, x: 0, transition: { delay: 700 + i * 150, duration: 450 } }">
          <span class="idx mono">{{ i + 1 }}</span>
          <div><b>{{ p.title }}</b><p>{{ p.text }}</p></div>
        </div>
      </div>
    </div>
  </SlideFrame>
</template>

<style scoped>
.layout { height: 100%; display: grid; grid-template-columns: 1fr 1fr 1.15fr; gap: 18px; }
.numbers { display: grid; grid-template-rows: repeat(4, 1fr) auto; gap: 10px; }
.num { display: flex; flex-direction: column; justify-content: center; padding: 10px 18px; }
.num b { font-size: 30px; font-weight: 800; letter-spacing: -0.03em; color: var(--rose); }
.num span { font-size: 14px; color: var(--ink-2); }
.note { font-size: 13px; line-height: 1.45; color: var(--muted); }

.gauge { display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 10px; padding: 16px 18px; text-align: center; }
.g-title { font-size: 16px; font-weight: 650; color: var(--ink); }
.g-sub { font-size: 12.5px; line-height: 1.4; color: var(--muted); }
.gauge svg { width: 100%; max-width: 210px; overflow: visible; }

.formula { display: flex; align-items: center; gap: 6px; font-size: 12px; }
.formula i { font-style: normal; font-weight: 700; color: var(--muted); }
.f-chip { padding: 3px 9px; border-radius: 999px; font-weight: 600; color: #9a6400; background: var(--amber-soft); }

.lanes { width: 100%; display: grid; gap: 7px; }
.lane { display: grid; grid-template-columns: 40px 1fr 38px; align-items: center; gap: 8px; }
.l-label { font-size: 12px; font-weight: 600; color: var(--ink-2); text-align: left; }
.l-pct { font: 700 13px 'Inter', sans-serif; color: #9a6400; text-align: right; }
.l-pct.goal { color: var(--green); }
.bar { position: relative; height: 16px; border-radius: 5px; background: var(--surface-2); overflow: hidden; }
.seg { position: absolute; top: 0; bottom: 0; background: #c3cad6; border-right: 1px solid var(--surface); }
.seg.read { background: var(--green); }
.head { position: absolute; top: -2px; bottom: -2px; width: 2px; margin-left: -1px; background: var(--ink); transition: opacity 0.3s; }
.head.gone { opacity: 0; }
.legend { display: flex; justify-content: center; gap: 14px; font-size: 11.5px; color: var(--muted); }
.legend span { display: flex; align-items: center; gap: 5px; }
.sw { width: 10px; height: 10px; border-radius: 3px; background: #c3cad6; }
.sw.read { background: var(--green); }
.track { fill: none; stroke: var(--surface-2); stroke-width: 22; stroke-linecap: round; }
.value { fill: none; stroke: var(--amber); stroke-width: 22; stroke-linecap: round; }
.goal { fill: var(--surface); stroke: var(--green); stroke-width: 4; }
.pct { font: 800 40px 'Inter', sans-serif; fill: var(--ink); letter-spacing: -0.03em; }
.goal-t { font: 700 13px 'Inter', sans-serif; fill: var(--green); }
.gauge p { margin: 0; font-size: 13px; line-height: 1.45; color: var(--ink-2); opacity: 1; }

.plans { display: flex; flex-direction: column; gap: 10px; }
.p-head { font: 600 12px 'JetBrains Mono', monospace; letter-spacing: 0.1em; text-transform: uppercase; color: var(--amber); }
.plan { flex: 1; display: flex; gap: 14px; align-items: flex-start; padding: 14px 16px; }
.idx { flex: none; display: grid; place-items: center; width: 30px; height: 30px; border-radius: 9px; font-weight: 700; color: #9a6400; background: var(--amber-soft); }
.plan b { font-size: 16px; color: var(--ink); }
.plan p { margin-top: 4px; font-size: 13.5px; line-height: 1.45; color: var(--ink-2); opacity: 1; }
</style>
