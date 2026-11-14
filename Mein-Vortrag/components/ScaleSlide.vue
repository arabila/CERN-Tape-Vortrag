<script setup lang="ts">
// "What does 2 EB/year look like?" – photos and figures from the CTA Roadmap talk
// (M. Davis, CTA Workshop 2026, slides 11–12) and the welcome talk (data on tape at CERN).
const bars = [
  { label: 'Rekordmonat Okt. 2025', v: 62, txt: '62 PB', color: '#94a3b8' },
  { label: 'Eine Bandlieferung (Foto)', v: 150, txt: '150 PB', color: 'var(--amber)' },
  { label: 'Run-3-Rekordjahr 2025', v: 304, txt: '+304 PB', color: 'var(--accent)' },
  { label: 'Alles auf Band am CERN, 2026', v: 1480, txt: '1,48 EB', color: 'var(--ink-2)' },
  { label: 'Run-4: pro Jahr', v: 2000, txt: '2 EB', color: 'var(--rose)' },
]
const MAX = 2000
</script>

<template>
  <SlideFrame :section="4" title="Wie 2 EB pro Jahr aussehen" kicker="Roadmap · Die Größenordnung bei Run-4">
    <div class="wrap">
      <figure class="card photo tall" v-motion :initial="{ opacity: 0, y: 16 }" :enter="{ opacity: 1, y: 0, transition: { duration: 450 } }">
        <img src="/roadmap/tapes-one-day.jpg" alt="Vladimír Bahyl neben einem Turm aus Bandkassetten">
        <figcaption><b>Ein Tag Run-3</b><span>So viele Kassetten füllt das CERN an einem Tag Datennahme.</span></figcaption>
      </figure>
      <figure class="card photo" v-motion :initial="{ opacity: 0, y: 16 }" :enter="{ opacity: 1, y: 0, transition: { delay: 120, duration: 450 } }">
        <img src="/roadmap/delivery-150pb.jpg" alt="Paletten mit Bandkassetten, 150 PB">
        <figcaption><b>150 PB auf Paletten</b><span>Bei Run-4-Datenraten in <em>2–3 Wochen</em> voll.</span></figcaption>
      </figure>

      <div class="card compare" v-motion :initial="{ opacity: 0, y: 16 }" :enter="{ opacity: 1, y: 0, transition: { delay: 240, duration: 450 } }">
        <div class="head">Im Vergleich</div>
        <div v-for="(b, i) in bars" :key="b.label" class="bar">
          <div class="lab"><span>{{ b.label }}</span><b :style="{ color: b.color }">{{ b.txt }}</b></div>
          <div class="track">
            <div class="fill" :style="{ width: `${Math.max((b.v / MAX) * 100, 1.5)}%`, background: b.color, transitionDelay: `${0.5 + i * 0.12}s` }" />
          </div>
        </div>
        <div class="foot">
          <div><b>8 EB</b><span>Rohdaten Run-4</span></div>
          <div><b>12 EB</b><span>Rohdaten Run-5</span></div>
          <div><b>300 GB/s</b><span>Schreibrate</span></div>
        </div>
      </div>
    </div>
  </SlideFrame>
</template>

<style scoped>
.wrap { height: 100%; display: grid; grid-template-columns: 0.62fr 1.25fr 1fr; gap: 16px; }
.photo { position: relative; margin: 0; overflow: hidden; min-height: 0; }
.photo img { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; }
.photo.tall img { object-position: 50% 30%; }
figcaption {
  position: absolute; left: 10px; right: 10px; bottom: 10px;
  display: flex; flex-direction: column; gap: 2px;
  padding: 10px 12px; border-radius: 12px;
  background: rgba(255, 255, 255, 0.92); backdrop-filter: blur(6px);
}
figcaption b { font-size: 16px; color: var(--ink); }
figcaption span { font-size: 13px; line-height: 1.35; color: var(--ink-2); }
figcaption em { font-style: normal; font-weight: 700; color: var(--rose); }

.compare { display: flex; flex-direction: column; gap: 17px; padding: 16px 18px; min-height: 0; }
.head { font: 600 11px 'JetBrains Mono', monospace; letter-spacing: 0.08em; text-transform: uppercase; color: var(--muted); }
.bar { display: flex; flex-direction: column; gap: 5px; }
.lab { display: flex; align-items: baseline; justify-content: space-between; gap: 8px; }
.lab span { font-size: 13.5px; color: var(--ink-2); }
.lab b { font: 700 15px 'JetBrains Mono', monospace; }
.track { height: 14px; border-radius: 7px; background: var(--surface-2); overflow: hidden; }
.fill { height: 100%; border-radius: 6px; animation: grow 0.8s cubic-bezier(0.2, 0.7, 0.2, 1) both; animation-delay: inherit; transform-origin: left; }
.bar:nth-child(2) .fill { animation-delay: 0.5s; }
.bar:nth-child(3) .fill { animation-delay: 0.62s; }
.bar:nth-child(4) .fill { animation-delay: 0.74s; }
.bar:nth-child(5) .fill { animation-delay: 0.86s; }
.bar:nth-child(6) .fill { animation-delay: 0.98s; }
@keyframes grow { from { transform: scaleX(0); } to { transform: scaleX(1); } }
.foot { margin-top: auto; display: grid; grid-template-columns: repeat(3, 1fr); gap: 8px; padding-top: 12px; border-top: 1px solid var(--line); }
.foot div { display: flex; flex-direction: column; }
.foot b { font-size: 19px; font-weight: 750; color: var(--ink); }
.foot span { font-size: 11.5px; color: var(--muted); }
</style>
