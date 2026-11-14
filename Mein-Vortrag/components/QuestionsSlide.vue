<template>
  <div class="qa">
    <div class="dots-bg" />
    <div class="blob" style="width: 520px; height: 520px; right: -120px; top: -160px; background: #c9d8ff" />
    <div class="blob" style="width: 380px; height: 380px; left: -140px; bottom: -200px; background: #c6efe6" />

    <div class="qa-left">
      <h1 v-motion :initial="{ opacity: 0, y: 24 }" :enter="{ opacity: 1, y: 0, transition: { delay: 200, duration: 700 } }">
        Danke für die <span class="accent">Aufmerksamkeit</span>
      </h1>

      <div class="meta" v-motion :initial="{ opacity: 0 }" :enter="{ opacity: 1, transition: { delay: 550, duration: 800 } }">
        <div class="chip"><span class="dot" />CTA Workshop 2026 · DESY Hamburg</div>
        <div class="chip mono">indico.desy.de/event/52689</div>
      </div>
      <div class="author" v-motion :initial="{ opacity: 0 }" :enter="{ opacity: 1, transition: { delay: 700, duration: 800 } }">
        Amine Karabila
      </div>
    </div>

    <div class="qa-right" v-motion :initial="{ opacity: 0, y: 20, scale: 0.96 }" :enter="{ opacity: 1, y: 0, scale: 1, transition: { delay: 300, duration: 900 } }">
      <TapeLibrary />
    </div>
  </div>
</template>

<style scoped>
.qa {
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
.qa-left, .qa-right { position: relative; }

.thanks { margin: 0; font-size: 26px; font-weight: 500; color: var(--muted); opacity: 1; }
h1 {
  margin: 10px 0 0;
  font-size: 76px;
  font-weight: 800;
  line-height: 0.98;
  letter-spacing: -0.05em;
  color: var(--ink);
}
h1 .accent {
  background: linear-gradient(90deg, var(--accent), var(--teal));
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}
.meta { display: flex; flex-wrap: wrap; gap: 10px; margin-top: 40px; }
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
.chip.mono { font-size: 13px; }
.chip .dot { width: 7px; height: 7px; border-radius: 50%; background: var(--teal); }
.author { margin-top: 22px; font-size: 15px; font-weight: 600; color: var(--muted); }
</style>
