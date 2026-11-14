<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue'

// Current click on the slide: 0 = overview, 1 = Disk Buffer, 2 = Disk Buffer details
// (a panel that grows out of the Disk Buffer node), 3..7 = the other services.
// Each step plays a looping scene: packets travel along the edges and the
// nodes show what happens inside them (disks fill, jobs queue up, a tape mounts).
const props = defineProps<{ step: number }>()

// Click 2 opens the detail panel on top of the Disk Buffer scene; later clicks shift by one
const DETAIL_STEP = 2
const detail = computed(() => props.step === DETAIL_STEP)
const sceneStep = computed(() => props.step >= DETAIL_STEP ? props.step - 1 : props.step)

type Node = { id: string; x: number; y: number; w: number; h: number; label: string; sub?: string; kind?: 'actor' | 'hw'; top?: boolean }

const nodes: Node[] = [
  { id: 'users', x: 20, y: 12, w: 150, h: 52, label: 'Nutzer', sub: 'Experimente', kind: 'actor' },
  { id: 'admin', x: 290, y: 12, w: 200, h: 52, label: 'Admin-Client', sub: 'cta-admin', kind: 'actor' },
  { id: 'db', x: 20, y: 100, w: 150, h: 400, label: 'Disk Buffer', sub: 'EOS / dCache' },
  { id: 'fe', x: 290, y: 130, w: 200, h: 70, label: 'CTA Frontend', sub: 'XRootD-SSI · gRPC' },
  { id: 'cat', x: 230, y: 280, w: 150, h: 76, label: 'Catalogue', sub: 'Metadaten', top: true },
  { id: 'sch', x: 420, y: 280, w: 150, h: 76, label: 'Scheduler', sub: 'Queues', top: true },
  { id: 'mnt', x: 610, y: 280, w: 150, h: 76, label: 'Maintenance', sub: 'Daemon' },
  { id: 'td', x: 290, y: 430, w: 200, h: 70, label: 'Tape Daemon', sub: 'cta-taped' },
  { id: 'lib', x: 540, y: 372, w: 232, h: 140, label: 'Tape-Bibliothek', kind: 'hw', top: true },
]

type Edge = { id: string; from: string; to: string; d: string; both?: boolean; dashed?: boolean }

const edges: Edge[] = [
  { id: 'u-db', from: 'users', to: 'db', d: 'M95 64 L95 100' },
  { id: 'adm-fe', from: 'admin', to: 'fe', d: 'M390 64 L390 130' },
  { id: 'db-fe', from: 'db', to: 'fe', d: 'M158 167 L290 167', both: true },
  { id: 'fe-sch', from: 'fe', to: 'sch', d: 'M440 200 L495 280' },
  { id: 'fe-cat', from: 'fe', to: 'cat', d: 'M340 200 L305 280', both: true },
  { id: 'sch-td', from: 'sch', to: 'td', d: 'M495 356 L440 430' },
  { id: 'td-cat', from: 'td', to: 'cat', d: 'M340 430 L305 356', both: true },
  // Tape Daemon <-> FST: data only. It never reports to the MGM itself; it leaves finished
  // jobs in the scheduler's ToReport queue, and the Maintenance Daemon reports them to the MGM.
  { id: 'td-db', from: 'td', to: 'db', d: 'M290 465 L158 465', both: true },
  { id: 'td-lib', from: 'td', to: 'lib', d: 'M490 465 L554 465', both: true },
  { id: 'mnt-sch', from: 'mnt', to: 'sch', d: 'M610 318 L570 318', both: true },
  { id: 'mnt-db', from: 'mnt', to: 'db', d: 'M685 280 L685 122 Q685 110 673 110 L162 110 Q150 110 150 122 L150 146', dashed: true },
]

// Static tags on the two links to the Disk Buffer, so it is clear who talks about what
const edgeTags = [
  { edge: 'td-db', x: 224, y: 455, text: 'nur Daten (FST)' },
  { edge: 'mnt-db', x: 600, y: 101, text: 'nur Status (MGM)' },
]

type Kind = 'file' | 'event' | 'job' | 'meta' | 'report' | 'sweep'
type Packet = { e: string; at: number; dur: number; kind: Kind; rev?: boolean; label?: string }
type Chip = 'job' | 'done' | 'fail'
type Row = 'empty' | 'new' | 'ok' | 'hit'
type Fx = { disks: number; queue: Chip[]; rows: Row[]; feCheck: boolean; mounted: boolean; mount: number; writing: boolean }
type Scene = {
  id: string | null
  name: string
  tag: string
  lead: string
  cycle: number
  phases: { label: string; from: number; to: number }[]
  packets: Packet[]
  fx: (t: number) => Partial<Fx>
}

const count = (t: number, times: number[]) => times.filter(x => t >= x).length
const clamp = (v: number) => Math.min(1, Math.max(0, v))
const within = (t: number, a: number, b: number) => t >= a && t < b

const scenes: Scene[] = [
  {
    id: null,
    name: 'Der Weg aufs Band',
    tag: 'Überblick',
    lead: 'Eine Datei durchläuft drei Ebenen.',
    cycle: 7.2,
    phases: [
      { label: 'Ankommen im Disk Buffer', from: 0, to: 2.2 },
      { label: 'Planen: Job & Metadaten', from: 2.2, to: 4.3 },
      { label: 'Schreiben aufs Band', from: 4.3, to: 6.6 },
    ],
    packets: [
      { e: 'u-db', at: 0, dur: 1, kind: 'file', label: 'Datei' },
      { e: 'db-fe', at: 1.2, dur: 0.9, kind: 'event', label: 'Event' },
      { e: 'fe-sch', at: 2.3, dur: 0.8, kind: 'job', label: 'Job' },
      { e: 'fe-cat', at: 2.3, dur: 0.8, kind: 'meta' },
      { e: 'sch-td', at: 3.4, dur: 0.8, kind: 'job' },
      { e: 'td-db', rev: true, at: 4.4, dur: 0.9, kind: 'file', label: 'Daten' },
      { e: 'td-lib', at: 5.4, dur: 0.8, kind: 'file' },
    ],
    fx: t => ({
      disks: within(t, 1, 4.4) ? 1 : 0,
      queue: within(t, 3.1, 3.4) ? ['job'] : [],
      rows: t >= 3.1 ? ['ok', 'new', 'ok'] : ['ok', 'empty', 'ok'],
      mounted: t >= 4.4,
      mount: clamp((t - 3.4) / 1),
      writing: within(t, 6.2, 7.2),
    }),
  },
  {
    id: 'db',
    name: 'Disk Buffer',
    tag: 'EOS / dCache',
    lead: 'Das Zwischenlager vor und nach dem Band: MGM verwaltet, FST speichert.',
    cycle: 7,
    phases: [
      { label: 'Dateien landen auf den FSTs', from: 0, to: 1.9 },
      { label: 'MGM meldet dem Frontend: neue Datei', from: 1.9, to: 3.6 },
      { label: 'FST liefert die Daten an den Tape Daemon', from: 3.6, to: 6.2 },
    ],
    packets: [
      { e: 'u-db', at: 0, dur: 1, kind: 'file', label: 'Dateien' },
      { e: 'u-db', at: 0.35, dur: 1, kind: 'file' },
      { e: 'u-db', at: 0.7, dur: 1, kind: 'file' },
      { e: 'db-fe', at: 2, dur: 1, kind: 'event', label: 'Event' },
      { e: 'td-db', rev: true, at: 3.8, dur: 1, kind: 'file', label: 'Daten' },
      { e: 'td-db', rev: true, at: 4.1, dur: 1, kind: 'file' },
      { e: 'td-db', rev: true, at: 4.4, dur: 1, kind: 'file' },
      { e: 'td-lib', at: 5.2, dur: 0.8, kind: 'file' },
    ],
    fx: t => ({
      disks: count(t, [1, 1.35, 1.7]) - count(t, [3.8, 4.1, 4.4]),
      mounted: true,
      mount: 1,
      writing: within(t, 5.8, 7),
    }),
  },
  {
    id: 'fe',
    name: 'CTA Frontend',
    tag: 'Die Eingangstür',
    lead: 'Nimmt Anfragen an, prüft sie und verteilt die Arbeit.',
    cycle: 6,
    phases: [
      { label: 'Anfragen kommen an', from: 0, to: 1.4 },
      { label: 'Prüft die Berechtigung', from: 1.4, to: 2.3 },
      { label: 'Job einreihen & Metadaten speichern', from: 2.3, to: 3.8 },
    ],
    packets: [
      { e: 'db-fe', at: 0, dur: 1, kind: 'event', label: 'Event' },
      { e: 'adm-fe', at: 0.4, dur: 0.9, kind: 'event', label: 'Befehl' },
      { e: 'fe-sch', at: 2.4, dur: 0.9, kind: 'job', label: 'Job' },
      { e: 'fe-cat', at: 2.6, dur: 0.9, kind: 'meta', label: 'Metadaten' },
    ],
    fx: t => ({
      feCheck: within(t, 1.4, 2.6),
      queue: t >= 3.3 ? ['job'] : [],
      rows: t >= 3.5 ? ['ok', 'new', 'ok'] : ['ok', 'empty', 'ok'],
    }),
  },
  {
    id: 'sch',
    name: 'Scheduler',
    tag: 'Die Warteschlange',
    lead: 'Jobs warten, bis ein passendes Band bereit ist.',
    cycle: 7.5,
    phases: [
      { label: 'Jobs werden eingereiht (Push)', from: 0, to: 2.2 },
      { label: 'Warten auf ein Band', from: 2.2, to: 3.6 },
      { label: 'Band bereit: Jobs abholen (Pull)', from: 3.6, to: 6 },
    ],
    packets: [
      { e: 'fe-sch', at: 0, dur: 0.8, kind: 'job', label: 'Job' },
      { e: 'fe-sch', at: 0.6, dur: 0.8, kind: 'job' },
      { e: 'fe-sch', at: 1.2, dur: 0.8, kind: 'job' },
      { e: 'sch-td', at: 4, dur: 0.8, kind: 'job', label: 'Pull' },
      { e: 'sch-td', at: 4.5, dur: 0.8, kind: 'job' },
      { e: 'sch-td', at: 5, dur: 0.8, kind: 'job' },
    ],
    fx: t => ({
      queue: Array(count(t, [0.8, 1.4, 2]) - count(t, [4, 4.5, 5])).fill('job'),
      mounted: t >= 3.4,
      mount: clamp((t - 2.4) / 1),
    }),
  },
  {
    id: 'cat',
    name: 'Catalogue',
    tag: 'Das Gedächtnis',
    lead: 'Weiß, welche Datei auf welchem Band liegt.',
    cycle: 7,
    phases: [
      { label: 'Neue Datei eintragen', from: 0, to: 1.8 },
      { label: 'Status: liegt jetzt auf Band', from: 1.8, to: 3.4 },
      { label: 'Auskunft: wo liegt Datei X?', from: 3.4, to: 6 },
    ],
    packets: [
      { e: 'fe-cat', at: 0, dur: 1, kind: 'meta', label: 'Neue Datei' },
      { e: 'td-cat', at: 2.2, dur: 1, kind: 'report', label: 'Auf Band ✓' },
      { e: 'fe-cat', at: 3.6, dur: 0.9, kind: 'meta', label: 'Wo liegt X?' },
      { e: 'fe-cat', rev: true, at: 4.8, dur: 0.9, kind: 'meta', label: 'Band V042' },
    ],
    fx: t => ({
      rows: [
        t >= 4.5 && t < 5.7 ? 'hit' : 'ok',
        t < 1 ? 'empty' : t < 3.2 ? 'new' : 'ok',
        'ok',
      ],
    }),
  },
  {
    id: 'td',
    name: 'Tape Daemon',
    tag: 'Ein Prozess pro Laufwerk',
    lead: 'Der Arbeiter am Laufwerk. Mit dem Disk Buffer tauscht er nur Daten aus.',
    cycle: 8,
    phases: [
      { label: 'Job holen', from: 0, to: 0.9 },
      { label: 'Band mounten', from: 0.9, to: 2.1 },
      { label: 'Daten vom FST aufs Band schreiben', from: 2.1, to: 4.6 },
      { label: 'In Catalogue eintragen, Job als erledigt ablegen', from: 4.6, to: 6.6 },
    ],
    packets: [
      { e: 'sch-td', at: 0, dur: 0.8, kind: 'job', label: 'Job' },
      { e: 'td-db', rev: true, at: 2.2, dur: 0.9, kind: 'file', label: 'Daten' },
      { e: 'td-db', rev: true, at: 2.5, dur: 0.9, kind: 'file' },
      { e: 'td-lib', at: 3.3, dur: 0.8, kind: 'file', label: 'SCSI' },
      { e: 'td-lib', at: 3.6, dur: 0.8, kind: 'file' },
      { e: 'td-cat', at: 4.8, dur: 0.9, kind: 'report', label: 'Auf Band ✓' },
      { e: 'sch-td', rev: true, at: 5.4, dur: 0.9, kind: 'report', label: 'Erledigt' },
    ],
    fx: t => ({
      queue: t < 0.1 ? ['job'] : t >= 6.3 ? ['done'] : [],
      disks: 2,
      mounted: t >= 2,
      mount: clamp((t - 1) / 1),
      writing: within(t, 4.1, 5.2),
    }),
  },
  {
    id: 'mnt',
    name: 'Maintenance',
    tag: 'Der Hausmeister',
    lead: 'Räumt auf und meldet dem MGM, was fertig ist. Der Tape Daemon muss darauf nicht warten.',
    cycle: 7.5,
    phases: [
      { label: 'Hängende Jobs aufräumen', from: 0, to: 2 },
      { label: 'Erledigte Jobs abholen', from: 2, to: 3.6 },
      { label: 'Erfolg oder Fehler ans MGM melden', from: 3.6, to: 6 },
    ],
    packets: [
      { e: 'mnt-sch', at: 0.4, dur: 0.6, kind: 'sweep', label: 'Aufräumen' },
      { e: 'mnt-sch', rev: true, at: 2.4, dur: 0.7, kind: 'report', label: 'Erledigt' },
      { e: 'mnt-db', at: 3.8, dur: 1.8, kind: 'report', label: 'Auf Band ✓ / Fehler' },
    ],
    fx: t => ({
      queue: (['done', 'done', 'fail'] as Chip[]).slice(count(t, [2.4, 2.55, 2.7])),
    }),
  },
]

const DEFAULT_FX: Fx = { disks: 0, queue: [], rows: ['ok', 'empty', 'ok'], feCheck: false, mounted: false, mount: 0, writing: false }

// ---------- Scene clock ----------
const t = ref(0)
let start = performance.now()
let raf = 0
const scene = computed(() => scenes[Math.min(Math.max(sceneStep.value, 0), scenes.length - 1)])

function tick(now: number) {
  t.value = ((now - start) / 1000) % scene.value.cycle
  raf = requestAnimationFrame(tick)
}
watch(sceneStep, () => { start = performance.now() })
onMounted(() => { raf = requestAnimationFrame(tick) })
onBeforeUnmount(() => cancelAnimationFrame(raf))

const fx = computed<Fx>(() => ({ ...DEFAULT_FX, ...scene.value.fx(t.value) }))

const activePhase = computed(() => scene.value.phases.findIndex(p => t.value >= p.from && t.value < p.to))
const phaseProgress = (i: number) => {
  const p = scene.value.phases[i]
  return i < activePhase.value ? 1 : i === activePhase.value ? clamp((t.value - p.from) / (p.to - p.from)) : 0
}

// ---------- Packets moving along the edges ----------
const pathEls = ref<Record<string, SVGPathElement>>({})
const ease = (x: number) => x * x * (3 - 2 * x)

const packets = computed(() => scene.value.packets.flatMap((p) => {
  const local = (t.value - p.at) / p.dur
  const el = pathEls.value[p.e]
  if (local < 0 || local > 1 || !el)
    return []
  const f = ease(p.rev ? 1 - local : local)
  const pt = el.getPointAtLength(f * el.getTotalLength())
  return [{ ...p, x: pt.x, y: pt.y, opacity: Math.min(1, local * 8, (1 - local) * 8) }]
}))

const activeEdges = computed(() => new Set(scene.value.packets.map(p => p.e)))
const involved = computed(() => {
  const ids = new Set<string>()
  if (scene.value.id)
    ids.add(scene.value.id)
  for (const e of edges) {
    if (activeEdges.value.has(e.id)) {
      ids.add(e.from)
      ids.add(e.to)
    }
  }
  return ids
})
const focused = computed(() => scene.value.id !== null)

// ---------- Tape library: robot carries the cartridge from its slot into the drive ----------
const SLOT = { x: 652, y: 462 }
const DRIVE = { x: 566, y: 458 }
const TRAVEL_Y = 418
const lerp = (a: number, b: number, f: number) => a + (b - a) * ease(clamp(f))
const libSlots = [0, 1, 2, 3, 4].flatMap(c => [0, 1, 2].map(r => ({
  key: `${c}-${r}`,
  x: 652 + c * 22,
  y: 440 + r * 22,
  label: ['#c9bfff', '#b8e9df', '#cfdcff', '#ffe2a8'][(c + r * 2) % 4],
}))).filter(s => s.key !== '0-1' && s.key !== '4-2') // 0-1: the moving cartridge, 4-2: empty slot
const cartPos = computed(() => {
  const m = fx.value.mount
  if (m < 0.25)
    return { x: SLOT.x, y: lerp(SLOT.y, TRAVEL_Y, m / 0.25) } // lift out of the slot
  if (m < 0.75)
    return { x: lerp(SLOT.x, DRIVE.x, (m - 0.25) / 0.5), y: TRAVEL_Y } // ride along the rail
  return { x: DRIVE.x, y: lerp(TRAVEL_Y, DRIVE.y, (m - 0.75) / 0.25) } // insert into the drive
})
const labelWidth = (s: string) => s.length * 6.4 + 16
</script>

<template>
  <SlideFrame :section="1" title="Die Bausteine von CTA" kicker="CTA heute · Architektur">
    <div class="arch-body">
      <svg viewBox="0 0 780 520" class="diagram">
        <defs>
          <marker id="arrow-dim" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
            <path d="M0 0 L10 5 L0 10 z" fill="#94a3b8" />
          </marker>
          <marker id="arrow-on" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
            <path d="M0 0 L10 5 L0 10 z" fill="#2f6bff" />
          </marker>
          <filter id="glow" x="-30%" y="-30%" width="160%" height="160%">
            <feGaussianBlur stdDeviation="6" result="b" />
            <feMerge><feMergeNode in="b" /><feMergeNode in="SourceGraphic" /></feMerge>
          </filter>
        </defs>

        <!-- Edges -->
        <g v-for="e in edges" :key="e.id" class="edge" :class="{ on: activeEdges.has(e.id), off: focused && !activeEdges.has(e.id) }">
          <path
            :ref="(el) => { if (el) pathEls[e.id] = el as SVGPathElement }"
            :d="e.d" class="edge-base"
            :marker-end="`url(#${activeEdges.has(e.id) ? 'arrow-on' : 'arrow-dim'})`"
            :marker-start="e.both ? `url(#${activeEdges.has(e.id) ? 'arrow-on' : 'arrow-dim'})` : undefined"
            :stroke-dasharray="e.dashed ? '5 6' : undefined"
          />
        </g>

        <!-- Who talks to which part of the Disk Buffer -->
        <text v-for="tg in edgeTags" :key="tg.edge" :x="tg.x" :y="tg.y" text-anchor="middle" class="edge-tag"
              :class="{ on: activeEdges.has(tg.edge), off: focused && !activeEdges.has(tg.edge) }">{{ tg.text }}</text>

        <!-- Nodes -->
        <g v-for="n in nodes" :key="n.id" class="node"
           :class="[n.kind, { active: n.id === scene.id, dim: focused && !involved.has(n.id) }]">
          <rect :x="n.x" :y="n.y" :width="n.w" :height="n.h" rx="14" class="node-box"
                :filter="n.id === scene.id ? 'url(#glow)' : undefined" />
          <template v-if="n.id === 'db'">
            <text :x="n.x + n.w / 2" :y="n.y + 22" text-anchor="middle" class="node-label">{{ n.label }}</text>
            <text :x="n.x + n.w / 2" :y="n.y + 37" text-anchor="middle" class="node-sub">{{ n.sub }}</text>
            <!-- EOS internals: MGM (namespace, talks to the frontend) and FST (stores the data) -->
            <rect x="32" y="146" width="126" height="44" rx="9" class="sub-box" />
            <text x="95" y="165" text-anchor="middle" class="sub-label">MGM</text>
            <text x="95" y="180" text-anchor="middle" class="node-sub">Namespace</text>
            <line x1="95" y1="190" x2="95" y2="208" class="sub-link" />
            <rect x="32" y="208" width="126" height="282" rx="9" class="sub-box" />
            <text x="95" y="229" text-anchor="middle" class="sub-label">FST</text>
            <text x="95" y="244" text-anchor="middle" class="node-sub">Daten</text>
          </template>
          <template v-else-if="n.top">
            <text :x="n.x + n.w / 2" :y="n.y + 26" text-anchor="middle" class="node-label">{{ n.label }}</text>
            <text v-if="n.sub" :x="n.x + n.w / 2" :y="n.y + 42" text-anchor="middle" class="node-sub">{{ n.sub }}</text>
          </template>
          <template v-else>
            <text :x="n.x + n.w / 2" :y="n.y + n.h / 2 - 2" text-anchor="middle" class="node-label">{{ n.label }}</text>
            <text :x="n.x + n.w / 2" :y="n.y + n.h / 2 + 18" text-anchor="middle" class="node-sub">{{ n.sub }}</text>
          </template>
        </g>

        <!-- Disk Buffer: disks fill up as files arrive -->
        <g v-for="k in 3" :key="`disk${k}`" :transform="`translate(65 ${276 + (k - 1) * 64})`" class="disk"
           :class="{ full: fx.disks >= 4 - k, dim: focused && !involved.has('db') }">
          <path d="M0 0 V34 A30 9 0 0 0 60 34 V0" />
          <ellipse cx="30" cy="0" rx="30" ry="9" />
        </g>

        <!-- Frontend: request check -->
        <g class="check" :class="{ show: fx.feCheck }" transform="translate(476 144)">
          <circle r="12" />
          <path d="M-5 0 L-1.5 4 L5.5 -4" />
        </g>

        <!-- Catalogue: table rows -->
        <g :class="{ dim: focused && !involved.has('cat') }">
          <rect v-for="(r, i) in fx.rows" :key="`row${i}`" x="250" :y="330 + i * 7" width="110" height="4" rx="2" class="row" :class="r" />
        </g>

        <!-- Scheduler: queue of jobs -->
        <g :class="{ dim: focused && !involved.has('sch') }">
          <rect x="434" y="328" width="122" height="18" rx="5" class="queue-slot" />
          <rect v-for="(c, i) in fx.queue" :key="i" :x="438 + i * 20" y="331" width="16" height="12" rx="3" class="chip" :class="c" />
        </g>

        <!-- Tape Daemon: mounted tape indicator -->
        <circle cx="466" cy="465" r="5" class="mounted" :class="{ on: fx.mounted, dim: focused && !involved.has('td') }" />

        <!-- Library: robot lifts a cartridge from its slot, carries it along the rail into the drive; reels spin while writing -->
        <g class="lib" :class="{ dim: focused && !involved.has('lib') }">
          <!-- Storage slots -->
          <rect x="646" y="434" width="118" height="70" rx="6" class="lib-rack" />
          <rect x="652" y="462" width="18" height="14" rx="2.5" class="slot-empty" />
          <rect x="740" y="484" width="18" height="14" rx="2.5" class="slot-empty" />
          <g v-for="s in libSlots" :key="s.key">
            <rect :x="s.x" :y="s.y" width="18" height="14" rx="2.5" class="lib-cart" />
            <rect :x="s.x + 3" :y="s.y + 4" width="12" height="6" rx="1.5" :fill="s.label" />
          </g>

          <!-- Drive with reel window -->
          <rect x="554" y="441" width="86" height="48" rx="7" class="drive" :class="{ busy: fx.writing }" />
          <rect x="560" y="452" width="30" height="26" rx="4" class="drive-mouth" />
          <rect x="596" y="448" width="38" height="22" rx="4" class="reel-window" />
          <g v-for="cx in [606, 624]" :key="cx" :transform="`translate(${cx} 459)`">
            <g class="reel" :class="{ spin: fx.writing }">
              <circle r="7" class="reel-disc" :class="{ loaded: fx.mounted }" />
              <path d="M0 -7 V7 M-6 -3.5 L6 3.5 M-6 3.5 L6 -3.5" class="reel-spokes" />
            </g>
          </g>
          <line x1="606" y1="466" x2="624" y2="466" class="tape" :class="{ loaded: fx.mounted }" />
          <circle cx="602" cy="481" r="3" class="drive-led" :class="{ on: fx.mounted, busy: fx.writing }" />
          <rect x="610" y="479.5" width="24" height="3" rx="1.5" class="drive-act" :class="{ on: fx.writing }" />

          <!-- Robot: rail, carriage, gripper with cartridge -->
          <line x1="552" y1="410" x2="764" y2="410" class="rail" />
          <rect :x="cartPos.x - 3" y="405" width="24" height="10" rx="3" class="carriage" />
          <line :x1="cartPos.x + 9" y1="414" :x2="cartPos.x + 9" :y2="cartPos.y - 3" class="cable" />
          <g :transform="`translate(${cartPos.x} ${cartPos.y})`">
            <rect width="18" height="14" rx="2.5" class="lib-cart moving" />
            <rect x="3" y="4" width="12" height="6" rx="1.5" fill="#ffc94d" />
            <path d="M-3 4 V-3 H21 V4" class="gripper" />
          </g>
        </g>

        <!-- Travelling packets -->
        <g v-for="(p, i) in packets" :key="`p${i}`" :transform="`translate(${p.x} ${p.y})`" :opacity="p.opacity" class="packet" :class="p.kind">
          <rect v-if="p.kind === 'file'" x="-8" y="-6" width="16" height="12" rx="2.5" />
          <rect v-else-if="p.kind === 'job'" x="-7" y="-7" width="14" height="14" rx="3.5" />
          <rect v-else-if="p.kind === 'meta'" x="-6" y="-6" width="12" height="12" rx="2" transform="rotate(45)" />
          <circle v-else r="7" />
          <g v-if="p.label" transform="translate(0 -20)">
            <rect :x="-labelWidth(p.label) / 2" y="-10" :width="labelWidth(p.label)" height="20" rx="10" class="plabel-bg" />
            <text text-anchor="middle" dy="4" class="plabel">{{ p.label }}</text>
          </g>
        </g>
      </svg>

      <aside class="panel card">
        <Transition name="swap" mode="out-in">
          <div :key="scene.name" class="panel-inner">
            <div class="panel-step">{{ sceneStep > 0 ? `${String(sceneStep).padStart(2, '0')} / 06` : 'Überblick' }}</div>
            <h3>{{ scene.name }}</h3>
            <div class="panel-tag">{{ scene.tag }}</div>
            <p class="lead">{{ scene.lead }}</p>
            <ol class="phases">
              <li v-for="(ph, i) in scene.phases" :key="ph.label" :class="{ on: i === activePhase, done: activePhase === -1 || i < activePhase }">
                <span class="num">{{ i + 1 }}</span>
                <span class="ph-label">{{ ph.label }}</span>
                <i class="bar" :style="{ transform: `scaleX(${activePhase === -1 ? 1 : phaseProgress(i)})` }" />
              </li>
            </ol>
          </div>
        </Transition>
        <div class="dots">
          <span v-for="k in 6" :key="k" :class="{ on: k === sceneStep }" />
        </div>
      </aside>

      <!-- Detail panel: grows out of the Disk Buffer node and collapses back into it -->
      <Transition name="zoom">
        <div v-if="detail" class="detail-panel">
          <DiskBufferDetail />
        </div>
      </Transition>
    </div>
  </SlideFrame>
</template>

<style scoped>
.arch-body {
  position: relative;
  height: 100%;
  display: grid;
  grid-template-columns: 1fr 330px;
  gap: 28px;
  align-items: center;
}
.diagram { width: 100%; height: 100%; max-height: 500px; }

.dim { opacity: 0.22; transition: opacity 0.4s; }

/* Edges */
.edge { transition: opacity 0.4s; }
.edge-base { fill: none; stroke: #c3cad6; stroke-width: 1.6; transition: stroke 0.4s; }
.edge.on .edge-base { stroke: var(--accent); stroke-opacity: 0.55; }
.edge.off { opacity: 0.2; }
.edge-tag { font: 600 10.5px 'JetBrains Mono', monospace; fill: #94a3b8; transition: fill 0.4s, opacity 0.4s; }
.edge-tag.on { fill: var(--accent); }
.edge-tag.off { opacity: 0.25; }

/* Nodes */
.node { transition: opacity 0.4s; }
.node.dim { opacity: 0.22; }
.node-box { fill: #ffffff; stroke: #d6dde8; stroke-width: 1.4; transition: stroke 0.4s, fill 0.4s; }
.node.actor .node-box { fill: transparent; stroke-dasharray: 4 5; stroke: #b3bccb; }
.node.hw .node-box { fill: #f4f1ff; stroke: #c9bfff; }
.node.active .node-box { fill: #f2f6ff; stroke: var(--accent); stroke-width: 2.2; }
.node-label { font: 650 16px 'Inter', sans-serif; fill: var(--ink); }
.node-sub { font: 500 11px 'JetBrains Mono', monospace; fill: var(--muted); }
.node.active .node-sub { fill: var(--accent); }
.sub-box { fill: #f7f9fc; stroke: #dfe4ec; stroke-width: 1.2; }
.node.active .sub-box { fill: #ffffff; stroke: #c3d4ff; }
.sub-label { font: 650 13px 'Inter', sans-serif; fill: var(--ink-2); }
.sub-link { stroke: #c3cad6; stroke-width: 1.4; stroke-dasharray: 3 3; }

/* Node internals */
.disk path, .disk ellipse { fill: #f1f4f9; stroke: #c3cad6; stroke-width: 1.2; transition: fill 0.35s, stroke 0.35s; }
.disk.full path { fill: #d9f3ee; stroke: var(--teal); }
.disk.full ellipse { fill: #b8e9df; stroke: var(--teal); }

.check { opacity: 0; transform-box: fill-box; transform-origin: center; transition: opacity 0.25s; }
.check.show { opacity: 1; animation: pop 0.35s cubic-bezier(0.3, 1.8, 0.5, 1); }
.check circle { fill: #dcfce7; stroke: var(--green); stroke-width: 1.5; }
.check path { fill: none; stroke: var(--green); stroke-width: 2.5; stroke-linecap: round; stroke-linejoin: round; }

.row { fill: #eef1f5; transition: fill 0.4s; }
.row.ok { fill: #cfd6e2; }
.row.new { fill: var(--accent); }
.row.hit { fill: var(--amber); }

.queue-slot { fill: #f1f4f9; stroke: #dfe4ec; }
.chip { transition: fill 0.3s; transform-box: fill-box; transform-origin: center; animation: pop 0.3s cubic-bezier(0.3, 1.8, 0.5, 1); }
.chip.job { fill: var(--violet); }
.chip.done { fill: var(--green); }
.chip.fail { fill: var(--rose); }

.mounted { fill: #cfd6e2; transition: fill 0.3s; }
.mounted.on { fill: var(--teal); filter: drop-shadow(0 0 4px rgba(0, 163, 137, 0.6)); }

.lib-rack { fill: #ebe6ff; stroke: #d9d0ff; }
.slot-empty { fill: none; stroke: #c9bfff; stroke-dasharray: 2 2; }
.lib-cart { fill: #2a2541; stroke: #4b4370; }
.lib-cart.moving { stroke: var(--accent); stroke-width: 1.4; }
.drive { fill: #1e1b33; stroke: #c9bfff; stroke-width: 1.4; transition: stroke 0.3s; }
.drive.busy { stroke: var(--violet); stroke-width: 2; }
.drive-mouth { fill: #0f0c1f; }
.reel-window { fill: #120f24; stroke: #3b3360; }
.reel-disc { fill: #3b3360; transition: fill 0.4s; }
.reel-disc.loaded { fill: #7a5c41; }
.reel-spokes { stroke: #c9bfff; stroke-width: 1.2; stroke-linecap: round; }
.reel.spin { animation: spin 0.7s linear infinite; }
.tape { stroke: #7a5c41; stroke-width: 2; opacity: 0; transition: opacity 0.4s; }
.tape.loaded { opacity: 1; }
.drive-led { fill: #4b4370; transition: fill 0.3s; }
.drive-led.on { fill: var(--teal); }
.drive-led.busy { fill: var(--violet); animation: blink 0.5s ease-in-out infinite; }
.drive-act { fill: #3b3360; }
.drive-act.on { fill: var(--violet); animation: blink 0.35s ease-in-out infinite; }
.rail { stroke: #b3a6ff; stroke-width: 3; stroke-linecap: round; }
.carriage { fill: var(--violet); }
.cable { stroke: var(--violet); stroke-width: 1.5; }
.gripper { fill: none; stroke: var(--accent); stroke-width: 2; stroke-linecap: round; stroke-linejoin: round; }

/* Packets */
.packet { filter: drop-shadow(0 2px 4px rgba(15, 23, 42, 0.25)); }
.packet.file { color: var(--teal); fill: var(--teal); }
.packet.event { color: var(--amber); fill: var(--amber); }
.packet.job { color: var(--violet); fill: var(--violet); }
.packet.meta { color: var(--accent); fill: var(--accent); }
.packet.report { color: var(--green); fill: var(--green); }
.packet.sweep { color: #db2777; fill: #db2777; }
.plabel-bg { fill: #0b1220; stroke: none; }
.plabel { font: 600 11px 'Inter', sans-serif; fill: #ffffff; }

/* Panel */
.panel { padding: 24px 24px 20px; }
.panel-step { font: 600 12px 'JetBrains Mono', monospace; color: var(--muted); letter-spacing: 0.1em; text-transform: uppercase; }
.panel h3 { margin: 8px 0 0; font-size: 28px; font-weight: 750; letter-spacing: -0.02em; color: var(--ink); }
.panel-tag {
  display: inline-block; margin-top: 8px; padding: 4px 10px; border-radius: 999px;
  font: 500 12px 'JetBrains Mono', monospace; color: var(--accent); background: var(--accent-soft);
}
.lead { margin: 14px 0 0; font-size: 16px; line-height: 1.45; color: var(--ink-2); opacity: 1; }

.phases { list-style: none; margin: 18px 0 0; padding: 0; display: grid; gap: 8px; }
.phases li {
  position: relative;
  display: flex; align-items: center; gap: 12px;
  padding: 10px 12px;
  border-radius: 12px;
  overflow: hidden;
  font-size: 14.5px;
  color: var(--muted);
  background: var(--surface-2);
  transition: color 0.3s, background 0.3s;
}
.phases li.done { color: var(--ink-2); }
.phases li.on { color: var(--ink); background: var(--accent-soft); font-weight: 600; }
.num {
  flex: none; display: grid; place-items: center;
  width: 24px; height: 24px; border-radius: 50%;
  font: 600 12px 'JetBrains Mono', monospace;
  background: #ffffff; color: var(--muted);
  transition: background 0.3s, color 0.3s;
}
.phases li.on .num { background: var(--accent); color: #fff; }
.bar {
  position: absolute; left: 0; bottom: 0; height: 3px; width: 100%;
  background: var(--accent);
  transform-origin: left;
}
.phases li:not(.on) .bar { opacity: 0.12; }

.dots { display: flex; gap: 6px; margin-top: 18px; }
.dots span { width: 20px; height: 4px; border-radius: 2px; background: var(--line); transition: background 0.3s, width 0.3s; }
.dots span.on { width: 34px; background: var(--accent); }

/* Detail panel over the whole diagram; the transform origin sits on the Disk Buffer node */
.detail-panel {
  position: absolute;
  inset: -6px -14px;
  z-index: 5;
  padding: 16px 20px 18px;
  border: 2px solid var(--accent);
  border-radius: 22px;
  background: var(--bg);
  box-shadow: 0 0 0 6px rgba(47, 107, 255, 0.12), 0 24px 60px rgba(15, 23, 42, 0.22);
  transform-origin: 11% 58%;
}
.zoom-enter-active { transition: transform 0.6s cubic-bezier(0.2, 0.9, 0.25, 1), opacity 0.35s ease; }
.zoom-leave-active { transition: transform 0.45s cubic-bezier(0.5, 0, 0.75, 0.4), opacity 0.2s ease 0.25s; }
.zoom-enter-from, .zoom-leave-to { opacity: 0; transform: scale(0.1); }

.swap-enter-active, .swap-leave-active { transition: opacity 0.25s ease, transform 0.25s ease; }
.swap-enter-from { opacity: 0; transform: translateY(8px); }
.swap-leave-to { opacity: 0; transform: translateY(-6px); }

@keyframes spin { to { transform: rotate(360deg); } }
@keyframes blink { 50% { opacity: 0.3; } }
@keyframes pop { from { transform: scale(0.3); } to { transform: scale(1); } }
</style>
