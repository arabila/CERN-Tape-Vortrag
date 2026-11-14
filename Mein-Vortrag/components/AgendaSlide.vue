<script setup lang="ts">
const parts = [
  { title: 'CTA heute', text: 'Was CTA ist, wie es aufgebaut ist und wo es nach Run-3 steht', tags: ['1,5 EB auf Band', 'Architektur'], color: 'var(--accent)', soft: 'var(--accent-soft)' },
  { title: 'Workshop & Community', text: 'Der 5. CTA Workshop bei DESY und was andere Institute berichtet haben', tags: ['Site Reports', 'Trends'], color: 'var(--teal)', soft: 'var(--teal-soft)' },
  { title: 'Neu in CTA v6', text: 'Das größte Release seit Jahren: neue Komponenten, einheitlicher Betrieb', tags: ['gRPC', 'PostgreSQL', 'TOML'], color: 'var(--violet)', soft: 'var(--violet-soft)' },
  { title: 'Roadmap', text: 'Data Challenge 2027, Long Shutdown 3 und der Weg zum HL-LHC', tags: ['DC27', 'Run-4'], color: 'var(--amber)', soft: 'var(--amber-soft)' },
  { title: 'IBM Diamondback', text: 'Die Rack-Library im Langzeittest am CERN: Mechanik, Mount-Zeiten, Kosten', tags: ['Rack-Library', 'Mount-Tests'], color: 'var(--rose)', soft: 'var(--rose-soft)' },
]
</script>

<template>
  <div class="agenda">
    <header>
      <div class="kicker">Agenda</div>
      <h2>Worum es heute geht</h2>
    </header>
    <div class="grid">
      <article
        v-for="(p, i) in parts" :key="p.title" class="card part"
        :style="{ '--c': p.color, '--soft': p.soft }"
        v-motion
        :initial="{ opacity: 0, y: 30 }"
        :enter="{ opacity: 1, y: 0, transition: { delay: 150 + i * 120, duration: 600 } }"
      >
        <div class="num">{{ String(i + 1).padStart(2, '0') }}</div>
        <h3>{{ p.title }}</h3>
        <p>{{ p.text }}</p>
        <div class="tags">
          <span v-for="t in p.tags" :key="t">{{ t }}</span>
        </div>
        <div class="bar" />
      </article>
    </div>
  </div>
</template>

<style scoped>
.agenda {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  padding: 56px 64px;
  background: var(--bg);
}
h2 { margin: 12px 0 0; font-size: 46px; font-weight: 800; letter-spacing: -0.035em; color: var(--ink); }
.grid { flex: 1; display: grid; grid-template-columns: repeat(5, 1fr); gap: 16px; margin-top: 40px; }
.part {
  position: relative;
  display: flex;
  flex-direction: column;
  padding: 26px 24px;
  overflow: hidden;
}
.num {
  display: grid;
  place-items: center;
  width: 52px;
  height: 52px;
  border-radius: 14px;
  font: 700 20px 'JetBrains Mono', monospace;
  color: var(--c);
  background: var(--soft);
}
h3 { margin: 26px 0 0; font-size: 26px; font-weight: 750; letter-spacing: -0.02em; color: var(--ink); }
p { margin-top: 12px; font-size: 16px; line-height: 1.5; color: var(--muted); opacity: 1; }
.tags { display: flex; flex-wrap: wrap; gap: 6px; margin-top: auto; padding-top: 20px; }
.tags span {
  padding: 4px 10px;
  border-radius: 999px;
  font: 500 12px 'JetBrains Mono', monospace;
  color: var(--c);
  background: var(--soft);
}
.bar { position: absolute; left: 0; right: 0; bottom: 0; height: 5px; background: var(--c); }
</style>
