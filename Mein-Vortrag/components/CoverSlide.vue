<template>
  <div class="cover">
    <div class="dots-bg" />
    <div class="blob" style="width: 560px; height: 560px; right: -120px; top: -180px; background: #c9d8ff" />
    <div class="blob" style="width: 420px; height: 420px; right: 300px; bottom: -260px; background: #c6efe6" />

    <div class="cover-text">
      <div class="kicker" v-motion :initial="{ opacity: 0, y: 10 }" :enter="{ opacity: 1, y: 0, transition: { delay: 100, duration: 600 } }">
        5. CERN Tape Archive Workshop
      </div>

      <h1 v-motion :initial="{ opacity: 0, y: 24 }" :enter="{ opacity: 1, y: 0, transition: { delay: 250, duration: 700 } }">
        CTA <span class="accent">2026</span>
      </h1>

      <p class="subtitle" v-motion :initial="{ opacity: 0, y: 16 }" :enter="{ opacity: 1, y: 0, transition: { delay: 420, duration: 700 } }">
        Rückblick auf den Workshop bei DESY in Hamburg: Stand, Community, CTA v6 und Roadmap
      </p>

      <div class="meta" v-motion :initial="{ opacity: 0 }" :enter="{ opacity: 1, transition: { delay: 650, duration: 800 } }">
        <div class="chip"><span class="dot" />DESY Hamburg · Flash Seminar Room</div>
        <div class="chip">9.–10. November 2026</div>
      </div>
      <div class="author" v-motion :initial="{ opacity: 0 }" :enter="{ opacity: 1, transition: { delay: 800, duration: 800 } }">
        Amine Karabila
      </div>
    </div>

    <div class="cover-art" v-motion :initial="{ opacity: 0, y: 20, scale: 0.96 }" :enter="{ opacity: 1, y: 0, scale: 1, transition: { delay: 300, duration: 900 } }">
      <TapeCartridge />
    </div>
  </div>
</template>

<style scoped>
.cover {
  position: absolute;
  inset: 0;
  display: grid;
  grid-template-columns: 1.15fr 1fr;
  align-items: center;
  gap: 48px;
  padding: 0 88px;
  overflow: hidden;
  background: var(--bg);
}
.cover-text, .cover-art { position: relative; }

h1 {
  margin: 22px 0 0;
  font-size: 120px;
  font-weight: 800;
  line-height: 0.95;
  letter-spacing: -0.05em;
  color: var(--ink);
}
h1 .accent {
  background: linear-gradient(90deg, var(--accent), var(--teal));
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}
.subtitle {
  margin-top: 24px;
  max-width: 30ch;
  font-size: 23px;
  line-height: 1.45;
  color: var(--ink-2);
  opacity: 1;
}
.meta { display: flex; flex-wrap: wrap; gap: 10px; margin-top: 36px; }
.chip {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 8px 14px;
  border: 1px solid var(--line);
  border-radius: 999px;
  background: var(--surface);
  font-size: 14px;
  font-weight: 500;
  color: var(--ink-2);
}
.chip .dot { width: 7px; height: 7px; border-radius: 50%; background: var(--teal); }
.author { margin-top: 22px; font-size: 15px; font-weight: 600; color: var(--muted); }
</style>
