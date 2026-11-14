<script setup lang="ts">
// Programme of the 5th CTA Workshop (indico.desy.de/event/52689), simplified into blocks.
// Clicks walk through it: tour → tour photos → dinner → evening photos → talks → photo from a talk.
import { computed } from 'vue'

const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

type Kind = 'key' | 'site' | 'dev' | 'ops' | 'extra' | 'dinner'
type Focus = 'all' | 'tour' | 'dinner' | 'talks'
type View = 'plan' | 'tour' | 'dinner' | 'talk'
const PLAN: [Focus, View][] = [
  ['all', 'plan'], ['tour', 'plan'], ['tour', 'tour'], ['dinner', 'plan'], ['dinner', 'dinner'], ['talks', 'plan'], ['talks', 'talk'],
]
const state = computed(() => PLAN[Math.min(Math.max(props.step, 0), PLAN.length - 1)])
const focus = computed(() => state.value[0])
const view = computed(() => state.value[1])
const TITLES: Record<View, [string, string]> = {
  plan: ['Zwei Tage CTA bei DESY in Hamburg', 'Workshop & Community · Programm'],
  tour: ['Campus-Tour bei DESY', 'Workshop & Community · Montagmorgen'],
  dinner: ['Abends in Hamburg', 'Workshop & Community · Abendessen'],
  talk: ['Aus dem Vortragssaal', 'Workshop & Community · Vorträge'],
}
const inFocus = (b: { kind: Kind; label: string }) =>
  focus.value === 'tour' ? b.label === 'DESY-Tour' : focus.value === 'dinner' ? b.kind === 'dinner' : !['extra', 'dinner'].includes(b.kind)
const START = 9
const END = 21
const days: { name: string; date: string; blocks: { from: number; to: number; kind: Kind; label: string }[] }[] = [
  {
    name: 'Montag',
    date: '9. Nov.',
    blocks: [
      { from: 9.5, to: 11, kind: 'extra', label: 'DESY-Tour' },
      { from: 11.25, to: 12.5, kind: 'key', label: 'Welcome · Keynotes' },
      { from: 14, to: 14.75, kind: 'site', label: 'Sites' },
      { from: 14.75, to: 15.5, kind: 'dev', label: 'v6 · gRPC' },
      { from: 16, to: 17.25, kind: 'dev', label: 'PostgreSQL · SDK · CLI' },
      { from: 18.5, to: 20.75, kind: 'dinner', label: 'Workshop-Dinner' },
    ],
  },
  {
    name: 'Dienstag',
    date: '10. Nov.',
    blocks: [
      { from: 9, to: 10, kind: 'key', label: 'Keynotes' },
      { from: 10, to: 10.75, kind: 'site', label: 'Sites' },
      { from: 11.25, to: 12.25, kind: 'ops', label: 'Betrieb am CERN' },
      { from: 14, to: 14.75, kind: 'dev', label: 'Tools · S3' },
      { from: 14.75, to: 15.5, kind: 'key', label: 'Roadmap' },
      { from: 16, to: 17.5, kind: 'extra', label: 'Hands-on' },
    ],
  },
]
const legend: { kind: Kind; label: string }[] = [
  { kind: 'key', label: 'Plenum & Keynotes' },
  { kind: 'site', label: 'Site Reports' },
  { kind: 'dev', label: 'Entwicklung' },
  { kind: 'ops', label: 'Betrieb' },
  { kind: 'extra', label: 'Tour & Hands-on' },
  { kind: 'dinner', label: 'Abendessen' },
]
const facts = [
  { big: '5.', text: 'CTA Workshop' },
  { big: '2', text: 'Keynotes' },
  { big: '5', text: 'Site Reports' },
  { big: '14', text: 'Technik-Vorträge' },
  { big: '1', text: 'Hands-on mit dem CTA-Team' },
]
const pos = (h: number) => `${((h - START) / (END - START)) * 100}%`
</script>

<template>
  <SlideFrame :section="2" :title="TITLES[view][0]" :kicker="TITLES[view][1]">
    <Transition name="view" mode="out-in">
    <TourPhotos v-if="view === 'tour'" key="tour" />
    <EveningPhotos v-else-if="view === 'dinner'" key="dinner" />
    <TalkPhoto v-else-if="view === 'talk'" key="talk" />
    <div v-else key="plan" class="wrap">
      <div class="facts">
        <div v-for="(f, i) in facts" :key="f.text" class="card fact" v-motion :initial="{ opacity: 0, y: 16 }" :enter="{ opacity: 1, y: 0, transition: { delay: i * 90, duration: 450 } }">
          <span class="big">{{ f.big }}</span><span>{{ f.text }}</span>
        </div>
      </div>

      <div class="card timeline">
        <div class="hours">
          <span v-for="h in END - START + 1" :key="h" :style="{ left: pos(START + h - 1) }">{{ START + h - 1 }}:00</span>
        </div>
        <div v-for="(d, di) in days" :key="d.name" class="day">
          <div class="day-name"><b>{{ d.name }}</b><span>{{ d.date }}</span></div>
          <div class="track">
            <div class="lunch" :style="{ left: pos(12.5), width: `calc(${pos(14)} - ${pos(12.5)})` }" />
            <div
              v-for="(b, bi) in d.blocks" :key="b.label + bi" class="block" :class="[b.kind, { dim: focus !== 'all' && !inFocus(b), hl: focus !== 'all' && inFocus(b) }]"
              :style="{ left: pos(b.from), width: `calc(${pos(b.to)} - ${pos(b.from)} - 4px)` }"
              v-motion :initial="{ opacity: 0, scaleX: 0.6 }" :enter="{ opacity: 1, scaleX: 1, transition: { delay: 400 + di * 500 + bi * 110, duration: 450 } }"
            >
              {{ b.label }}
            </div>
          </div>
        </div>
        <div class="legend">
          <span v-for="l in legend" :key="l.kind"><i :class="l.kind" />{{ l.label }}</span>
          <span class="note">Livestream, keine Aufzeichnung</span>
        </div>
      </div>
    </div>
    </Transition>
  </SlideFrame>
</template>

<style scoped>
.wrap { height: 100%; display: flex; flex-direction: column; gap: 18px; }
.facts { display: grid; grid-template-columns: repeat(5, 1fr); gap: 12px; }
.fact { display: flex; align-items: baseline; gap: 10px; padding: 16px 18px; font-size: 15px; font-weight: 600; color: var(--ink-2); }
.big { font-size: 38px; font-weight: 800; letter-spacing: -0.04em; color: var(--teal); }

.timeline { flex: 1; display: flex; flex-direction: column; padding: 18px 22px 16px 22px; }
.hours { position: relative; height: 20px; margin-left: 110px; }
.hours span { position: absolute; transform: translateX(-50%); font: 500 11px 'JetBrains Mono', monospace; color: var(--muted); }
.day { flex: 1; display: flex; align-items: center; }
.day-name { width: 110px; display: flex; flex-direction: column; }
.day-name b { font-size: 17px; color: var(--ink); }
.day-name span { font-size: 13px; color: var(--muted); }
.track { position: relative; flex: 1; height: 64px; border-radius: 12px; background: repeating-linear-gradient(90deg, var(--surface-2) 0 1px, transparent 1px calc(100% / 9)); }
.lunch { position: absolute; top: 0; bottom: 0; border-radius: 10px; background: repeating-linear-gradient(135deg, transparent 0 6px, rgba(100, 116, 139, 0.08) 6px 12px); }
.block {
  position: absolute;
  top: 6px;
  bottom: 6px;
  display: flex;
  align-items: center;
  padding: 0 8px 0 10px;
  border-radius: 10px;
  overflow: hidden;
  font-size: 12.5px;
  font-weight: 600;
  line-height: 1.15;
  transform-origin: left;
}
.block.key, .legend i.key { background: var(--amber-soft); color: #9a6400; box-shadow: inset 3px 0 0 var(--amber); }
.block.site, .legend i.site { background: var(--teal-soft); color: #006b5a; box-shadow: inset 3px 0 0 var(--teal); }
.block.dev, .legend i.dev { background: var(--violet-soft); color: #4b32d6; box-shadow: inset 3px 0 0 var(--violet); }
.block.ops, .legend i.ops { background: var(--accent-soft); color: #1d4fd8; box-shadow: inset 3px 0 0 var(--accent); }
.block.dinner, .legend i.dinner { background: var(--rose-soft); color: #b42318; box-shadow: inset 3px 0 0 var(--rose); }
.block { transition: filter 0.4s, box-shadow 0.4s; }
.block.dim { filter: grayscale(1) opacity(0.25); }
.block.hl { box-shadow: inset 3px 0 0 currentColor, 0 0 0 2px currentColor, 0 8px 18px rgba(15, 23, 42, 0.18); z-index: 1; }
.view-enter-active, .view-leave-active { transition: opacity 0.35s ease, transform 0.35s ease; }
.view-enter-from { opacity: 0; transform: scale(0.98); }
.view-leave-to { opacity: 0; transform: scale(1.01); }
.block.extra, .legend i.extra { background: var(--surface-2); color: var(--ink-2); box-shadow: inset 3px 0 0 #94a3b8; }
.legend { display: flex; flex-wrap: wrap; gap: 18px; margin-top: 10px; font-size: 13px; color: var(--muted); }
.legend span { display: inline-flex; align-items: center; gap: 7px; }
.legend i { width: 14px; height: 14px; border-radius: 4px; }
.legend .note { margin-left: auto; font-style: italic; }
</style>
