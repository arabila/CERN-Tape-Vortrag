<script setup lang="ts">
// Rack-sized tape libraries CERN looked at. Specs from slide 1 of
// "Evaluation of the IBM Diamondback rack-space tape library" (V. Bahyl, CTA Workshop 2026).
type Lib = { vendor: string; model: string; img: string; dims: string; weight: string; slots: string; pb: string; drives: number; tested?: boolean }

const libs: Lib[] = [
  { vendor: 'Spectra Logic', model: 'Cube', img: '/diamondback/cube.png', dims: '202 × 91 × 115 cm', weight: '850 kg', slots: '1.670', pb: '66,8', drives: 16 },
  { vendor: 'IBM', model: 'Diamondback', img: '/diamondback/diamondback.png', dims: '203 × 60 × 123 cm', weight: '785 kg', slots: '1.584', pb: '63,4', drives: 14, tested: true },
  { vendor: 'Quantum', model: 'Scalar i7 Raptor', img: '/diamondback/raptor.png', dims: '229 × 61 × 138 cm', weight: '1.104 kg', slots: '2.016', pb: '80,6', drives: 20 },
  { vendor: 'BDT', model: 'Orion', img: '/diamondback/orion.png', dims: '231 × 60 × 120 cm', weight: '1.000 kg', slots: '1.115', pb: '44,6', drives: 24 },
]
</script>

<template>
  <SlideFrame :section="5" title="Bandbibliotheken im Rack-Format" kicker="Diamondback · Was CERN sich angesehen hat">
    <div class="wrap">
      <div class="grid">
        <article
          v-for="(l, i) in libs" :key="l.model" class="card lib" :class="{ tested: l.tested }"
          v-motion :initial="{ opacity: 0, y: 16 }" :enter="{ opacity: 1, y: 0, transition: { delay: i * 90, duration: 450 } }"
        >
          <span v-if="l.tested" class="badge">Am CERN getestet</span>
          <div class="photo"><img :src="l.img" :alt="`${l.vendor} ${l.model}`"></div>
          <div class="name">
            <span>{{ l.vendor }}</span>
            <b>{{ l.model }}</b>
          </div>
          <div class="cap"><b>{{ l.pb }} PB</b><span>mit LTO-10 (40 TB)</span></div>
          <dl>
            <div><dt>Slots</dt><dd>{{ l.slots }}</dd></div>
            <div><dt>Laufwerke</dt><dd>bis {{ l.drives }}</dd></div>
            <div><dt>Maße</dt><dd>{{ l.dims }}</dd></div>
            <div><dt>Gewicht</dt><dd>{{ l.weight }}</dd></div>
          </dl>
        </article>
      </div>

      <div class="ref" v-motion :initial="{ opacity: 0, y: 10 }" :enter="{ opacity: 1, y: 0, transition: { delay: 450, duration: 400 } }">
        <div class="ref-item"><b>Ein Rack, ~60 PB</b><span>Alle vier sind etwa so groß wie ein Server-Rack (60–91 cm breit).</span></div>
        <div class="ref-item"><b>Zum Vergleich: TS4500 am CERN</b><span>18 Frames, 22.000 Slots, 64 Laufwerke. Dafür bräuchte man 14 Diamondbacks.</span></div>
        <div class="ref-item"><b>Langzeittest</b><span>Leihgabe von IBM, mit echten CMS-Daten in Produktion (Bericht Sept. 2026).</span></div>
      </div>
    </div>
  </SlideFrame>
</template>

<style scoped>
.wrap { height: 100%; display: flex; flex-direction: column; gap: 14px; }
.grid { flex: 1; min-height: 0; display: grid; grid-template-columns: repeat(4, 1fr); gap: 14px; }
.lib { position: relative; display: flex; flex-direction: column; gap: 8px; min-height: 0; padding: 14px 16px 12px; }
.lib.tested { border-color: var(--accent); box-shadow: 0 0 0 2px var(--accent-soft), var(--shadow); }
.badge {
  position: absolute; top: 12px; right: 12px; z-index: 1;
  padding: 4px 10px; border-radius: 999px;
  font-size: 11.5px; font-weight: 700; color: #fff; background: var(--accent);
}
.photo { flex: 1; min-height: 0; display: flex; align-items: flex-end; justify-content: center; padding-top: 6px; }
.photo img { height: 100%; max-width: 100%; object-fit: contain; filter: drop-shadow(0 6px 10px rgba(15, 23, 42, 0.18)); }
.name { display: flex; flex-direction: column; padding-top: 8px; border-top: 1px solid var(--line); }
.name span { font: 600 11px 'JetBrains Mono', monospace; letter-spacing: 0.06em; text-transform: uppercase; color: var(--muted); }
.name b { font-size: 19px; color: var(--ink); }
.cap { display: flex; align-items: baseline; gap: 8px; }
.cap b { font-size: 24px; font-weight: 750; letter-spacing: -0.02em; color: var(--ink); }
.tested .cap b { color: var(--accent); }
.cap span { font-size: 12px; color: var(--muted); }
dl { margin: 0; display: grid; grid-template-columns: 1fr 1fr; gap: 4px 10px; }
dl div { display: flex; flex-direction: column; }
dt { font-size: 11px; color: var(--muted); }
dd { margin: 0; font-size: 13.5px; font-weight: 600; color: var(--ink-2); }
.ref { display: grid; grid-template-columns: repeat(3, 1fr); gap: 14px; }
.ref-item { display: flex; flex-direction: column; gap: 3px; padding: 10px 14px; border-radius: 12px; background: var(--accent-soft); box-shadow: inset 3px 0 0 var(--accent); }
.ref-item b { font-size: 14.5px; color: #1d4fd8; }
.ref-item span { font-size: 13px; line-height: 1.35; color: var(--ink-2); }
</style>
