<script setup lang="ts">
// v5 vs v6 Scheduler-DB: objectstore on Ceph (Rados) vs. PostgreSQL with three queue tables.
// Based on "CTA Scheduling with PostgreSQL" (J. Guenther) and the CTA roadmap, CTA Workshop 2026.
// Both panels play the same story in sync: one archive job ("Datei C") is queued,
// picked up by a tape daemon, written, and finally removed and reported.
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

const CYCLE = 8.6
const t = ref(0)
let raf = 0
onMounted(() => {
  const start = performance.now()
  const step = (now: number) => {
    t.value = ((now - start) / 1000) % CYCLE
    raf = requestAnimationFrame(step)
  }
  raf = requestAnimationFrame(step)
})
onBeforeUnmount(() => cancelAnimationFrame(raf))

const clamp = (v: number) => Math.min(1, Math.max(0, v))
const ease = (x: number) => x * x * (3 - 2 * x)
const prog = (a: number, b: number) => ease(clamp((t.value - a) / (b - a)))
const within = (a: number, b: number) => t.value >= a && t.value < b
type Pt = [number, number]
const lerp = (p: Pt, q: Pt, f: number): Pt => [p[0] + (q[0] - p[0]) * f, p[1] + (q[1] - p[1]) * f]

// ---------- Steps (shared by both panels) ----------
const steps = [
  { from: 0, to: 2, old: 'Frontend legt ein Request-Objekt an und trägt es in die ArchiveQueue ein.', new: 'Frontend schreibt per INSERT eine neue Zeile in PENDING.' },
  { from: 2, to: 4, old: 'Tape Daemon sperrt die Queue, nimmt den Auftrag heraus und schreibt sich als Besitzer ins Objekt.', new: 'Tape Daemon holt die Zeile: Sie wandert von PENDING nach ACTIVE.' },
  { from: 4, to: 6, old: 'Er schreibt aufs Band. Stürzt er ab, bleibt das Objekt mit seinem Namen liegen und muss aufgeräumt werden.', new: 'Er schreibt aufs Band. Die Zeile bleibt so lange in ACTIVE.' },
  { from: 6, to: CYCLE, old: 'Fertig: Request-Objekt wird gelöscht. Sperren, Suchen und Aufräumen ist alles eigener CTA-Code.', new: 'Fertig: DELETE und Ergebnis melden. Sperren und Suchen erledigt die Datenbank.' },
]
const stepIdx = computed(() => steps.findIndex(s => within(s.from, s.to)))

// ---------- Packets travelling along short paths ----------
type Hop = { from: Pt; to: Pt; a: number; b: number }
function dots(hops: Hop[]) {
  return hops.flatMap((h, i) => {
    if (!within(h.a, h.b))
      return []
    const [x, y] = lerp(h.from, h.to, prog(h.a, h.b))
    return [{ key: i, x, y }]
  })
}

// v5 (objectstore)
const oldDots = computed(() => dots([
  { from: [280, 64], to: [385, 196], a: 0.2, b: 1 }, // create Request C
  { from: [326, 222], to: [236, 214], a: 1, b: 1.6 }, // add pointer to queue
  { from: [280, 300], to: [150, 262], a: 2, b: 2.6 }, // daemon goes to the queue
  { from: [150, 262], to: [280, 300], a: 3.05, b: 3.55 }, // takes the job
  { from: [300, 300], to: [385, 250], a: 3.55, b: 4 }, // writes its name into the request
  { from: [300, 300], to: [385, 250], a: 6, b: 6.5 }, // done: delete the request
]))
const reqC = computed(() => ({
  show: t.value >= 1 && t.value < 7,
  opacity: t.value < 6.5 ? 1 : 1 - prog(6.5, 7),
  owner: t.value >= 4 ? 'TD 2' : '—',
  owned: t.value >= 4,
}))
const chipC = computed(() => t.value >= 1.6 && t.value < 3.05)
const locked = computed(() => within(2.6, 3.3))
const td2Old = computed(() => within(2, 6.6))
const writingOld = computed(() => within(4, 6))

// v6 (PostgreSQL)
const newDots = computed(() => dots([
  { from: [95, 64], to: [95, 200], a: 0.2, b: 1 }, // INSERT
  { from: [285, 64], to: [285, 128], a: 2, b: 2.6 }, // daemon asks for work
  { from: [285, 176], to: [285, 64], a: 3.4, b: 4 }, // row handed to the daemon
]))
const PENDING_SLOT: Pt = [30, 202]
const ACTIVE_SLOT: Pt = [220, 182]
const DONE_SLOT: Pt = [220, 326]
const row = computed(() => {
  if (t.value < 1 || t.value >= 6.8)
    return null
  let p: Pt = PENDING_SLOT
  if (t.value >= 2.6)
    p = lerp(PENDING_SLOT, ACTIVE_SLOT, prog(2.6, 3.4))
  if (t.value >= 6)
    p = lerp(ACTIVE_SLOT, DONE_SLOT, prog(6, 6.6))
  return { x: p[0], y: p[1], opacity: t.value < 6.4 ? 1 : 1 - prog(6.4, 6.8), active: t.value >= 3.4 }
})
const td2New = computed(() => within(2, 6.6))
const doneGlow = computed(() => within(6.4, 7.6))

const daemons = [60, 220, 380]
</script>

<template>
  <div class="cmp">
    <section class="panel old">
      <header><span class="ver">v5</span><b>Objectstore im Ceph-Cluster</b></header>
      <svg viewBox="0 0 560 360">
        <g class="actor">
          <rect x="180" y="16" width="200" height="48" rx="12" />
          <text x="280" y="45" text-anchor="middle" class="t">Frontend</text>
        </g>

        <g class="cluster">
          <rect x="40" y="96" width="480" height="180" rx="18" />
          <text x="58" y="118" class="s strong">Ceph · Rados: lauter einzelne Objekte</text>
        </g>

        <!-- Queue object: a list of pointers to request objects -->
        <g class="obj queue" :class="{ locked }">
          <rect x="60" y="132" width="176" height="128" rx="9" />
          <text x="76" y="152" class="ot">ArchiveQueue</text>
          <text x="76" y="166" class="os">Liste von Verweisen</text>
          <g class="ptr"><rect x="72" y="176" width="152" height="18" rx="5" /><text x="82" y="189">→ Request A</text></g>
          <g class="ptr"><rect x="72" y="198" width="152" height="18" rx="5" /><text x="82" y="211">→ Request B</text></g>
          <g v-if="chipC" class="ptr new"><rect x="72" y="220" width="152" height="18" rx="5" /><text x="82" y="233">→ Request C</text></g>
          <g class="lock" :class="{ on: locked }" transform="translate(214 146)">
            <rect x="-9" y="-3" width="18" height="14" rx="3" />
            <path d="M-5 -3 V-7 A5 5 0 0 1 5 -7 V-3" />
          </g>
        </g>

        <!-- Request objects: one per file -->
        <g class="obj req">
          <rect x="268" y="132" width="116" height="50" rx="9" />
          <text x="280" y="152" class="ot">Request A</text>
          <text x="280" y="170" class="os">Besitzer: TD 1</text>
        </g>
        <g class="obj req">
          <rect x="396" y="132" width="116" height="50" rx="9" />
          <text x="408" y="152" class="ot">Request B</text>
          <text x="408" y="170" class="os">Besitzer: —</text>
        </g>
        <g v-if="reqC.show" class="obj req new" :class="{ owned: reqC.owned }" :opacity="reqC.opacity">
          <rect x="328" y="198" width="116" height="50" rx="9" />
          <text x="340" y="218" class="ot">Request C</text>
          <text x="340" y="236" class="os">Besitzer: {{ reqC.owner }}</text>
        </g>

        <g v-for="(x, i) in daemons" :key="x" class="actor" :class="{ on: i === 1 && td2Old, busy: i === 1 && writingOld }">
          <rect :x="x" y="300" width="120" height="44" rx="11" />
          <text :x="x + 60" y="327" text-anchor="middle" class="t sm">Tape Daemon {{ i + 1 }}</text>
        </g>

        <circle v-for="d in oldDots" :key="d.key" :cx="d.x" :cy="d.y" r="6" class="dot old-dot" />
      </svg>
      <div class="caption">
        <span class="num">{{ stepIdx + 1 }}</span>
        <Transition name="cap" mode="out-in"><p :key="stepIdx">{{ steps[stepIdx]?.old }}</p></Transition>
      </div>
    </section>

    <div class="arrow">
      <svg viewBox="0 0 40 40"><path d="M6 20 H32 M24 11 L34 20 L24 29" /></svg>
    </div>

    <section class="panel new">
      <header><span class="ver">v6</span><b>PostgreSQL mit drei Tabellen</b></header>
      <svg viewBox="0 0 560 360">
        <defs>
          <marker id="sc-a-new" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0 0 L10 5 L0 10 z" fill="#7357ff" /></marker>
          <marker id="sc-a-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0 0 L10 5 L0 10 z" fill="#e5484d" /></marker>
        </defs>
        <g class="actor on">
          <rect x="20" y="16" width="150" height="48" rx="12" />
          <text x="95" y="45" text-anchor="middle" class="t">Frontend</text>
        </g>
        <g class="actor on" :class="{ busy: td2New }">
          <rect x="205" y="16" width="160" height="48" rx="12" />
          <text x="285" y="45" text-anchor="middle" class="t">Tape Daemon</text>
        </g>
        <path d="M95 64 L95 126" class="edge on" marker-end="url(#sc-a-new)" />
        <text x="103" y="98" class="elabel on">INSERT</text>
        <path d="M285 126 L285 64" class="edge on" marker-end="url(#sc-a-new)" />
        <text x="293" y="98" class="elabel on">holt Jobs</text>

        <rect x="8" y="114" width="544" height="188" rx="16" class="pg" />
        <text x="540" y="104" text-anchor="end" class="s strong pgl">PostgreSQL</text>

        <g v-for="tb in [{ x: 20, w: 150, n: 'PENDING', c: '1–10 Mio. Zeilen' }, { x: 210, w: 150, n: 'ACTIVE', c: '10–100 Tsd.' }, { x: 395, w: 145, n: 'FAILED', c: '10–100 Tsd.' }]" :key="tb.n" class="tbl" :class="{ failed: tb.n === 'FAILED' }">
          <rect :x="tb.x" y="128" :width="tb.w" height="122" rx="10" />
          <rect :x="tb.x" y="128" :width="tb.w" height="30" rx="10" class="head" />
          <text :x="tb.x + tb.w / 2" y="148" text-anchor="middle" class="th">{{ tb.n }}</text>
          <text :x="tb.x + tb.w / 2" y="240" text-anchor="middle" class="s">{{ tb.c }}</text>
        </g>
        <!-- static rows -->
        <g class="row"><rect x="30" y="166" width="130" height="16" rx="4" /><text x="40" y="178">Datei A</text></g>
        <g class="row"><rect x="30" y="184" width="130" height="16" rx="4" /><text x="40" y="196">Datei B</text></g>
        <g class="row"><rect x="220" y="164" width="130" height="16" rx="4" /><text x="230" y="176">Datei Y</text></g>
        <g class="row fail"><rect x="405" y="166" width="125" height="16" rx="4" /><text x="415" y="178">Datei X</text></g>

        <path d="M360 200 L393 200" class="edge red" marker-end="url(#sc-a-red)" />
        <!-- retry: a job from ACTIVE goes back to PENDING (orthogonal path below the tables) -->
        <path d="M240 252 V266 Q240 274 232 274 H123 Q115 274 115 266 V256" class="edge dashed" marker-end="url(#sc-a-new)" />
        <text x="177" y="292" text-anchor="middle" class="elabel">erneut einreihen</text>

        <path d="M285 252 L285 316" class="edge on" marker-end="url(#sc-a-new)" />
        <g class="done" :class="{ glow: doneGlow }">
          <rect x="200" y="318" width="170" height="40" rx="10" />
          <text x="285" y="343" text-anchor="middle">✓ DELETE + melden</text>
        </g>

        <!-- the moving job row -->
        <g v-if="row" class="row moving" :class="{ active: row.active }" :opacity="row.opacity" :transform="`translate(${row.x} ${row.y})`">
          <rect width="130" height="16" rx="4" />
          <text x="10" y="12">Datei C</text>
        </g>

        <circle v-for="d in newDots" :key="d.key" :cx="d.x" :cy="d.y" r="6" class="dot" />
      </svg>
      <div class="caption">
        <span class="num">{{ stepIdx + 1 }}</span>
        <Transition name="cap" mode="out-in"><p :key="stepIdx">{{ steps[stepIdx]?.new }}</p></Transition>
      </div>
    </section>

    <div class="facts">
      <div class="fact"><b>Skalierungstest</b><span>100 Laufwerke, 10 Mio. Dateien: 1 kHz Metadatenrate, 10× die Run-3-Spitze.</span></div>
      <div class="fact"><b>Repack in Produktion</b><span>CERN, 26.10.–5.11.2026: über 150 Bänder, bis 8 GB/s Lesen.</span></div>
      <div class="fact"><b>Noch offen</b><span>Nutzer-Workflows laufen am CERN noch auf dem Objectstore.</span></div>
    </div>
  </div>
</template>

<style scoped>
.cmp { height: 100%; display: grid; grid-template-columns: 1fr 44px 1fr; grid-template-rows: 1fr auto; column-gap: 6px; row-gap: 12px; }
.panel { display: flex; flex-direction: column; min-height: 0; padding: 12px 14px 10px; border-radius: var(--radius); border: 1px solid var(--line); background: var(--surface); box-shadow: var(--shadow); }
.panel.new { border-color: #d8cfff; }
.panel header { display: flex; align-items: center; gap: 10px; }
.panel header b { font-size: 17px; color: var(--ink); }
.ver { padding: 3px 9px; border-radius: 999px; font: 700 12px 'JetBrains Mono', monospace; color: var(--muted); background: var(--surface-2); }
.new .ver { color: #fff; background: var(--violet); }
.panel svg { flex: 1; min-height: 0; width: 100%; }
.arrow { display: flex; align-items: center; }
.arrow svg { width: 40px; height: 40px; }
.arrow path { fill: none; stroke: var(--violet); stroke-width: 3; stroke-linecap: round; stroke-linejoin: round; }

.t { font: 650 15px Inter, sans-serif; fill: var(--ink); }
.t.sm { font-size: 13px; }
.s { font: 500 12px 'JetBrains Mono', monospace; fill: var(--muted); }
.s.strong { font-weight: 700; fill: var(--ink-2); }
.pgl { fill: #3b23c9; }
.actor rect { fill: var(--surface-2); stroke: #d5dbe4; transition: fill 0.3s, stroke 0.3s; }
.actor.on rect { fill: #fff; stroke: #c9bdff; }
.old .actor.on rect { fill: #fff; stroke: var(--amber); stroke-width: 2; }
.actor.busy rect { animation: busy 0.8s ease-in-out infinite; }
.edge { fill: none; stroke: #b6c0ce; stroke-width: 2; }
.edge.on { stroke: #b9aaff; }
.edge.red { stroke: #f2a7a9; }
.edge.dashed { stroke: #b9aaff; stroke-dasharray: 5 5; }
.elabel { font: 600 11px 'JetBrains Mono', monospace; fill: var(--muted); }
.elabel.on { fill: var(--violet); }

/* v5: objects in the Ceph cluster */
.cluster rect { fill: #f3f5f8; stroke: #b6c0ce; stroke-width: 1.5; stroke-dasharray: 6 5; }
.obj rect { fill: #fff; stroke: #c3cbd7; stroke-width: 1.4; transition: stroke 0.3s, fill 0.3s; }
.ot { font: 700 12px 'JetBrains Mono', monospace; fill: var(--ink-2); }
.os { font: 500 10.5px 'JetBrains Mono', monospace; fill: var(--muted); }
.queue.locked > rect { stroke: var(--amber); stroke-width: 2.2; fill: var(--amber-soft); }
.ptr rect { fill: var(--surface-2); stroke: none; }
.ptr text { font: 600 10.5px 'JetBrains Mono', monospace; fill: var(--ink-2); }
.ptr.new rect { fill: #ffe7b0; }
.ptr.new { animation: pop 0.3s cubic-bezier(0.3, 1.8, 0.5, 1); transform-box: fill-box; transform-origin: center; }
.lock { opacity: 0; transition: opacity 0.2s; }
.lock.on { opacity: 1; }
.lock rect { fill: var(--amber); stroke: none; }
.lock path { fill: none; stroke: var(--amber); stroke-width: 2.2; }
.req.new rect { stroke: var(--amber); stroke-width: 2; fill: var(--amber-soft); }
.req.new { animation: pop 0.35s cubic-bezier(0.3, 1.8, 0.5, 1); transform-box: fill-box; transform-origin: center; }
.req.owned .os { fill: #9a6400; font-weight: 700; }
.old-dot { fill: var(--amber); }

/* v6: PostgreSQL tables */
.pg { fill: #f8f6ff; stroke: #d8cfff; stroke-width: 1.5; }
.tbl rect { fill: #fff; stroke: #b9aaff; stroke-width: 1.5; }
.tbl rect.head { fill: var(--violet); stroke: var(--violet); }
.th { font: 700 13px 'JetBrains Mono', monospace; fill: #fff; letter-spacing: 0.04em; }
.tbl.failed rect { stroke: #f2a7a9; }
.tbl.failed rect.head { fill: var(--rose); stroke: var(--rose); }
.row rect { fill: var(--violet-soft); stroke: none; }
.row text { font: 600 10.5px 'JetBrains Mono', monospace; fill: #3b23c9; }
.row.fail rect { fill: var(--rose-soft); }
.row.fail text { fill: var(--rose); }
.row.moving rect { fill: var(--violet); }
.row.moving text { fill: #fff; }
.row.moving.active rect { animation: blink 0.7s ease-in-out infinite; }
.done rect { fill: #e3f6ea; stroke: #9bd8b1; transition: fill 0.3s, stroke 0.3s; }
.done.glow rect { fill: #bfecd0; stroke: var(--green); stroke-width: 2; }
.done text { font: 600 13px Inter, sans-serif; fill: #0f7a3a; }
.dot { fill: var(--violet); filter: drop-shadow(0 1px 3px rgba(15, 23, 42, 0.3)); }

/* Step caption under each picture */
.caption { display: flex; align-items: flex-start; gap: 10px; min-height: 44px; padding: 8px 4px 0; border-top: 1px solid var(--line); }
.caption p { margin: 0; font-size: 13.5px; line-height: 1.4; color: var(--ink-2); opacity: 1; }
.num { flex: none; display: grid; place-items: center; width: 22px; height: 22px; border-radius: 50%; font: 700 12px 'JetBrains Mono', monospace; color: #fff; background: var(--amber); }
.new .num { background: var(--violet); }
.cap-enter-active, .cap-leave-active { transition: opacity 0.2s ease, transform 0.2s ease; }
.cap-enter-from { opacity: 0; transform: translateY(4px); }
.cap-leave-to { opacity: 0; transform: translateY(-4px); }

.facts { grid-column: 1 / -1; display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; }
.fact { display: flex; flex-direction: column; gap: 3px; padding: 10px 14px; border-radius: 12px; background: var(--violet-soft); box-shadow: inset 3px 0 0 var(--violet); }
.fact b { font-size: 14.5px; color: #3b23c9; }
.fact span { font-size: 13px; line-height: 1.35; color: var(--ink-2); }

@keyframes pop { from { transform: scale(0.4); } to { transform: scale(1); } }
@keyframes blink { 50% { opacity: 0.55; } }
@keyframes busy { 50% { stroke-opacity: 0.4; } }
</style>
