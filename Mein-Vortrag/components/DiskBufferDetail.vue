<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

// EOSCTA tape buffer at CERN, from J. Leduc's CTA service talk (IN2P3 tape workshop, 10 June 2026)
const stats = [
  { value: '≈ 2', unit: 'PB', label: 'SSD-Buffer gesamt, alle Instanzen', sub: '64 Server × 16 SSDs × 2 TB' },
  { value: '≈ 320', unit: 'TB', label: 'davon pro LHC-Instanz', sub: '10 Server × 16 SSDs × 2 TB, zugesagt: 10 GB/s' },
  { value: '8', unit: 'h', label: 'bis eine Instanz voll ist', sub: '10 GB/s × 8 h ≈ 290 TB, wenn das Band steht' },
]

const facts = [
  { title: 'Nur eine Kopie', text: 'Keine Redundanz auf den SSDs. Geht eine Datei verloren, kommt sie neu vom EOS.' },
  { title: 'Sofort wieder frei', text: 'Eine Datei wird gelöscht, sobald sie sicher auf Band liegt.' },
  { title: 'Nur Durchgangslager', text: 'Experimente können hier nichts dauerhaft speichern.' },
]

// Outage story, looping: data flows through normally, then the CTA instance breaks,
// the buffer's 10 servers fill up one after another (8 h), and finally the data
// backs up at the experiment.
const HOURS = 8
const SERVERS = 10
const CYCLE = 10.5
const BREAK_AT = 1.6
const FILL_FROM = 2.1
const FILL_TO = 7.1

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
const broken = computed(() => t.value >= BREAK_AT)
const hit = computed(() => t.value >= BREAK_AT && t.value < BREAK_AT + 0.5)
const fill = computed(() => clamp((t.value - FILL_FROM) / (FILL_TO - FILL_FROM)))
const full = computed(() => fill.value >= 1)
const hours = computed(() => Math.floor(fill.value * HOURS))
const serverFill = (i: number) => clamp(fill.value * SERVERS - i)

// Little data packets travelling along the arrows
const SPAWN = 0.32
const TRAVEL = 0.7
function packetsOn(active: (s: number) => boolean) {
  const out: { key: number; left: number; opacity: number }[] = []
  for (let k = 0; k * SPAWN < CYCLE; k++) {
    const s = k * SPAWN
    const local = (t.value - s) / TRAVEL
    if (local < 0 || local > 1 || !active(s))
      continue
    out.push({ key: k, left: local * 100, opacity: Math.min(1, local * 6, (1 - local) * 6) })
  }
  return out
}
// Into the buffer: always, but once it is full the packets stall halfway (jam)
const inPackets = computed(() => packetsOn(() => true).map(p => full.value ? { ...p, left: Math.min(p.left, 45) } : p))
// Out to tape: only while the CTA instance is running
const outPackets = computed(() => packetsOn(s => s + TRAVEL < BREAK_AT && t.value < BREAK_AT))
const queued = computed(() => full.value ? Math.min(5, Math.floor((t.value - FILL_TO) / 0.45) + 1) : 0)
</script>

<template>
  <div class="detail">
    <header class="d-head">
      <span class="d-tag mono">Disk Buffer · am CERN: EOSCTA</span>
      <h3>Acht Stunden Luft, wenn das Band steht</h3>
      <span class="d-src">Quelle: J. Leduc, „CERN CTA Service“, IN2P3 Tape Workshop, 10.06.2026, Folien 19–22</span>
    </header>
    <div class="layout">
      <div class="stats">
        <div v-for="(s, i) in stats" :key="s.label" class="card stat" v-motion :initial="{ opacity: 0, x: -20 }" :enter="{ opacity: 1, x: 0, transition: { delay: 100 + i * 150, duration: 500 } }">
          <div class="value"><span>{{ s.value }}</span><small>{{ s.unit }}</small></div>
          <div>
            <div class="label">{{ s.label }}</div>
            <div class="sub">{{ s.sub }}</div>
          </div>
        </div>
      </div>

      <div class="right">
        <div class="card outage">
          <div class="o-head">
            <span class="o-title">Was passiert, wenn das Band steht?</span>
            <span class="mono legend">eine LHC-Instanz</span>
          </div>

          <div class="flow">
            <!-- Experiment: stylised detector cross-section (not an official logo) -->
            <div class="node exp" :class="{ jam: full }">
              <svg viewBox="-20 -20 40 40" class="detector" :class="{ busy: !full }">
                <circle r="18" class="ring outer" />
                <circle r="12.5" class="ring" />
                <circle r="6" class="core" />
                <path v-for="a in 8" :key="a" :transform="`rotate(${a * 45})`" d="M0 -7.5 V-17" class="spoke" />
              </svg>
              <div>
                <span class="mono eg">z. B.</span>
                <b>ATLAS</b>
                <span>EOS, Festplatten</span>
              </div>
              <div class="pile">
                <i v-for="q in queued" :key="q" />
              </div>
            </div>

            <div class="arrow" :class="{ jam: full }">
              <span class="mono">{{ full ? 'Stau' : '10 GB/s' }}</span>
              <div class="track">
                <i class="line" />
                <b v-for="p in inPackets" :key="p.key" class="pkt" :style="{ left: `${p.left}%`, opacity: p.opacity }" />
              </div>
            </div>

            <!-- Buffer: 10 SSD servers filling up one after another -->
            <div class="node buffer" :class="{ full }">
              <div class="b-head">
                <b>EOSCTA-Buffer</b>
                <em class="mono">{{ full ? 'voll nach 8 h' : broken ? `${hours} h` : 'läuft durch' }}</em>
              </div>
              <div class="servers">
                <div v-for="i in SERVERS" :key="i" class="srv">
                  <div class="srv-fill" :style="{ transform: `scaleY(${serverFill(i - 1)})` }" />
                  <i class="srv-led" :class="{ on: serverFill(i - 1) > 0 && serverFill(i - 1) < 1 }" />
                </div>
              </div>
              <span>10 Server · ≈ 320 TB SSDs</span>
            </div>

            <div class="arrow out" :class="{ blocked: broken }">
              <span class="mono">{{ broken ? 'steht' : '10 GB/s' }}</span>
              <div class="track">
                <i class="line" />
                <b v-for="p in outPackets" :key="p.key" class="pkt" :style="{ left: `${p.left}%`, opacity: p.opacity }" />
                <svg v-if="broken" viewBox="-8 -8 16 16" class="cut"><path d="M-5 -5 L5 5 M5 -5 L-5 5" /></svg>
              </div>
            </div>

            <!-- CTA instance: drive with spinning reels, stops and alarms when it breaks -->
            <div class="node cta" :class="{ broken, hit }">
              <span class="alarm mono">⚠ Störung</span>
              <div class="drive">
                <div v-for="r in 2" :key="r" class="reel" :class="{ spin: !broken }">
                  <svg viewBox="-10 -10 20 20"><circle r="9" class="disc" /><path d="M0 -8 V8 M-7 -4 L7 4 M-7 4 L7 -4" class="spokes" /></svg>
                </div>
                <i class="led" />
              </div>
              <b>CTA-Instanz</b>
              <span>{{ broken ? 'z. B. Bibliothek defekt' : 'schreibt aufs Band' }}</span>
            </div>
          </div>

          <div class="timeline">
            <div class="seg ok" :class="{ on: broken && !full }">
              <span class="mono">0 – 8 h</span>
              <b>Der Buffer fängt alles auf</b>
            </div>
            <div class="seg late" :class="{ on: full }">
              <span class="mono">danach</span>
              <b>Daten warten auf dem EOS des Experiments</b>
            </div>
          </div>
        </div>

        <div class="facts">
          <div v-for="(f, i) in facts" :key="f.title" class="card fact" v-motion :initial="{ opacity: 0, y: 14 }" :enter="{ opacity: 1, y: 0, transition: { delay: 500 + i * 120, duration: 450 } }">
            <b>{{ f.title }}</b>
            <p>{{ f.text }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.detail { height: 100%; display: flex; flex-direction: column; gap: 14px; }
.d-head { display: flex; align-items: center; gap: 14px; }
.d-tag { padding: 5px 11px; border-radius: 999px; font-size: 12px; font-weight: 600; color: var(--accent); background: var(--accent-soft); }
.d-head h3 { margin: 0; font-size: 22px; font-weight: 750; letter-spacing: -0.02em; color: var(--ink); }
.d-src { margin-left: auto; max-width: 260px; font-size: 10.5px; line-height: 1.35; text-align: right; color: var(--muted); }
.layout { flex: 1; min-height: 0; display: grid; grid-template-columns: 1fr 1.75fr; gap: 20px; }

.stats { display: grid; gap: 12px; }
.stat { display: flex; flex-direction: column; justify-content: center; gap: 4px; padding: 10px 22px; }
.value { font-size: 40px; font-weight: 800; line-height: 1; letter-spacing: -0.04em; color: var(--ink); font-variant-numeric: tabular-nums; }
.value small { margin-left: 6px; font-size: 18px; font-weight: 700; color: var(--accent); }
.label { font-size: 16px; font-weight: 650; color: var(--ink); }
.sub { margin-top: 3px; font-size: 13px; color: var(--muted); }

.right { display: flex; flex-direction: column; gap: 12px; min-width: 0; }
.outage { flex: 1; display: flex; flex-direction: column; justify-content: space-between; gap: 10px; padding: 14px 20px 16px; }
.o-head { display: flex; justify-content: space-between; align-items: baseline; }
.o-title { font-size: 15px; font-weight: 650; color: var(--ink); }
.legend { font-size: 12px; color: var(--muted); }

.flow { display: grid; grid-template-columns: 1fr 64px 1.3fr 64px 0.95fr; align-items: stretch; }
.node {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 3px;
  min-height: 104px;
  padding: 10px 14px;
  border: 1px solid var(--line);
  border-radius: 14px;
  background: var(--surface-2);
  transition: border-color 0.4s, background 0.4s;
}
.node b { font-size: 15px; font-weight: 700; color: var(--ink); }
.node span { font-size: 12px; color: var(--muted); }

/* Experiment */
.node.exp { flex-direction: row; align-items: center; gap: 10px; }
.node.exp > div:not(.pile) { display: flex; flex-direction: column; gap: 1px; }
.node.exp .eg { font-size: 10px; letter-spacing: 0.08em; text-transform: uppercase; }
.node.exp.jam { border-color: var(--amber); background: var(--amber-soft); }
.detector { flex: none; width: 42px; height: 42px; }
.detector .ring { fill: none; stroke: var(--accent); stroke-width: 1.6; }
.detector .ring.outer { stroke-width: 2.4; }
.detector .core { fill: var(--accent); }
.detector .spoke { stroke: var(--accent); stroke-width: 1.2; opacity: 0.55; }
.detector.busy { animation: spin 6s linear infinite; }
.jam .detector .ring, .jam .detector .spoke { stroke: var(--amber); }
.jam .detector .core { fill: var(--amber); }
.pile { position: absolute; right: 8px; bottom: 8px; display: flex; gap: 3px; }
.pile i { width: 8px; height: 8px; border-radius: 2px; background: var(--amber); animation: pop 0.3s cubic-bezier(0.3, 1.8, 0.5, 1); }

/* Arrows with travelling packets */
.arrow { display: flex; flex-direction: column; justify-content: center; align-items: center; gap: 6px; padding: 0 6px; }
.arrow span { font-size: 11px; font-weight: 600; color: var(--accent); transition: color 0.3s; }
.track { position: relative; width: 100%; height: 12px; }
.line { position: absolute; left: 0; right: 0; top: 5px; height: 2px; background: var(--accent); opacity: 0.35; }
.pkt { position: absolute; top: 1px; width: 10px; height: 10px; margin-left: -5px; border-radius: 3px; background: var(--accent); box-shadow: 0 1px 4px rgba(47, 107, 255, 0.45); }
.arrow.jam span { color: #9a6400; }
.arrow.jam .pkt { background: var(--amber); box-shadow: none; }
.arrow.jam .line { background: var(--amber); }
.arrow.blocked span { color: var(--rose); }
.arrow.blocked .line { height: 0; background: none; border-top: 2px dashed var(--rose); opacity: 0.8; }
.cut { position: absolute; left: 50%; top: -2px; width: 16px; height: 16px; margin-left: -8px; animation: pop 0.35s cubic-bezier(0.3, 1.8, 0.5, 1); }
.cut path { stroke: var(--rose); stroke-width: 2.6; stroke-linecap: round; }

/* Buffer: 10 server bars */
.node.buffer { gap: 6px; border-color: var(--accent); background: var(--accent-soft); }
.b-head { display: flex; flex-direction: column; gap: 1px; }
.b-head b { white-space: nowrap; }
.node.buffer em { font-size: 11.5px; font-style: normal; font-weight: 600; color: var(--accent); white-space: nowrap; }
.node.buffer.full { border-color: var(--amber); background: var(--amber-soft); }
.node.buffer.full em { color: #9a6400; }
.servers { display: grid; grid-template-columns: repeat(10, 1fr); gap: 4px; height: 36px; }
.srv { position: relative; border-radius: 3px; background: var(--surface); border: 1px solid #c9d6f5; overflow: hidden; }
.srv-fill { position: absolute; inset: 0; background: var(--accent); transform-origin: bottom; }
.full .srv { border-color: #f0cf8a; }
.full .srv-fill { background: var(--amber); }
.srv-led { position: absolute; top: 3px; left: 50%; width: 4px; height: 4px; margin-left: -2px; border-radius: 50%; background: #c9d6f5; }
.srv-led.on { background: #fff; animation: blink 0.35s ease-in-out infinite; }

/* CTA instance */
.node.cta { padding-top: 26px; align-items: center; text-align: center; border-color: #c9bfff; background: var(--violet-soft); }
.node.cta.broken { border-color: var(--rose); background: var(--rose-soft); animation: alarm 1.2s ease-in-out infinite; }
.node.cta.hit { animation: shake 0.45s ease-in-out; }
.node.cta.broken b { color: var(--rose); }
.node .alarm {
  position: absolute; top: 6px; left: 50%; transform: translateX(-50%) scale(0.4);
  padding: 2px 8px; border-radius: 999px; white-space: nowrap;
  font-size: 10.5px; font-weight: 700; color: #fff; background: var(--rose);
  opacity: 0; transition: opacity 0.2s, transform 0.3s cubic-bezier(0.3, 1.8, 0.5, 1);
}
.node.broken .alarm { opacity: 1; transform: translateX(-50%) scale(1); }
.drive { position: relative; display: flex; gap: 6px; padding: 5px 16px 5px 7px; margin-bottom: 4px; border-radius: 7px; background: #1e1b33; }
.reel { width: 20px; height: 20px; }
.reel svg { width: 100%; height: 100%; display: block; }
.reel.spin svg { animation: spin 0.7s linear infinite; }
.reel .disc { fill: #7a5c41; }
.reel .spokes { stroke: #c9bfff; stroke-width: 1.4; stroke-linecap: round; }
.led { position: absolute; right: 6px; top: 50%; width: 6px; height: 6px; margin-top: -3px; border-radius: 50%; background: var(--teal); }
.broken .led { background: var(--rose); animation: blink 0.5s ease-in-out infinite; }

.timeline { display: grid; grid-template-columns: 1fr 1.25fr; gap: 6px; }
.seg { display: flex; flex-direction: column; gap: 4px; padding: 10px 16px; opacity: 0.5; transition: opacity 0.4s; }
.seg.on { opacity: 1; }
.seg span { font-size: 11.5px; font-weight: 600; letter-spacing: 0.06em; text-transform: uppercase; }
.seg b { font-size: 15px; font-weight: 650; color: var(--ink); }
.seg.ok { border-radius: 12px 4px 4px 12px; background: var(--accent-soft); }
.seg.ok span { color: var(--accent); }
.seg.late { border-radius: 4px 12px 12px 4px; background: var(--amber-soft); }
.seg.late span { color: #9a6400; }

.facts { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; }
.fact { padding: 10px 16px; }
.fact b { font-size: 14.5px; color: var(--ink); }
.fact p { margin: 4px 0 0; font-size: 12.5px; line-height: 1.45; color: var(--ink-2); opacity: 1; }

@keyframes spin { to { transform: rotate(360deg); } }
@keyframes blink { 50% { opacity: 0.3; } }
@keyframes pop { from { transform: scale(0.3); } to { transform: scale(1); } }
@keyframes shake { 20% { transform: translateX(-4px) rotate(-1deg); } 40% { transform: translateX(4px) rotate(1deg); } 60% { transform: translateX(-3px); } 80% { transform: translateX(2px); } }
@keyframes alarm { 50% { box-shadow: 0 0 0 5px rgba(229, 72, 77, 0.18); } }
</style>
