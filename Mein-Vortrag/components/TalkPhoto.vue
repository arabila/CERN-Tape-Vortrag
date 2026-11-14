<script setup lang="ts">
// Own photo from the Fermilab site report (Tim Messer, 9 Nov 2026), shown inside the programme slide.
// The photo is cropped to the stage; pins are in % of the cropped box.
const pins = [
  { x: 55.7, y: 66.8, label: 'Tim Messer, Fermilab', side: 'left' },
  { x: 26, y: 54.3, label: 'Teilnehmer per Zoom', side: 'right' },
]
</script>

<template>
  <div class="stage">
    <img src="/workshop/img_1433.jpg" alt="" class="bg">
    <div class="photo">
      <img src="/workshop/img_1433.jpg" alt="Tim Messer hält den Fermilab-Vortrag im Flash Seminar Room">
      <div
        v-for="(p, i) in pins" :key="p.label" class="pin" :class="p.side"
        :style="{ left: `${p.x}%`, top: `${p.y}%`, animationDelay: `${0.5 + i * 0.3}s` }"
      >
        <i /><span>{{ p.label }}</span>
      </div>
      <div class="cap">
        <img src="/logos/fermilab.png" alt="Fermilab">
        <div><b>„The first 18 months with CTA“</b><span>Site Report Fermilab · Montag, 9. Nov.</span></div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.stage { position: relative; height: 100%; display: flex; justify-content: center; border-radius: var(--radius); overflow: hidden; background: #111827; }
.bg { position: absolute; inset: -30px; width: calc(100% + 60px); height: calc(100% + 60px); object-fit: cover; filter: blur(24px) brightness(0.55); }
.photo { position: relative; height: 100%; aspect-ratio: 1200 / 760; overflow: hidden; box-shadow: 0 10px 40px rgba(0, 0, 0, 0.35); }
.photo > img { width: 100%; height: 100%; object-fit: cover; object-position: 50% 41.9%; display: block; }

.pin { position: absolute; display: flex; align-items: center; gap: 7px; transform: translate(-8px, -8px); opacity: 0; animation: pin-in 0.5s ease forwards; }
.pin.right { transform: translate(-8px, -8px); }
.pin.left { flex-direction: row-reverse; transform: translate(calc(-100% + 8px), -8px); }
.pin i { flex: none; width: 16px; height: 16px; border-radius: 50%; background: var(--accent); animation: pulse 1.8s ease-out infinite; }
.pin span { padding: 6px 12px; border-radius: 9px; font-size: 15px; font-weight: 700; white-space: nowrap; color: #fff; background: rgba(11, 18, 32, 0.85); }
@keyframes pin-in { to { opacity: 1; } }
@keyframes pulse { 0% { box-shadow: 0 0 0 0 rgba(47, 107, 255, 0.7); } 100% { box-shadow: 0 0 0 14px rgba(47, 107, 255, 0); } }

.cap {
  position: absolute; left: 16px; bottom: 16px;
  display: flex; align-items: center; gap: 12px;
  padding: 10px 16px 10px 12px; border-radius: 14px;
  background: rgba(255, 255, 255, 0.94);
}
.cap img { height: 34px; }
.cap div { display: flex; flex-direction: column; }
.cap b { font-size: 17px; color: var(--ink); }
.cap span { font-size: 12.5px; color: var(--muted); }
</style>
