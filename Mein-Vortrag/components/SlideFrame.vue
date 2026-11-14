<script setup lang="ts">
import { computed } from 'vue'
import { useSlideContext } from '@slidev/client'

// Shared frame for content slides: section tag + title on top, footer with event and page number
const props = defineProps<{ section: number; title: string; kicker?: string }>()

const SECTIONS = ['CTA heute', 'Workshop & Community', 'Neu in CTA v6', 'Roadmap', 'IBM Diamondback']

const { $page, $nav } = useSlideContext()
const sectionName = computed(() => SECTIONS[props.section - 1])
</script>

<template>
  <div class="frame">
    <header class="frame-head">
      <div class="crumbs">
        <span v-for="(s, i) in SECTIONS" :key="s" class="crumb" :class="{ on: i + 1 === section }">
          <b>{{ String(i + 1).padStart(2, '0') }}</b>{{ i + 1 === section ? s : '' }}
        </span>
      </div>
      <div class="kicker">{{ kicker ?? sectionName }}</div>
      <h2>{{ title }}</h2>
    </header>

    <main class="frame-body">
      <slot />
    </main>

    <footer class="frame-foot">
      <span>CTA Workshop 2026 · DESY Hamburg</span>
      <span class="mono">{{ String($page).padStart(2, '0') }} / {{ String($nav.total).padStart(2, '0') }}</span>
    </footer>
  </div>
</template>

<style scoped>
.frame {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  padding: 34px 64px 22px;
  background: var(--bg);
}
.crumbs { display: flex; gap: 6px; margin-bottom: 22px; }
.crumb {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  height: 26px;
  padding: 0 10px;
  border-radius: 999px;
  font-size: 12px;
  font-weight: 600;
  color: var(--muted);
  background: var(--surface-2);
}
.crumb b { font: 600 11px 'JetBrains Mono', monospace; opacity: 0.7; }
.crumb.on { color: #fff; background: var(--ink); }
.crumb.on b { opacity: 0.6; }

.frame-head h2 {
  margin: 10px 0 0;
  font-size: 38px;
  font-weight: 750;
  line-height: 1.12;
  letter-spacing: -0.028em;
  color: var(--ink);
}
.frame-body { position: relative; flex: 1; min-height: 0; margin-top: 22px; }
.frame-foot {
  display: flex;
  justify-content: space-between;
  padding-top: 12px;
  border-top: 1px solid var(--line);
  font-size: 12px;
  color: var(--muted);
}
</style>
