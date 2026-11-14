<script setup lang="ts">
import { computed } from 'vue'

// Milestones from the CTA Roadmap talk; each click advances the timeline by one milestone
const props = defineProps<{ step: number }>()

const milestones = [
  { when: 'Q4 2026', title: 'CTA v6', text: 'gRPC, PostgreSQL, neuer Tape Daemon. Bereit für DC27.', color: 'var(--violet)' },
  { when: 'Ende Jan. 2027', title: 'Skalierungstests', text: 'Tape-REST-API mit WLCG-Tokens, gemeinsam mit den Tier-1-Sites.', color: 'var(--accent)' },
  { when: 'Feb. 2027', title: 'Data Challenge 27', text: 'Tokens für Tape-Recall im großen Maßstab.', color: 'var(--accent)' },
  { when: '2027 – 2029', title: 'Long Shutdown 3', text: 'Scheduler neu bauen, Tape-Supply integrieren, Datenplatzierung optimieren.', color: 'var(--amber)' },
  { when: 'nach LS3', title: 'Run-4 / HL-LHC', text: '2 EB pro Jahr archivieren, 300 GB/s Durchsatz.', color: 'var(--rose)' },
]
const shown = computed(() => Math.min(props.step + 1, milestones.length))
const fill = computed(() => `${((shown.value - 1) / (milestones.length - 1)) * 100}%`)
</script>

<template>
  <SlideFrame :section="4" title="Von v6 zum HL-LHC" kicker="Roadmap · Zeitplan">
    <div class="wrap">
      <div class="drivers">
        <span>Getrieben von:</span>
        <b>LHC-Zeitplan</b><b>Entwicklung der Bandtechnologie</b><b>Anforderungen von Experimenten & WLCG</b>
      </div>

      <div class="timeline">
        <div class="rail"><div class="rail-fill" :style="{ width: fill }" /></div>
        <div class="stops">
          <div v-for="(m, i) in milestones" :key="m.title" class="stop" :class="{ on: i < shown, now: i === shown - 1 }" :style="{ '--c': m.color }">
            <div class="when mono">{{ m.when }}</div>
            <div class="node"><span /></div>
            <div class="card body">
              <b>{{ m.title }}</b>
              <p>{{ m.text }}</p>
            </div>
          </div>
        </div>
      </div>

      <div class="events">
        <div class="card ev"><span class="mono">01.10.2026</span> 2. globales CTA-Ops-Meeting, weitere folgen regelmäßig</div>
        <div class="card ev"><span class="mono">Juni 2027</span> IBM „Future of Tape“ am CERN</div>
        <div class="card ev"><span class="mono">Frühjahr 2028</span> nächster CTA Workshop (geplant)</div>
      </div>
    </div>
  </SlideFrame>
</template>

<style scoped>
.wrap { height: 100%; display: flex; flex-direction: column; gap: 18px; }
.drivers { display: flex; flex-wrap: wrap; align-items: center; gap: 8px; font-size: 14px; color: var(--muted); }
.drivers b { padding: 5px 12px; border-radius: 999px; font-weight: 600; color: var(--ink-2); background: var(--surface); border: 1px solid var(--line); }

.timeline { position: relative; flex: 1; }
.rail { position: absolute; left: 10%; right: 10%; top: 62px; height: 6px; border-radius: 3px; background: var(--line); }
.rail-fill { height: 100%; border-radius: 3px; background: linear-gradient(90deg, var(--violet), var(--accent), var(--amber), var(--rose)); transition: width 0.7s cubic-bezier(0.3, 0.8, 0.3, 1); }
.stops { position: relative; display: grid; grid-template-columns: repeat(5, 1fr); gap: 12px; height: 100%; }
.stop { display: flex; flex-direction: column; align-items: center; opacity: 0.35; transition: opacity 0.4s; }
.stop.on { opacity: 1; }
.when { height: 40px; display: flex; align-items: flex-end; font-size: 14px; font-weight: 700; color: var(--c); }
.node { display: grid; place-items: center; width: 30px; height: 30px; margin: 8px 0 14px; border-radius: 50%; background: var(--surface); border: 3px solid var(--line); transition: border-color 0.4s, transform 0.4s; }
.node span { width: 10px; height: 10px; border-radius: 50%; background: var(--line); transition: background 0.4s; }
.stop.on .node { border-color: var(--c); }
.stop.on .node span { background: var(--c); }
.stop.now .node { transform: scale(1.25); box-shadow: 0 0 0 8px color-mix(in srgb, var(--c) 15%, transparent); }
.body { width: 100%; padding: 14px 14px; border-top: 4px solid var(--c); transform: translateY(10px); transition: transform 0.45s; }
.stop.on .body { transform: none; }
.body b { font-size: 17px; color: var(--ink); }
.body p { margin-top: 6px; font-size: 13.5px; line-height: 1.45; color: var(--ink-2); opacity: 1; }

.events { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; }
.ev { padding: 10px 14px; font-size: 13.5px; color: var(--ink-2); }
.ev span { display: block; margin-bottom: 2px; font-size: 12px; font-weight: 700; color: var(--accent); }
</style>
