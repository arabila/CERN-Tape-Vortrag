<script setup lang="ts">
// Site reports, part 1: the four sites that run CTA in production.
// All facts come from the site report slides on Indico (indico.desy.de/event/52689).
type Site = {
  name: string
  place: string
  logo: string
  logoH: number
  since: string
  stats: { v: string; l: string }[]
  good: string[]
  issues: string[]
  next: string[]
}

const sites: Site[] = [
  {
    name: 'DESY',
    place: 'Hamburg · DE',
    logo: '/logos/desy.png',
    logoH: 58,
    since: '3. Jahr in Produktion',
    stats: [{ v: '160 PB', l: 'mit CTA geschrieben' }, { v: '45', l: 'Laufwerke' }],
    good: ['Läuft stabil, kaum Eingriffe nötig', '6 dCache-Instanzen, 15 VOs'],
    issues: ['Keine größeren Probleme gemeldet'],
    next: ['PostgreSQL-Scheduler testen', 'LTO-10 eventuell überspringen'],
  },
  {
    name: 'RAL · STFC',
    place: 'Oxfordshire · UK',
    logo: '/logos/stfc.png',
    logoH: 44,
    since: 'In Produktion („Antares“)',
    stats: [{ v: '300 PB', l: 'in 4 Jahren archiviert' }, { v: '100', l: 'Laufwerke' }],
    good: ['Library um 83\u00a0PB erweitert, mit\u00a08\u00a0TS1170-Laufwerken', 'Neuer EOS-Disk-Buffer'],
    issues: ['Hitzewelle Juni 2026: 59 Dateien unlesbar, 9 verloren'],
    next: ['WLCG-Tokens und PostgreSQL testen', 'Entfeuchtung im Rechenzentrum'],
  },
  {
    name: 'Fermilab',
    place: 'Illinois · US',
    logo: '/logos/fermilab.png',
    logoH: 56,
    since: 'Seit Mai 2025 (von Enstore)',
    stats: [{ v: '≈ 240', l: 'Laufwerke' }, { v: '7', l: 'Libraries' }],
    good: ['„Tag und Nacht“ gegenüber Enstore', 'LTO-10 über SAS läuft gut'],
    issues: ['Fällt die CTA-Datenbank kurz aus, stürzen alle Tape-Server ab', 'Experimente schreiben viele kleine Dateien, das bremst die Laufwerke'],
    next: ['Repack in LS3: LTO-7/8 → LTO-9/10', 'Selbst gebaut: Logbuch-Datenbank für jeden Mount und Transfer'],
  },
  {
    name: 'PIC',
    place: 'Barcelona · ES',
    logo: '/logos/pic.png',
    logoH: 58,
    since: 'Seit Sept. 2026 (von Enstore)',
    stats: [{ v: '108 PB', l: 'auf Band' }, { v: '29,8 Mio.', l: 'Dateien migriert' }],
    good: ['Migration in 2 Tagen', 'Schneller, weniger Remounts als Enstore'],
    issues: ['LTO-10: Fehler beim Labeln und Lesen', 'Klimaanlage im neuen Bandraum'],
    next: ['9 weitere LTO-10-Laufwerke (2027)', 'LTO-8 → LTO-10, später v6'],
  },
]

const rows = [
  { key: 'good', label: 'Läuft gut', cls: 'good' },
  { key: 'issues', label: 'Baustellen', cls: 'issue' },
  { key: 'next', label: 'Als Nächstes', cls: 'next' },
] as const
</script>

<template>
  <SlideFrame :section="2" title="CTA im Einsatz: Stand der Sites" kicker="Workshop & Community · Site Reports 1 / 2">
    <div class="grid">
      <article
        v-for="(s, i) in sites" :key="s.name" class="card site"
        v-motion :initial="{ opacity: 0, y: 16 }" :enter="{ opacity: 1, y: 0, transition: { delay: i * 90, duration: 450 } }"
      >
        <div class="logo"><img :src="s.logo" :alt="`${s.name} Logo`" :style="{ height: `${s.logoH}px` }"></div>
        <div class="who">
          <b>{{ s.name }}</b>
          <span>{{ s.place }}</span>
        </div>
        <div class="status"><i />{{ s.since }}</div>
        <div class="stats">
          <div v-for="st in s.stats" :key="st.l" class="stat">
            <b>{{ st.v }}</b>
            <span>{{ st.l }}</span>
          </div>
        </div>
        <section v-for="r in rows" :key="r.key" class="sec" :class="r.cls">
          <h4>{{ r.label }}</h4>
          <ul>
            <li v-for="t in s[r.key]" :key="t">{{ t }}</li>
          </ul>
        </section>
      </article>
    </div>
  </SlideFrame>
</template>

<style scoped>
.grid {
  height: 100%;
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: 78px auto auto auto 1fr 1fr 1fr;
  column-gap: 14px;
}
.site {
  grid-row: 1 / -1;
  display: grid;
  grid-template-rows: subgrid;
  row-gap: 8px;
  padding: 16px 16px 14px;
}
.logo { display: flex; align-items: center; height: 62px; }
.logo img { max-width: 100%; object-fit: contain; }
.who { display: flex; align-items: baseline; justify-content: space-between; gap: 8px; }
.who b { font-size: 18px; color: var(--ink); }
.who span { font-size: 11.5px; color: var(--muted); white-space: nowrap; }
.status {
  display: inline-flex;
  align-items: center;
  gap: 7px;
  justify-self: start;
  padding: 4px 10px;
  border-radius: 999px;
  font-size: 12px;
  font-weight: 600;
  color: #0f7a3a;
  background: #e3f6ea;
}
.status i { width: 7px; height: 7px; border-radius: 50%; background: var(--green); }
.stats { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; padding: 8px 0; border-top: 1px solid var(--line); border-bottom: 1px solid var(--line); }
.stat b { display: block; font-size: 20px; font-weight: 700; color: var(--ink); letter-spacing: -0.01em; }
.stat span { display: block; font-size: 11.5px; line-height: 1.3; color: var(--muted); }
.sec h4 {
  margin: 0 0 4px;
  font: 600 10.5px/1 'JetBrains Mono', monospace;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}
.sec.good h4 { color: var(--teal); }
.sec.issue h4 { color: var(--rose); }
.sec.next h4 { color: var(--accent); }
ul { margin: 0; padding: 0; list-style: none; display: grid; gap: 3px; }
li { position: relative; padding-left: 12px; font-size: 13.5px; line-height: 1.35; color: var(--ink-2); }
li::before { content: ''; position: absolute; left: 0; top: 7px; width: 5px; height: 5px; border-radius: 50%; background: #cfd6e2; }
.good li::before { background: var(--teal); }
.issue li::before { background: var(--rose); }
.next li::before { background: var(--accent); }
</style>
