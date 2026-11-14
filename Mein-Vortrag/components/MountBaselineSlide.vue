<script setup lang="ts">
// Baseline mount test on the IBM Diamondback: one drive, 50 random cartridges per tier.
// Data and reading hints from V. Bahyl, CTA Workshop 2026 (slides 9–10).
// Click 1 opens a window (grows out of hint 4) that animates the dual gripper and explains
// why the tiers come in pairs; click 2 closes it again.
import { computed, ref } from 'vue'

const props = defineProps<{ step?: number }>()
const open = computed(() => (props.step ?? 0) === 1)
const hintEls = ref<HTMLElement[]>([])
// transform origin = centre of hint 4, relative to .body (window overhangs by 14px / 6px)
const origin = computed(() => {
  const el = hintEls.value[3]
  if (!el) return '85% 85%'
  return `${el.offsetLeft + el.offsetWidth / 2 + 14}px ${el.offsetTop + el.offsetHeight / 2 + 6}px`
})
const hints = [
  { t: 'Jeder Punkt eine Kassette', d: 'Zeit vom Griff bis die Kassette am Bandanfang (BOT) steht. 1 Laufwerk, 50 Kassetten pro Ebene.' },
  { t: 'Neun Wolken nebeneinander', d: 'Jede Ebene tiefer kostet Zeit: T1 ab 18 s, T9 bis 111 s.' },
  { t: 'Oben oder unten egal', d: 'Die Wolken stehen fast senkrecht. Der Greifer fährt sehr schnell hoch und runter.' },
  { t: 'Lücken in Paaren', d: 'Kleine Lücke (T1→T2): Tausch im selben Slot. Große Lücke (T2→T3): Kassetten müssen erst weggelegt werden.' },
]
</script>

<template>
  <SlideFrame :section="5" title="Mount-Zeit hängt an der Ebene" kicker="Diamondback · Messung 1: ein Laufwerk">
    <div class="body">
    <div class="wrap">
      <div class="left">
        <TierLegend />
        <MountScatter set="m1" :x-max="120" :x-step="20" big title="1 Laufwerk · Mount bis BOT" sub="450 Kassetten" accent="var(--accent)" />
      </div>
      <ol class="hints">
        <li v-for="(h, i) in hints" :key="h.t" ref="hintEls" :class="{ more: i === 3 }" v-motion :initial="{ opacity: 0, x: 12 }" :enter="{ opacity: 1, x: 0, transition: { delay: 500 + i * 120, duration: 400 } }">
          <span class="n">{{ i + 1 }}</span>
          <div><b>{{ h.t }}</b><p>{{ h.d }}</p><span v-if="i === 3" class="hint-more">Klick: Wie der Greifer das macht ▸</span></div>
        </li>
      </ol>
    </div>
    <ZoomWindow :open="open" :origin="origin" tag="Doppelgreifer" title="Warum die Ebenen paarweise kommen">
      <GripperPairs />
    </ZoomWindow>
    </div>
  </SlideFrame>
</template>

<style scoped>
.body { position: relative; height: 100%; }
.wrap { height: 100%; display: grid; grid-template-columns: 1fr 330px; gap: 18px; }
.left { display: flex; flex-direction: column; gap: 8px; min-height: 0; }
.left > :last-child { flex: 1; }
.hints { margin: 0; padding: 0; list-style: none; display: flex; flex-direction: column; gap: 10px; }
.hints li { display: flex; gap: 12px; padding: 12px 14px; border-radius: 14px; border: 1px solid var(--line); background: var(--surface); }
.n { flex: none; display: grid; place-items: center; width: 26px; height: 26px; border-radius: 8px; font: 700 13px 'JetBrains Mono', monospace; color: var(--accent); background: var(--accent-soft); }
.hints b { font-size: 15.5px; color: var(--ink); }
.hints p { margin: 3px 0 0; font-size: 13.5px; line-height: 1.4; color: var(--ink-2); opacity: 1; }
.hints li.more { border-color: #d8cfff; }
.hint-more { display: inline-block; margin-top: 5px; font: 600 11.5px 'JetBrains Mono', monospace; color: var(--violet); }
</style>
