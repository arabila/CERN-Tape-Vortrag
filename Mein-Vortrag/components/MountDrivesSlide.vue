<script setup lang="ts">
// 1, 3 and 5 drives mounting at the same time. Click 0: Mount + BOT, click 1: full cycle incl. dismount.
// Data: V. Bahyl, CTA Workshop 2026 (slides 10, 11, 13, 14).
import { computed } from 'vue'

const props = defineProps<{ step: number }>()
const cycle = computed(() => props.step >= 1)

const panels = [
  { n: 1, m: 'm1', c: 'c1', perHour: 32, color: 'var(--accent)' },
  { n: 3, m: 'm3', c: 'c3', perHour: 70, color: 'var(--violet)' },
  { n: 5, m: 'm5', c: 'c5', perHour: 90, color: 'var(--rose)' },
]
</script>

<template>
  <SlideFrame :section="5" :title="cycle ? 'Ganzer Zyklus: Mount, BOT und Dismount' : 'Mehr Laufwerke gleichzeitig'" kicker="Diamondback · Messung 2: 1, 3 und 5 Laufwerke">
    <div class="wrap">
      <div class="top">
        <div class="mode">
          <span :class="{ on: !cycle }">Mount bis BOT</span>
          <span :class="{ on: cycle }">+ Dismount</span>
        </div>
        <TierLegend />
      </div>
      <div class="grid">
        <MountScatter
          v-for="p in panels" :key="`${p.n}-${cycle}`"
          :set="cycle ? p.c : p.m" :x-max="cycle ? 360 : 240" :x-step="cycle ? 60 : 40"
          :title="`${p.n} ${p.n === 1 ? 'Laufwerk' : 'Laufwerke'}`"
          :sub="cycle ? `~${p.perHour} Wechsel pro Stunde` : 'gleichzeitig'"
          :accent="p.color"
          :mark-max="p.n > 1"
        />
      </div>
      <div class="take">
        <template v-if="!cycle">
          <b>Die Stufen verschwimmen.</b> Mit 5 Laufwerken wartet schon eine T1-Kassette oft fast eine Minute, Ausreißer gehen bis über 3½ min. Der Greifer ist der Engpass, nicht das Laufwerk.
        </template>
        <template v-else>
          <b>Robotik am Limit.</b> Ein ganzer Zyklus dauert mindestens ~60 s (1 LW), ~80 s (3 LW) und ~100 s (5 LW). Mehr als ~90 Wechsel pro Stunde schafft die Library nicht. Die IBM-Doku nennt nur Werte für T1.
        </template>
      </div>
    </div>
  </SlideFrame>
</template>

<style scoped>
.wrap { height: 100%; display: flex; flex-direction: column; gap: 10px; }
.top { display: flex; align-items: center; justify-content: space-between; gap: 16px; }
.mode { display: inline-flex; padding: 3px; border-radius: 999px; background: var(--surface-2); }
.mode span { padding: 5px 14px; border-radius: 999px; font-size: 13px; font-weight: 600; color: var(--muted); transition: background 0.3s, color 0.3s; }
.mode span.on { color: #fff; background: var(--ink); }
.grid { flex: 1; min-height: 0; display: grid; grid-template-columns: repeat(3, 1fr); gap: 14px; }
.take { padding: 10px 14px; border-radius: 12px; font-size: 14px; line-height: 1.4; color: var(--ink-2); background: var(--rose-soft); box-shadow: inset 3px 0 0 var(--rose); }
.take b { color: var(--ink); }
</style>
