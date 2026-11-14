<script setup lang="ts">
// Short interlude: an evening in Hamburg around the workshop (own photos).
const food = [
  { img: '/workshop/img_1410.jpg', pos: '50% 50%' },
  { img: '/workshop/img_1411.jpg', pos: '50% 50%' },
  { img: '/workshop/img_1412.jpg', pos: '50% 55%' },
]
</script>

<template>
  <div class="grid">
    <figure class="tile tunnel" v-motion :initial="{ opacity: 0, y: 14 }" :enter="{ opacity: 1, y: 0, transition: { duration: 500 } }">
      <img src="/workshop/img_1400.jpg" alt="Alter Elbtunnel" style="object-position: 50% 45%">
      <img src="/workshop/img_1399.jpg" alt="Schacht des Alten Elbtunnels" class="inset">
      <figcaption><b>Alter Elbtunnel</b><span>Von 1911, führt unter der Elbe nach Steinwerder</span></figcaption>
    </figure>
    <figure class="tile" v-motion :initial="{ opacity: 0, y: 14 }" :enter="{ opacity: 1, y: 0, transition: { delay: 120, duration: 500 } }">
      <img src="/workshop/img_1408.jpg" alt="Hamburger Rathaus bei Nacht" style="object-position: 50% 40%">
      <figcaption><b>Rathaus bei Nacht</b><span>Auf dem Weg zum Essen</span></figcaption>
    </figure>
    <div class="food" v-motion :initial="{ opacity: 0, y: 14 }" :enter="{ opacity: 1, y: 0, transition: { delay: 240, duration: 500 } }">
      <figure v-for="f in food" :key="f.img" class="tile"><img :src="f.img" alt="Essen beim Abendessen" :style="{ objectPosition: f.pos }"></figure>
      <div class="food-cap"><b>Gemeinsames Abendessen</b><span>Ins Gespräch kommen mit den anderen Teilnehmern</span></div>
    </div>
  </div>
</template>

<style scoped>
.grid { height: 100%; display: grid; grid-template-columns: 1fr 0.8fr 1.1fr; gap: 12px; }
.tile { position: relative; margin: 0; overflow: hidden; border-radius: var(--radius); box-shadow: var(--shadow); min-height: 0; background: #ddd; }
.tile > img { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; animation: drift 14s ease-in-out infinite alternate; }
@keyframes drift { from { transform: scale(1.02); } to { transform: scale(1.08); } }
.tile > img.inset {
  inset: auto; top: 12px; right: 12px; width: 34%; height: 40%;
  border-radius: 12px; border: 3px solid #fff; box-shadow: var(--shadow); animation: none;
}
figcaption {
  position: absolute; left: 0; right: 0; bottom: 0;
  display: flex; flex-direction: column; gap: 2px;
  padding: 28px 14px 12px;
  background: linear-gradient(transparent, rgba(11, 18, 32, 0.82));
  color: #fff;
}
figcaption b { font-size: 18px; }
figcaption span { font-size: 13px; opacity: 0.9; }
.food { display: grid; grid-template-columns: 1fr 1fr; grid-template-rows: 1fr 1fr auto; gap: 10px; min-height: 0; }
.food .tile:first-child { grid-row: 1 / 3; }
.food-cap { grid-column: 1 / -1; display: flex; flex-direction: column; gap: 2px; padding: 10px 14px; border-radius: 12px; background: var(--amber-soft); box-shadow: inset 3px 0 0 var(--amber); }
.food-cap b { font-size: 16px; color: var(--ink); }
.food-cap span { font-size: 13px; color: var(--ink-2); }
</style>
