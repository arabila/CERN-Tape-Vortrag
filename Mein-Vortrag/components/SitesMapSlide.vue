<script setup lang="ts">
// Map of the site reports, taken from the Welcome talk (M. Davis, CTA Workshop 2026, slide 8),
// plus which disk system each site puts in front of CTA.
// IHEP/HEPS had no slides on Indico: EOS + CTA per IHEP's own HEPiX/ISGC/CTA-workshop talks (2022–2025).
const stacks = [
  { disk: 'dCache', tape: 'CTA', sites: ['DESY', 'Fermilab', 'PIC', 'European XFEL'], color: 'var(--accent)', soft: 'var(--accent-soft)', note: 'XFEL archiviert über DESY' },
  { disk: 'EOS', tape: 'CTA', sites: ['RAL', 'IHEP / HEPS'], color: 'var(--amber)', soft: 'var(--amber-soft)', note: 'Wie am CERN selbst' },
  { disk: 'dCache', tape: 'Tapeguy', sites: ['TRIUMF'], color: 'var(--violet)', soft: 'var(--violet-soft)', note: 'Eigenes HSM seit 2006, CTA wird geprüft' },
  { disk: 'NFS / dCache', tape: 'ScoutAM', sites: ['SURF'], color: '#64748b', soft: 'var(--surface-2)', note: 'Kein CTA: Versity ScoutAM statt DMF' },
]
</script>

<template>
  <SlideFrame :section="2" title="Wer berichtet hat und womit" kicker="Workshop & Community · Site Reports">
    <div class="wrap">
      <figure class="card map" v-motion :initial="{ opacity: 0, scale: 0.98 }" :enter="{ opacity: 1, scale: 1, transition: { duration: 500 } }">
        <img src="/workshop-sites-map.png" alt="Weltkarte mit den Sites, die beim Workshop berichtet haben">
        <figcaption>Quelle: Welcome-Talk, M. Davis</figcaption>
      </figure>

      <aside class="side">
        <div class="head">Disk-System + Band-System</div>
        <div
          v-for="(s, i) in stacks" :key="`${s.disk}-${s.tape}`" class="card stack" :style="{ '--c': s.color, '--soft': s.soft }"
          v-motion :initial="{ opacity: 0, x: 14 }" :enter="{ opacity: 1, x: 0, transition: { delay: 300 + i * 120, duration: 400 } }"
        >
          <div class="formula"><b>{{ s.disk }}</b><span>+</span><b class="cta">{{ s.tape }}</b></div>
          <div class="sites"><span v-for="n in s.sites" :key="n">{{ n }}</span></div>
          <p>{{ s.note }}</p>
        </div>
      </aside>
    </div>
  </SlideFrame>
</template>

<style scoped>
.wrap { height: 100%; display: grid; grid-template-columns: 1fr 300px; gap: 16px; }
.map { position: relative; margin: 0; overflow: hidden; display: flex; align-items: center; justify-content: center; background: #fff; min-height: 0; }
.map img { width: 100%; height: 100%; object-fit: contain; }
figcaption { position: absolute; right: 12px; bottom: 8px; font: 500 10.5px 'JetBrains Mono', monospace; color: #94a3b8; }
.side { display: flex; flex-direction: column; gap: 8px; min-height: 0; }
.head { font: 600 11px 'JetBrains Mono', monospace; letter-spacing: 0.08em; text-transform: uppercase; color: var(--muted); }
.stack { flex: 1 1 0; min-height: 0; display: flex; flex-direction: column; justify-content: center; gap: 4px; padding: 7px 13px; box-shadow: inset 4px 0 0 var(--c), var(--shadow); }
.formula { display: flex; align-items: baseline; gap: 8px; }
.formula b { font-size: 18px; font-weight: 800; letter-spacing: -0.02em; color: var(--c); }
.formula span { font-size: 15px; color: var(--muted); }
.formula .cta { color: var(--ink-2); }
.sites { display: flex; flex-wrap: wrap; gap: 4px; }
.sites span { padding: 2px 8px; border-radius: 999px; font-size: 12.5px; font-weight: 600; color: var(--ink-2); background: var(--soft, var(--surface-2)); }
.stack p { margin: 0; font-size: 12px; color: var(--muted); opacity: 1; }
</style>
