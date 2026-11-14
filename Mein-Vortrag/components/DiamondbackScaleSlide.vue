<script setup lang="ts">
// What the Diamondback would mean at CERN scale: counts, list prices, verdict.
// All figures from "Evaluation of the IBM Diamondback rack-space tape library"
// (V. Bahyl, CTA Workshop 2026), slides 24–30.
const counts = [
  { v: '1', l: 'Diamondback heute am CERN', d: 'Leihgabe von IBM für den Test', color: 'var(--muted)' },
  { v: '16', l: 'pro Jahr für die Kapazität', d: '1 EB neue Daten pro Jahr ab 2030, ~62 PB pro Library', color: 'var(--accent)' },
  { v: '~35', l: 'für den Durchsatz', d: '200 GB/s brauchen ~500 LTO-10-Laufwerke, max. 14 pro Library', color: 'var(--rose)' },
]
const reasons = [
  'CERN liest ständig: jede Kassette kann jederzeit gebraucht werden, oft viele auf einmal',
  'Eine TS4500 läuft 15+ Jahre, neue Laufwerke werden einfach nachgerüstet. Die Diamondback kann nur LTO-9 bis LTO-11 und muss danach ersetzt werden',
  'Laufwerke sind per Lizenz an eine Library gebunden',
  'Kein Kostenvorteil gegenüber der TS4500',
]

// Schematic geometry (viewBox 340 wide). FW = TS4500 frame width, RW/GAP = Diamondback rack + gap.
const FW = 340 / 18
const GAP = 5
const RW = (340 - 13 * GAP) / 14
// 14 drives of the TS4500, all sitting together in the first frame (2 columns × 7)
const tsDrives = Array.from({ length: 14 }, (_, i) => [FW / 2 + (i % 2 ? 3.4 : -3.4), 10 + Math.floor(i / 2) * 2.9])
</script>

<template>
  <SlideFrame :section="5" title="Die Diamondback im CERN-Maßstab" kicker="Diamondback · Anzahl und Kosten">
    <div class="wrap">
      <div class="counts">
        <div
          v-for="(c, i) in counts" :key="c.l" class="card count" :style="{ '--c': c.color }"
          v-motion :initial="{ opacity: 0, y: 14 }" :enter="{ opacity: 1, y: 0, transition: { delay: i * 110, duration: 450 } }"
        >
          <b>{{ c.v }}</b>
          <div><strong>{{ c.l }}</strong><span>{{ c.d }}</span></div>
        </div>
      </div>

      <div class="card cost" v-motion :initial="{ opacity: 0, y: 14 }" :enter="{ opacity: 1, y: 0, transition: { delay: 350, duration: 450 } }">
        <div class="head">Listenpreise im Vergleich (Stand 12/2025)</div>
        <div class="top">
          <p class="same">Gleiche Slot-Zahl: ~22.000 Slots</p>
          <div class="delta">+50 % teurer</div>
        </div>
        <!-- Each option: label + price bar + schematic (same slots and drives, 1 big library vs 14 small ones) -->
        <div class="opt">
          <div class="lab"><span><b>IBM TS4500</b> · 1 Library aus 18 Frames, 14 Laufwerke</span><em>~4,3 Mio. CHF</em></div>
          <div class="track"><div class="fill ts" style="width: 65%" /></div>
          <svg viewBox="0 0 340 30" class="lib ts">
            <rect x="0.75" y="2.75" width="338.5" height="26.5" rx="3" class="shell" />
            <line v-for="i in 17" :key="'f' + i" :x1="i * FW" :x2="i * FW" y1="3" y2="29" class="seam" />
            <line x1="5" x2="335" y1="6.5" y2="6.5" class="rail" />
            <rect x="96" y="4.5" width="12" height="4" rx="1" class="robot" />
            <g v-for="r in 2" :key="'s' + r">
              <line v-for="i in 18" :key="'l' + r + i" :x1="(i - 1) * FW + 4" :x2="i * FW - 4" :y1="12 + r * 5" :y2="12 + r * 5" class="slots" />
            </g>
            <circle v-for="(d, i) in tsDrives" :key="'d' + i" :cx="d[0]" :cy="d[1]" r="1.3" class="drive" />
          </svg>
        </div>

        <div class="divider" />

        <div class="opt">
          <div class="lab"><span><b>14 × Diamondback</b> · 14 Racks, je 1 Laufwerk und eigener Roboter</span><em class="hi">~6,6 Mio. CHF</em></div>
          <div class="track"><div class="fill db" style="width: 100%" /></div>
          <svg viewBox="0 0 340 30" class="lib db">
            <g v-for="i in 14" :key="'r' + i" :transform="`translate(${(i - 1) * (RW + GAP)} 0)`"><g class="rack" :style="{ animationDelay: 900 + i * 45 + 'ms' }">
              <rect x="0.75" y="0.75" :width="RW - 1.5" height="28.5" rx="2.5" class="shell" />
              <line x1="3" :x2="RW - 3" y1="4.5" y2="4.5" class="rail" />
              <line v-for="r in 3" :key="'l' + r" x1="4" :x2="RW - 4" :y1="12 + r * 4.5" :y2="12 + r * 4.5" class="slots" />
              <circle :cx="RW / 2" cy="10" r="1.8" class="drive" />
            </g></g>
          </svg>
        </div>
        <div class="legend"><i /> = 1 LTO-9-Laufwerk · ~1.550 Slots pro Diamondback</div>

        <p class="note">Je Library also rund 0,47 Mio. CHF Listenpreis, mit nur einem Laufwerk und ohne Bänder.</p>
      </div>

      <div class="card verdict" v-motion :initial="{ opacity: 0, y: 14 }" :enter="{ opacity: 1, y: 0, transition: { delay: 470, duration: 450 } }">
        <div class="head">Fazit des CERN</div>
        <b class="big">Passt nicht zum CERN-Modell</b>
        <p>Technisch solide, aber gebaut als „Tiefenarchiv“: Daten einmal schreiben und dann jahrelang kaum lesen. CERN braucht etwas anderes:</p>
        <ul>
          <li v-for="r in reasons" :key="r">{{ r }}</li>
        </ul>
      </div>
    </div>
  </SlideFrame>
</template>

<style scoped>
.wrap { height: 100%; display: grid; grid-template-columns: 300px 1fr 1fr; gap: 16px; }
.head { font: 600 11px 'JetBrains Mono', monospace; letter-spacing: 0.08em; text-transform: uppercase; color: var(--muted); }

.counts { display: flex; flex-direction: column; gap: 12px; }
.count { flex: 1; display: flex; align-items: center; gap: 16px; padding: 12px 18px; box-shadow: inset 4px 0 0 var(--c), var(--shadow); }
.count > b { min-width: 74px; font-size: 44px; font-weight: 800; letter-spacing: -0.04em; color: var(--c); }
.count div { display: flex; flex-direction: column; gap: 2px; }
.count strong { font-size: 15px; color: var(--ink); }
.count span { font-size: 12.5px; line-height: 1.35; color: var(--muted); }

.cost, .verdict { display: flex; flex-direction: column; gap: 12px; padding: 18px 20px; min-height: 0; }
.same { margin: 0; font-size: 14px; color: var(--ink-2); opacity: 1; }
.lab { display: flex; justify-content: space-between; align-items: baseline; gap: 10px; }
.lab span { font-size: 13.5px; color: var(--ink-2); }
.lab b { color: var(--ink); }
.lab em { font: normal 700 16px 'JetBrains Mono', monospace; color: var(--ink-2); white-space: nowrap; }
.lab em.hi { color: var(--rose); }
.track { height: 26px; border-radius: 8px; background: var(--surface-2); overflow: hidden; }
.fill { height: 100%; border-radius: 8px; transform-origin: left; animation: grow 0.9s cubic-bezier(0.2, 0.7, 0.2, 1) both; }
.fill.ts { background: #94a3b8; animation-delay: 0.6s; }
.fill.db { background: var(--rose); animation-delay: 0.8s; }
@keyframes grow { from { transform: scaleX(0); } to { transform: scaleX(1); } }
.top { display: flex; justify-content: space-between; align-items: center; gap: 10px; }
.delta { flex-shrink: 0; padding: 4px 12px; border-radius: 999px; font-size: 16px; font-weight: 800; color: var(--rose); background: var(--rose-soft); }
.cost { gap: 10px; }
.opt { display: flex; flex-direction: column; gap: 6px; }
.divider { flex-shrink: 0; height: 1px; margin: 4px 0; background: #cbd5e1; }
.lib { display: block; width: 100%; height: auto; overflow: visible; }
.lib .shell { fill: var(--surface-2); stroke-width: 1.5; }
.ts .shell { stroke: #94a3b8; }
.db .shell { stroke: var(--rose); }
.lib .seam { stroke: #94a3b8; stroke-width: 0.8; opacity: 0.7; }
.lib .rail { stroke: var(--ink-2); stroke-width: 1; opacity: 0.45; }
.lib .robot { fill: var(--ink-2); }
.lib .slots { stroke: var(--muted); stroke-width: 1.4; stroke-dasharray: 1.6 1.4; opacity: 0.35; }
.lib .drive { fill: var(--accent); }
.rack { animation: rise 0.45s cubic-bezier(0.2, 0.7, 0.2, 1) both; }
@keyframes rise { from { opacity: 0; transform: translateY(4px); } }
.legend { display: flex; align-items: center; gap: 5px; font-size: 11.5px; color: var(--muted); }
.legend i { width: 7px; height: 7px; border-radius: 50%; background: var(--accent); }
.note { margin: auto 0 0; padding-top: 10px; border-top: 1px solid var(--line); font-size: 13px; line-height: 1.4; color: var(--muted); opacity: 1; }

.verdict { background: var(--surface); }
.big { font-size: 26px; font-weight: 800; letter-spacing: -0.02em; color: var(--ink); }
.verdict p { margin: 0; font-size: 14px; line-height: 1.4; color: var(--ink-2); opacity: 1; }
.verdict ul { margin: 0; padding: 0; list-style: none; display: grid; gap: 8px; }
.verdict li { position: relative; padding-left: 16px; font-size: 14px; line-height: 1.4; color: var(--ink-2); }
.verdict li::before { content: ''; position: absolute; left: 0; top: 7px; width: 7px; height: 7px; border-radius: 50%; background: var(--rose); }
</style>
