<script setup lang="ts">
// Site reports, part 2: sites that use CTA indirectly, evaluate it or chose another system,
// plus the themes that came up across several reports. Facts from the Indico slides.
type Tone = 'blue' | 'amber' | 'gray' | 'none'
type Site = { name: string; place: string; logo?: string; mono?: string; logoH?: number; tone: Tone; status: string; facts: string[] }

const sites: Site[] = [
  {
    name: 'European XFEL',
    place: 'Hamburg · DE',
    logo: '/logos/xfel.png',
    logoH: 60,
    tone: 'blue',
    status: 'Archiviert über CTA bei DESY',
    facts: ['≈ 30 PB Rohdaten pro Jahr, Spitzen 1–2 PB am Tag', 'Wunsch: Experiment zusammen auf Band, eigene Lese-Laufwerke', 'Wunsch: Rohdaten (RAW) zuerst sichern, RED ist neu berechenbar'],
  },
  {
    name: 'TRIUMF',
    place: 'Vancouver · CA',
    logo: '/logos/triumf.png',
    logoH: 38,
    tone: 'amber',
    status: 'Prüft CTA, noch nicht entschieden',
    facts: ['ATLAS-Tier-1 in Kanada', 'Alte Bänder im GNU-tar-Format (Linux-Standard) kann CTA jetzt direkt lesen, ohne Umkopieren', 'Neue Daten schreibt CTA weiter im eigenen Format'],
  },
  {
    name: 'SURF',
    place: 'Amsterdam · NL',
    logo: '/logos/surf.png',
    logoH: 48,
    tone: 'gray',
    status: 'Anderer Weg: ScoutAM (Versity)',
    facts: ['Altes Band-System DMF nach 30 Jahren abgelöst, 90 PB übernommen', 'CTA wurde geprüft, ScoutAM gewann: läuft auf der vorhandenen Hardware'],
  },
  {
    name: 'IHEP / HEPS',
    place: 'Beijing · CN',
    mono: 'IHEP',
    tone: 'none',
    status: 'Nutzt EOS + CTA, ohne Folien',
    facts: ['Institut für Hochenergiephysik, Peking', 'Zwei Vorträge im Programm: CTA bei IHEP und bei HEPS (neue Lichtquelle)'],
  },
]

const themes = [
  { title: 'Klima im Bandraum', text: 'Feuchte führte bei RAL zu Datenverlust, PIC braucht einen eigenen Klimaraum.', color: 'var(--rose)', soft: 'var(--rose-soft)' },
  { title: 'LTO-10 kommt an', text: 'Fermilab zufrieden, bei PIC noch Firmware- und Labeling-Probleme.', color: 'var(--violet)', soft: 'var(--violet-soft)' },
  { title: 'Altsysteme verschwinden', text: 'Enstore ist weg (Fermilab, PIC), DMF auch (SURF).', color: 'var(--accent)', soft: 'var(--accent-soft)' },
  { title: 'Lesen wird wichtiger', text: 'Daten so ablegen, dass sie schnell zurückkommen (XFEL, Archive Metadata).', color: 'var(--teal)', soft: 'var(--teal-soft)' },
]
</script>

<template>
  <SlideFrame :section="2" title="Weitere Berichte und gemeinsame Themen" kicker="Workshop & Community · Site Reports 2 / 2">
    <div class="wrap">
      <div class="grid">
        <article
          v-for="(s, i) in sites" :key="s.name" class="card site"
          v-motion :initial="{ opacity: 0, y: 16 }" :enter="{ opacity: 1, y: 0, transition: { delay: i * 90, duration: 450 } }"
        >
          <div class="logo">
            <img v-if="s.logo" :src="s.logo" :alt="`${s.name} Logo`" :style="{ height: `${s.logoH}px` }">
            <span v-else class="mono-logo">{{ s.mono }}</span>
          </div>
          <div class="who">
            <b>{{ s.name }}</b>
            <span>{{ s.place }}</span>
          </div>
          <div class="status" :class="s.tone"><i />{{ s.status }}</div>
          <ul>
            <li v-for="f in s.facts" :key="f">{{ f }}</li>
          </ul>
        </article>
      </div>

      <div class="themes">
        <div class="themes-head">Was sich durch viele Berichte zieht</div>
        <div class="theme-row">
          <div
            v-for="(t, i) in themes" :key="t.title" class="card theme" :style="{ '--c': t.color, '--soft': t.soft }"
            v-motion :initial="{ opacity: 0, y: 12 }" :enter="{ opacity: 1, y: 0, transition: { delay: 450 + i * 90, duration: 400 } }"
          >
            <b>{{ t.title }}</b>
            <span>{{ t.text }}</span>
          </div>
        </div>
      </div>
    </div>
  </SlideFrame>
</template>

<style scoped>
.wrap { height: 100%; display: flex; flex-direction: column; gap: 16px; }
.grid { flex: 1; min-height: 0; display: grid; grid-template-columns: repeat(4, 1fr); gap: 14px; }
.site { display: flex; flex-direction: column; gap: 8px; padding: 16px 16px 14px; }
.logo { display: flex; align-items: center; height: 62px; flex: none; }
.logo img { max-width: 100%; object-fit: contain; }
.mono-logo {
  display: inline-flex;
  align-items: center;
  height: 40px;
  padding: 0 12px;
  border: 1.5px dashed #cbd3df;
  border-radius: 10px;
  font: 700 17px 'JetBrains Mono', monospace;
  letter-spacing: 0.06em;
  color: var(--muted);
}
.who { display: flex; align-items: baseline; justify-content: space-between; gap: 8px; }
.who b { font-size: 18px; color: var(--ink); }
.who span { font-size: 11.5px; color: var(--muted); white-space: nowrap; }
.status {
  display: inline-flex;
  align-items: center;
  gap: 7px;
  align-self: flex-start;
  padding: 4px 10px;
  border-radius: 999px;
  font-size: 12px;
  font-weight: 600;
}
.status i { flex: none; width: 7px; height: 7px; border-radius: 50%; background: currentColor; }
.status.blue { color: #1d4fd8; background: var(--accent-soft); }
.status.amber { color: #9a6400; background: var(--amber-soft); }
.status.gray { color: var(--ink-2); background: var(--surface-2); }
.status.none { color: var(--muted); background: transparent; border: 1px dashed #cbd3df; }
ul { margin: 2px 0 0; padding: 8px 0 0; border-top: 1px solid var(--line); list-style: none; display: grid; gap: 4px; }
li { position: relative; padding-left: 12px; font-size: 14px; line-height: 1.4; color: var(--ink-2); }
li::before { content: ''; position: absolute; left: 0; top: 8px; width: 5px; height: 5px; border-radius: 50%; background: #cfd6e2; }

.themes-head {
  margin-bottom: 8px;
  font: 600 11px/1 'JetBrains Mono', monospace;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--muted);
}
.theme-row { display: grid; grid-template-columns: repeat(4, 1fr); gap: 14px; }
.theme { display: flex; flex-direction: column; gap: 5px; padding: 14px 16px; background: var(--soft); border-color: transparent; box-shadow: inset 3px 0 0 var(--c); }
.theme b { font-size: 16px; color: var(--ink); }
.theme span { font-size: 13.5px; line-height: 1.38; color: var(--ink-2); }
</style>
