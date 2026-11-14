<script setup lang="ts">
import { computed } from 'vue'

// Our own next steps after the workshop; each click reveals the next step on the path.
// Every card has a small looping illustration that only animates once the step is revealed.
const props = defineProps<{ step: number }>()

const steps = [
  { key: 'build', when: 'Jetzt', title: 'Echte CTA-Instanz aufbauen', text: 'Vom Tutorial-Stand zu einem Setup für mehrere Experimente', who: 'Vorschlag: Tino', color: 'var(--accent)' },
  { key: 'v6', when: 'Sobald v6 da ist', title: 'Neuaufbau mit CTA v6', text: 'Von Grund auf neu, dann echte Data Challenges', who: 'Rat: Julio', color: 'var(--violet)' },
  { key: 'visit', when: 'In ein paar Monaten', title: 'Mini-Workshop bei uns', text: 'Besuch aus der Archive-Gruppe des CERN', who: 'Michael Davis (+ Julio)', color: 'var(--amber)' },
]
const shown = computed(() => Math.min(Math.max(props.step, 0) + 1, steps.length))
const fill = computed(() => `${((shown.value - 1) / (steps.length - 1)) * 100}%`)

const arc = 'M48 112 Q160 0 268 112'
</script>

<template>
  <SlideFrame :section="0" title="Wie geht es bei uns weiter?" kicker="Ausblick">
    <div class="stops">
      <div class="rail"><div class="rail-fill" :style="{ width: fill }" /></div>
      <div v-for="(s, i) in steps" :key="s.key" class="stop" :class="{ on: i < shown, now: i === shown - 1 }" :style="{ '--c': s.color }">
        <div class="when">{{ s.when }}</div>
        <div class="node"><span class="mono">{{ i + 1 }}</span></div>

        <div class="card body">
          <div class="pic">
            <!-- 1: cta-admin fills a library with one tape pool per experiment -->
            <svg v-if="s.key === 'build'" viewBox="0 0 320 160">
              <rect x="8" y="34" width="128" height="96" rx="9" class="term" />
              <circle cx="20" cy="46" r="3" fill="#ff5f57" /><circle cx="30" cy="46" r="3" fill="#febc2e" /><circle cx="40" cy="46" r="3" fill="#28c840" />
              <text x="18" y="70" class="t-prompt">$ cta-admin</text>
              <text x="18" y="88" class="t-line l1">vo add …</text>
              <text x="18" y="104" class="t-line l2">tp add …</text>
              <text x="18" y="120" class="t-line l3">sc add …</text>
              <rect x="76" y="111" width="6" height="11" class="cursor" />
              <path d="M142 82 H164" class="flow" />
              <rect x="170" y="14" width="142" height="134" rx="11" class="lib" />
              <g v-for="(t, k) in [['Exp. A', 'var(--teal)'], ['Exp. B', 'var(--violet)'], ['Exp. C', 'var(--amber)']]" :key="k">
                <rect x="182" :y="28 + k * 40" width="118" height="28" rx="6" class="slot" />
                <g class="tape" :style="{ animationDelay: `${0.3 + k * 0.45}s` }">
                  <rect x="188" :y="33 + k * 40" width="74" height="18" rx="4" :fill="t[1]" />
                  <text x="225" :y="45.5 + k * 40" text-anchor="middle" class="t-tape">{{ t[0] }}</text>
                </g>
              </g>
            </svg>

            <!-- 2: v5 falls apart, v6 is stacked up, then a data challenge ramps up -->
            <svg v-else-if="s.key === 'v6'" viewBox="0 0 320 160">
              <g class="old">
                <rect x="14" y="88" width="70" height="46" rx="7" class="v5" />
                <text x="49" y="116" text-anchor="middle" class="t-v5">v5</text>
              </g>
              <path d="M94 111 H112" class="flow" />
              <g v-for="(b, k) in ['Tape Daemon', 'PostgreSQL', 'gRPC']" :key="b" class="brick" :style="{ animationDelay: `${0.5 + k * 0.35}s` }">
                <rect x="120" :y="112 - k * 32" width="104" height="28" rx="6" :style="{ fill: `color-mix(in srgb, var(--violet) ${100 - k * 22}%, white)` }" />
                <text x="172" :y="130.5 - k * 32" text-anchor="middle" class="t-brick">{{ b }}</text>
              </g>
              <g class="badge">
                <circle cx="224" cy="40" r="17" fill="var(--violet)" />
                <text x="224" y="45" text-anchor="middle" class="t-badge">v6</text>
              </g>
              <g v-for="(h, k) in [26, 44, 62, 84]" :key="k">
                <rect :x="252 + k * 15" :y="140 - h" width="10" :height="h" rx="2" class="bar" :style="{ animationDelay: `${1.9 + k * 0.15}s` }" />
              </g>
              <text x="281" y="154" text-anchor="middle" class="t-small">Data Challenge</text>
            </svg>

            <!-- 3: guests fly from CERN to us, then the mini workshop starts -->
            <svg v-else viewBox="0 0 320 160">
              <path :d="arc" class="route" />
              <circle cx="44" cy="116" r="7" fill="var(--amber)" />
              <text x="44" y="138" text-anchor="middle" class="t-place">CERN</text>
              <path d="M258 104 L274 90 L290 104 Z" fill="var(--accent)" />
              <rect x="261" y="104" width="26" height="20" rx="2" fill="var(--accent)" />
              <rect x="271" y="112" width="6" height="12" fill="var(--surface)" />
              <text x="274" y="140" text-anchor="middle" class="t-place">bei uns</text>
              <g class="plane">
                <path d="M-11 -4 L11 0 L-11 4 L-7 0 Z" fill="var(--amber)">
                  <animateMotion :path="arc" dur="3.6s" repeatCount="indefinite" rotate="auto" />
                </path>
              </g>
              <g v-for="(p, k) in [[124, 'var(--amber)'], [160, 'var(--amber)'], [196, 'var(--accent)']]" :key="k" class="person" :style="{ animationDelay: `${0.4 + k * 0.25}s` }">
                <circle :cx="p[0]" cy="116" r="8" :fill="p[1]" />
                <rect :x="Number(p[0]) - 11" y="127" width="22" height="20" rx="9" :fill="p[1]" />
              </g>
              <g class="bubble" style="animation-delay: 1.4s">
                <rect x="106" y="74" width="40" height="22" rx="8" class="bub" />
                <circle cx="117" cy="85" r="2.2" class="dot" /><circle cx="126" cy="85" r="2.2" class="dot" /><circle cx="135" cy="85" r="2.2" class="dot" />
              </g>
              <g class="bubble" style="animation-delay: 2.1s">
                <rect x="180" y="74" width="40" height="22" rx="8" class="bub" />
                <text x="200" y="89.5" text-anchor="middle" class="t-q">?!</text>
              </g>
            </svg>
          </div>

          <div class="txt">
            <b>{{ s.title }}</b>
            <p>{{ s.text }}</p>
            <span class="who">{{ s.who }}</span>
          </div>
        </div>
      </div>
    </div>
  </SlideFrame>
</template>

<style scoped>
.stops { position: relative; height: 100%; display: grid; grid-template-columns: repeat(3, 1fr); gap: 28px; }
.rail { position: absolute; left: calc(100% / 6); right: calc(100% / 6); top: 55px; height: 6px; border-radius: 3px; background: var(--line); }
.rail-fill { height: 100%; border-radius: 3px; background: linear-gradient(90deg, var(--accent), var(--violet), var(--amber)); transition: width 0.8s cubic-bezier(0.3, 0.8, 0.3, 1); }
.stop { position: relative; display: flex; flex-direction: column; align-items: center; min-height: 0; opacity: 0.25; transition: opacity 0.5s; }
.stop.on { opacity: 1; }
.when { height: 26px; font-size: 17px; font-weight: 700; color: var(--c); }
.node { display: grid; place-items: center; width: 44px; height: 44px; margin: 6px 0 16px; border-radius: 50%; background: var(--surface); border: 3px solid var(--line); transition: border-color 0.4s, transform 0.4s, box-shadow 0.4s; }
.node span { font-size: 16px; font-weight: 700; color: var(--muted); transition: color 0.4s; }
.stop.on .node { border-color: var(--c); }
.stop.on .node span { color: var(--c); }
.stop.now .node { transform: scale(1.2); box-shadow: 0 0 0 10px color-mix(in srgb, var(--c) 15%, transparent); }

.body { flex: 1; width: 100%; min-height: 0; display: flex; flex-direction: column; overflow: hidden; border-top: 4px solid var(--c); transform: translateY(18px); transition: transform 0.55s cubic-bezier(0.3, 0.8, 0.3, 1); }
.stop.on .body { transform: none; }
.pic { flex: 1; min-height: 0; display: grid; place-items: center; padding: 14px 16px 6px; background: color-mix(in srgb, var(--c) 6%, var(--surface)); }
.pic svg { width: 100%; height: 100%; max-height: 190px; overflow: visible; font-family: 'Inter', sans-serif; }
.txt { display: flex; flex-direction: column; gap: 7px; min-height: 146px; padding: 14px 20px 18px; }
.txt b { font-size: 21px; line-height: 1.2; color: var(--ink); }
.txt p { margin: 0; font-size: 15px; line-height: 1.45; color: var(--ink-2); opacity: 1; }
.who { align-self: flex-start; margin-top: 4px; padding: 4px 11px; border-radius: 999px; font-size: 12.5px; font-weight: 600; color: var(--c); background: color-mix(in srgb, var(--c) 12%, transparent); }

/* shared drawing styles */
.flow { fill: none; stroke: var(--muted); stroke-width: 2; stroke-dasharray: 4 4; stroke-linecap: round; }
.t-small { font-size: 10px; font-weight: 600; fill: var(--muted); }

/* 1: terminal + library */
.term { fill: var(--ink); }
.t-prompt { font: 600 11px 'JetBrains Mono', monospace; fill: #9fb6ff; }
.t-line { font: 500 10.5px 'JetBrains Mono', monospace; fill: #dbe3f0; opacity: 0; }
.cursor { fill: #9fb6ff; opacity: 0; }
.lib { fill: var(--surface); stroke: var(--line); stroke-width: 1.5; }
.slot { fill: var(--surface-2); }
.tape { opacity: 0; }
.t-tape { font-size: 10px; font-weight: 700; fill: #fff; }

/* 2: rebuild */
.v5 { fill: #c3cad6; }
.t-v5 { font-size: 16px; font-weight: 800; fill: #fff; }
.t-brick { font-size: 11px; font-weight: 700; fill: #fff; }
.brick, .badge { opacity: 0; }
.t-badge { font-size: 13px; font-weight: 800; fill: #fff; }
.bar { fill: var(--green); transform-box: fill-box; transform-origin: bottom; transform: scaleY(0); }
.old { transform-box: fill-box; transform-origin: center; }

/* 3: visit */
.route { fill: none; stroke: var(--amber); stroke-width: 2; stroke-dasharray: 5 6; opacity: 0.6; }
.t-place { font-size: 11px; font-weight: 700; fill: var(--ink-2); }
.person, .bubble { opacity: 0; transform-box: fill-box; transform-origin: bottom center; }
.bub { fill: var(--surface); stroke: var(--line); stroke-width: 1.5; }
.dot { fill: var(--muted); }
.t-q { font-size: 13px; font-weight: 800; fill: var(--amber); }
.stop:not(.on) .plane { opacity: 0; }

/* Animations only run once a step is revealed */
.stop.on .t-line { animation: type 5s infinite; }
.stop.on .l2 { animation-delay: 0.45s; }
.stop.on .l3 { animation-delay: 0.9s; }
.stop.on .cursor { animation: blink 0.9s steps(1) infinite; }
.stop.on .tape { animation: slide-in 5s cubic-bezier(0.3, 0.8, 0.3, 1) infinite backwards; }
.stop.on .old { animation: crumble 5s infinite; }
.stop.on .brick { animation: drop 5s cubic-bezier(0.3, 1.3, 0.5, 1) infinite backwards; }
.stop.on .badge { animation: pop 5s infinite; animation-delay: 1.6s; transform-box: fill-box; transform-origin: center; }
.stop.on .bar { animation: grow 5s cubic-bezier(0.3, 0.8, 0.3, 1) infinite backwards; }
.stop.on .person { animation: appear 3.6s infinite backwards; }
.stop.on .bubble { animation: appear 3.6s infinite backwards; }

@keyframes type { 0%, 4% { opacity: 0; } 8%, 88% { opacity: 1; } 96%, 100% { opacity: 0; } }
@keyframes blink { 0% { opacity: 1; } 50% { opacity: 0; } }
@keyframes slide-in { 0% { opacity: 0; transform: translateX(70px); } 14%, 86% { opacity: 1; transform: none; } 96%, 100% { opacity: 0; transform: none; } }
@keyframes crumble { 0%, 6% { opacity: 1; transform: none; } 10% { transform: rotate(-4deg); } 14% { transform: rotate(4deg); } 26% { opacity: 0; transform: translateY(26px) rotate(14deg); } 92% { opacity: 0; transform: none; } 100% { opacity: 1; } }
@keyframes drop { 0% { opacity: 0; transform: translateY(-50px); } 10%, 88% { opacity: 1; transform: none; } 96%, 100% { opacity: 0; } }
@keyframes pop { 0% { opacity: 0; transform: scale(0); } 8%, 66% { opacity: 1; transform: scale(1); } 74%, 100% { opacity: 0; transform: scale(1); } }
@keyframes grow { 0% { transform: scaleY(0); } 14%, 60% { transform: scaleY(1); } 68%, 100% { transform: scaleY(0); } }
@keyframes appear { 0% { opacity: 0; transform: scale(0.4); } 10%, 85% { opacity: 1; transform: scale(1); } 95%, 100% { opacity: 0; } }

@media (prefers-reduced-motion: reduce) {
  .stop.on * { animation: none !important; opacity: 1 !important; transform: none !important; }
}
</style>
