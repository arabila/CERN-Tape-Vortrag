<script setup lang="ts">
// People from the DESY storage / CTA team met at the workshop, as a 2×2 grid.
// Each card has a small animated sketch of the person's current topic.
// Topics from their Indico talks (Karimi, Chodak), the XFEL author list and personal conversations.
type Person = {
  name: string; ini: string; site: string; role: string; now: string[]; why: string
  art: 'pg' | 'vtl' | 'rucio' | 'site'; color: string; soft: string; contact?: boolean; link?: string; src: string
}

const people: Person[] = [
  {
    name: 'Mwai Karimi', ini: 'MK', site: 'Hamburg', art: 'pg', color: 'var(--accent)', soft: 'var(--accent-soft)',
    role: 'Betreibt CTA bei DESY, hielt den DESY-Site-Report',
    now: ['PostgreSQL-Scheduler testen', 'Catalogue-Tabelle per Hash-Partitionierung beschleunigen', 'LTO-10 eventuell überspringen'],
    why: 'Kennt den CTA-Betrieb mit dCache aus erster Hand',
    src: 'Vortrag „CTA at DESY“',
  },
  {
    name: 'Jacek Chodak', ini: 'JC', site: 'Hamburg', art: 'vtl', color: 'var(--violet)', soft: 'var(--violet-soft)',
    role: 'Entwickelt an mhVTL, einer virtuellen Tape-Library',
    now: ['mhVTL vom eigenen Kernel-Modul in den User Space (TCMU) geholt', 'Schreiben +73 %, Lesen +112 %, auch über iSCSI'],
    why: 'CTA testen ganz ohne echte Bandhardware',
    src: 'Vortrag „Off the Kernel, Through the Ring“',
  },
  {
    name: 'Peter van der Reest', ini: 'PR', site: 'Hamburg', art: 'rucio', color: 'var(--amber)', soft: 'var(--amber-soft)',
    role: 'Langjährig im DESY-Storage-Team für dCache und CTA',
    now: ['Beschäftigt sich künftig mit Rucio als Management-Werkzeug', 'Mitautor der XFEL-Anbindung an CTA'],
    why: 'Rucio über dCache und CTA',
    link: 'in/peter-van-der-reest',
    src: 'Gespräch, XFEL-Autorenliste',
  },
  {
    name: 'Tino Reichert', ini: 'TR', site: 'Zeuthen', art: 'site', color: 'var(--teal)', soft: 'var(--teal-soft)',
    role: 'Betreut allein die kleinere CTA-Instanz am Standort Zeuthen',
    now: ['Eine kleine CTA-Instanz für einen Standort', 'Betrieb durch eine Person'],
    why: 'Am nächsten an unserer Größenordnung',
    contact: true, link: 'in/mcmilk',
    src: 'Gespräch',
  },
]
</script>

<template>
  <SlideFrame :section="2" title="Kontakte im DESY-Team" kicker="Workshop & Community · Ansprechpartner für CTA und dCache">
    <div class="grid">
      <article
        v-for="(p, i) in people" :key="p.name" class="card person" :class="{ contact: p.contact }"
        :style="{ '--c': p.color, '--soft': p.soft }"
        v-motion :initial="{ opacity: 0, y: 16 }" :enter="{ opacity: 1, y: 0, transition: { delay: i * 110, duration: 450 } }"
      >
        <!-- animated sketch of the current topic -->
        <div class="art">
          <svg v-if="p.art === 'pg'" viewBox="0 0 160 170">
            <g class="cyl">
              <path d="M30 34 a30 9 0 0 0 60 0 v34 a30 9 0 0 1 -60 0 z" />
              <ellipse cx="60" cy="34" rx="30" ry="9" />
            </g>
            <text x="60" y="62" text-anchor="middle" class="lbl">CTA</text>
            <text x="60" y="94" text-anchor="middle" class="sm">archive_file</text>
            <g v-for="k in 4" :key="k">
              <rect :x="104" :y="16 + (k - 1) * 38" width="44" height="26" rx="6" class="part" />
              <text :x="126" :y="33 + (k - 1) * 38" text-anchor="middle" class="sm">P{{ k }}</text>
            </g>
            <circle v-for="k in 4" :key="`d${k}`" r="4" class="dot" :style="{ animationDelay: `${(k - 1) * 0.55}s`, offsetPath: `path('M90 52 C 98 52, 96 ${29 + (k - 1) * 38}, 104 ${29 + (k - 1) * 38}')` }" />
            <text x="60" y="140" text-anchor="middle" class="sm">PostgreSQL</text>
            <text x="60" y="156" text-anchor="middle" class="sm">Partitionen</text>
          </svg>

          <svg v-else-if="p.art === 'vtl'" viewBox="0 0 160 170">
            <rect x="12" y="20" width="136" height="92" rx="14" class="vbox" />
            <text x="80" y="38" text-anchor="middle" class="sm">virtuelles Laufwerk</text>
            <g class="reel r1"><circle cx="52" cy="72" r="22" /><path d="M52 52 v40 M32 72 h40" /></g>
            <g class="reel r2"><circle cx="108" cy="72" r="22" /><path d="M108 52 v40 M88 72 h40" /></g>
            <path d="M52 94 L108 94" class="tape" />
            <rect x="34" y="128" width="92" height="22" rx="11" class="chip" />
            <text x="80" y="143" text-anchor="middle" class="sm strong">User Space</text>
          </svg>

          <svg v-else-if="p.art === 'rucio'" viewBox="0 0 160 170">
            <path v-for="(d, k) in ['M80 82 L32 34', 'M80 82 L128 34', 'M80 82 L32 132', 'M80 82 L128 132']" :key="k" :d="d" class="edge" />
            <circle v-for="(d, k) in ['M80 82 L32 34', 'M128 34 L80 82', 'M80 82 L32 132', 'M128 132 L80 82']" :key="`m${k}`" r="4" class="dot" :style="{ offsetPath: `path('${d}')`, animationDelay: `${k * 0.45}s` }" />
            <g class="node hub"><circle cx="80" cy="82" r="24" /><text x="80" y="86" text-anchor="middle" class="lbl">Rucio</text></g>
            <g class="node"><circle cx="32" cy="34" r="17" /><text x="32" y="38" text-anchor="middle" class="sm strong">dCache</text></g>
            <g class="node"><circle cx="128" cy="34" r="17" /><text x="128" y="38" text-anchor="middle" class="sm strong">CTA</text></g>
            <g class="node"><circle cx="32" cy="132" r="17" /><text x="32" y="136" text-anchor="middle" class="sm strong">XFEL</text></g>
            <g class="node"><circle cx="128" cy="132" r="17" /><text x="128" y="136" text-anchor="middle" class="sm strong">Sites</text></g>
          </svg>

          <svg v-else viewBox="0 0 160 170">
            <rect x="34" y="14" width="92" height="132" rx="10" class="rack" />
            <g v-for="r in 5" :key="r">
              <rect v-for="c in 4" :key="c" :x="42 + (c - 1) * 20" :y="24 + (r - 1) * 18" width="16" height="12" rx="3" class="slot" :style="{ animationDelay: `${((r * 4 + c) % 7) * 0.4}s` }" />
            </g>
            <rect x="46" y="118" width="68" height="18" rx="5" class="drive" />
            <rect x="0" y="0" width="14" height="10" rx="2" class="cart" />
            <text x="80" y="164" text-anchor="middle" class="sm">eine Instanz, ein Admin</text>
          </svg>
        </div>

        <div class="body">
          <header>
            <div class="avatar">{{ p.ini }}</div>
            <div class="who">
              <b>{{ p.name }}</b>
              <span><img src="/logos/desy.png" alt="DESY">{{ p.site }}<template v-if="p.link"> · <em>linkedin.com/{{ p.link }}</em></template></span>
            </div>
            <span v-if="p.contact" class="badge">Im Kontakt</span>
          </header>
          <p class="role">{{ p.role }}</p>
          <ul><li v-for="n in p.now" :key="n">{{ n }}</li></ul>
          <div class="foot">
            <div class="why"><b>Für uns:</b> {{ p.why }}</div>
            <span class="src">{{ p.src }}</span>
          </div>
        </div>
      </article>
    </div>
  </SlideFrame>
</template>

<style scoped>
.grid { height: 100%; display: grid; grid-template-columns: 1fr 1fr; grid-template-rows: 1fr 1fr; gap: 14px; }
.person { display: grid; grid-template-columns: 150px 1fr; gap: 16px; padding: 14px 18px 12px 12px; min-height: 0; }
.person.contact { border-color: var(--teal); box-shadow: 0 0 0 2px var(--teal-soft), var(--shadow); }

.art { display: flex; align-items: center; justify-content: center; border-radius: 14px; background: var(--soft); }
.art svg { width: 100%; height: 100%; max-height: 190px; }

.body { display: flex; flex-direction: column; gap: 7px; min-width: 0; }
header { display: flex; align-items: center; gap: 10px; }
.avatar { flex: none; display: grid; place-items: center; width: 40px; height: 40px; border-radius: 12px; font: 700 15px 'JetBrains Mono', monospace; color: var(--c); background: var(--soft); }
.who { display: flex; flex-direction: column; gap: 2px; min-width: 0; flex: 1; }
.who b { font-size: 18px; line-height: 1.15; color: var(--ink); }
.who span { display: inline-flex; align-items: center; gap: 5px; font-size: 12px; color: var(--muted); white-space: nowrap; }
.who img { height: 13px; }
.who em { font: normal 500 11.5px 'JetBrains Mono', monospace; color: var(--c); }
.badge { flex: none; align-self: flex-start; padding: 4px 10px; border-radius: 999px; font-size: 11.5px; font-weight: 700; color: #fff; background: var(--teal); }
.role { margin: 0; font-size: 14px; line-height: 1.38; color: var(--ink); font-weight: 600; opacity: 1; }
ul { margin: 0; padding: 0; list-style: none; display: grid; gap: 3px; }
li { position: relative; padding-left: 12px; font-size: 13.5px; line-height: 1.36; color: var(--ink-2); }
li::before { content: ''; position: absolute; left: 0; top: 7px; width: 5px; height: 5px; border-radius: 50%; background: var(--c); }
.foot { margin-top: auto; display: flex; align-items: flex-end; justify-content: space-between; gap: 10px; }
.why { padding: 6px 10px; border-radius: 9px; font-size: 13px; line-height: 1.3; color: var(--ink-2); background: var(--surface-2); }
.why b { color: var(--ink); }
.src { flex: none; max-width: 40%; text-align: right; font: 500 10.5px 'JetBrains Mono', monospace; color: #94a3b8; }

/* sketches */
.lbl { font: 700 14px Inter, sans-serif; fill: var(--ink); }
.sm { font: 600 10.5px 'JetBrains Mono', monospace; fill: var(--muted); }
.sm.strong { fill: var(--ink-2); }
.cyl path, .cyl ellipse { fill: #fff; stroke: var(--c); stroke-width: 2; }
.part { fill: #fff; stroke: var(--c); stroke-width: 1.5; }
.dot { fill: var(--c); offset-rotate: 0deg; animation: travel 2.2s ease-in-out infinite; }
@keyframes travel { from { offset-distance: 0%; opacity: 0; } 15% { opacity: 1; } 85% { opacity: 1; } to { offset-distance: 100%; opacity: 0; } }

.vbox { fill: #fff; stroke: var(--c); stroke-width: 2; stroke-dasharray: 6 5; }
.reel circle { fill: var(--soft); stroke: var(--c); stroke-width: 2; }
.reel path { stroke: var(--c); stroke-width: 2; }
.reel { transform-box: fill-box; transform-origin: center; animation: spin 2.4s linear infinite; }
.tape { stroke: var(--ink-2); stroke-width: 2; }
.chip { fill: #fff; stroke: var(--c); stroke-width: 1.5; }
@keyframes spin { to { transform: rotate(360deg); } }

.edge { stroke: var(--c); stroke-width: 2; stroke-dasharray: 4 4; opacity: 0.6; }
.node circle { fill: #fff; stroke: var(--c); stroke-width: 1.8; }
.node.hub circle { fill: var(--c); }
.node.hub text { fill: #fff; font-size: 12px; }

.rack { fill: #fff; stroke: var(--c); stroke-width: 2; }
.slot { fill: var(--soft); stroke: var(--c); stroke-width: 1; animation: blink 2.8s ease-in-out infinite; }
@keyframes blink { 0%, 100% { fill: var(--soft); } 50% { fill: var(--c); } }
.drive { fill: var(--ink-2); }
.cart { fill: var(--c); offset-path: path('M112 30 L112 110 L80 120 L112 110 L112 30'); offset-rotate: 0deg; animation: shuttle 4s ease-in-out infinite; }
@keyframes shuttle { from { offset-distance: 0%; } to { offset-distance: 100%; } }
</style>
