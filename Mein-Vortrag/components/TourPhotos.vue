<script setup lang="ts">
// Photos from the DESY campus tour on the first workshop morning (own photos, 9 Nov 2026).
type Tile = { img: string; title: string; text: string; pos?: string; area: string; tag?: string }

const tiles: Tile[] = [
  { area: 'big', img: '/workshop/img_1430.jpg', title: 'PETRA IV', text: 'Meine Gruppe: hier entsteht aus PETRA III eine Röntgenquelle der nächsten Generation.', pos: '50% 60%', tag: 'Meine Gruppe' },
  { area: 'p3', img: '/workshop/img_1422.jpg', title: 'PETRA III · Halle „Max von Laue“', text: 'Speicherring mit Messplätzen, hier auch EMBL', pos: '50% 35%' },
  { area: 'ion', img: '/workshop/img_1424.jpg', title: 'Ionen-Spektrometer an PETRA III', text: 'Messplatz der Uni Gießen', pos: '50% 45%' },
  { area: 'flash', img: '/workshop/img_1431.jpg', title: 'FLASH · Halle „Albert Einstein“', text: 'Freie-Elektronen-Laser, Blick von oben', pos: '50% 50%' },
  { area: 'campus', img: '/workshop/img_1416.jpg', title: 'DESY-Campus', text: 'Hamburg-Bahrenfeld', pos: '50% 40%' },
]
</script>

<template>
  <div class="grid">
    <figure
      v-for="(t, i) in tiles" :key="t.img" class="tile" :class="t.area" :style="{ gridArea: t.area }"
      v-motion :initial="{ opacity: 0, scale: 0.96 }" :enter="{ opacity: 1, scale: 1, transition: { delay: i * 110, duration: 500 } }"
    >
      <img :src="t.img" :alt="t.title" :style="{ objectPosition: t.pos }">
      <span v-if="t.tag" class="tag">{{ t.tag }}</span>
      <figcaption><b>{{ t.title }}</b><span>{{ t.text }}</span></figcaption>
    </figure>
  </div>
</template>

<style scoped>
.grid {
  height: 100%;
  display: grid;
  grid-template-columns: 1.15fr 1fr 1fr 0.7fr;
  grid-template-rows: 1fr 1fr;
  grid-template-areas: 'big p3 ion campus' 'big flash flash campus';
  gap: 12px;
}
.tile { position: relative; margin: 0; overflow: hidden; border-radius: var(--radius); box-shadow: var(--shadow); min-height: 0; background: #ddd; }
.tile img { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; animation: drift 14s ease-in-out infinite alternate; }
.tile:nth-child(2n) img { animation-delay: -5s; }
@keyframes drift { from { transform: scale(1.02); } to { transform: scale(1.09); } }
.tile.big { box-shadow: 0 0 0 3px var(--teal), var(--shadow); }
.tag { position: absolute; top: 12px; left: 12px; padding: 5px 11px; border-radius: 999px; font-size: 12px; font-weight: 700; color: #fff; background: var(--teal); }
figcaption {
  position: absolute; left: 0; right: 0; bottom: 0;
  display: flex; flex-direction: column; gap: 2px;
  padding: 26px 14px 11px;
  background: linear-gradient(transparent, rgba(11, 18, 32, 0.82));
  color: #fff;
}
figcaption b { font-size: 15px; }
figcaption span { font-size: 12.5px; line-height: 1.35; opacity: 0.9; }
.big figcaption b { font-size: 22px; }
.big figcaption span { font-size: 14px; }
</style>
