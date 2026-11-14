<script setup lang="ts">
// Same 5 drives, but with a 5 min pause after each mount (simulates a ~120 GB transfer).
// Data: V. Bahyl, CTA Workshop 2026 (slides 15–17).
const panels = [
  { set: 'm1', title: '1 Laufwerk', sub: 'Referenz', color: 'var(--accent)' },
  { set: 'm5', title: '5 Laufwerke', sub: 'ohne Pause', color: 'var(--rose)' },
  { set: 's5', title: '5 Laufwerke', sub: '5 min Pause pro Kassette', color: 'var(--teal)' },
]
const facts = [
  { t: 'Pause = realistischer Betrieb', d: '5 min entsprechen einem Transfer von ~120 GB. Dann sieht das Bild aus wie mit einem Laufwerk.' },
  { t: 'CMS schreibt, Nov. 2025', d: '3 Tage, alle 5 Laufwerke parallel bei ~400 MB/s. Die Robotik hielt locker mit.' },
  { t: 'CMS liest, Feb. 2026', d: 'Nur ~150 MB/s pro Laufwerk. Grund war die Datenlage auf dem Band, nicht die Library.' },
]
</script>

<template>
  <SlideFrame :section="5" title="Mit Pausen wie im echten Betrieb" kicker="Diamondback · Messung 3: Sleep-Test">
    <div class="wrap">
      <TierLegend />
      <div class="grid">
        <MountScatter v-for="p in panels" :key="p.set" :set="p.set" :x-max="240" :x-step="40" :title="p.title" :sub="p.sub" :accent="p.color" :mark-max="p.set === 'm5'" />
      </div>
      <div class="facts">
        <div v-for="(f, i) in facts" :key="f.t" class="fact" :class="{ main: i === 0 }"><b>{{ f.t }}</b><span>{{ f.d }}</span></div>
      </div>
    </div>
  </SlideFrame>
</template>

<style scoped>
.wrap { height: 100%; display: flex; flex-direction: column; gap: 10px; }
.grid { flex: 1; min-height: 0; display: grid; grid-template-columns: repeat(3, 1fr); gap: 14px; }
.facts { display: grid; grid-template-columns: repeat(3, 1fr); gap: 14px; }
.fact { display: flex; flex-direction: column; gap: 3px; padding: 10px 14px; border-radius: 12px; background: var(--surface-2); box-shadow: inset 3px 0 0 #b6c0ce; }
.fact.main { background: var(--teal-soft); box-shadow: inset 3px 0 0 var(--teal); }
.fact b { font-size: 14.5px; color: var(--ink); }
.fact span { font-size: 13px; line-height: 1.38; color: var(--ink-2); }
</style>
