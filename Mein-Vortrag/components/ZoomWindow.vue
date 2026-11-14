<script setup lang="ts">
// Detail window that grows out of a point on the slide (transform origin) and collapses back into it.
// The parent needs `position: relative`; the window covers it completely.
defineProps<{ open: boolean; origin: string; tag: string; title: string }>()
</script>

<template>
  <Transition name="zoom">
    <div v-if="open" class="zoom-window" :style="{ transformOrigin: origin }">
      <header class="z-head">
        <span class="z-tag mono">{{ tag }}</span>
        <h3>{{ title }}</h3>
      </header>
      <div class="z-body"><slot /></div>
    </div>
  </Transition>
</template>

<style scoped>
.zoom-window {
  position: absolute;
  inset: -6px -14px;
  z-index: 5;
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 14px 20px 16px;
  border: 2px solid var(--violet);
  border-radius: 22px;
  background: var(--bg);
  box-shadow: 0 0 0 6px rgba(115, 87, 255, 0.12), 0 24px 60px rgba(15, 23, 42, 0.22);
}
.z-head { display: flex; align-items: center; gap: 14px; }
.z-tag { padding: 5px 11px; border-radius: 999px; font-size: 12px; font-weight: 600; color: #3b23c9; background: var(--violet-soft); }
.z-head h3 { margin: 0; font-size: 22px; font-weight: 750; letter-spacing: -0.02em; color: var(--ink); }
.z-body { flex: 1; min-height: 0; }

.zoom-enter-active { transition: transform 0.6s cubic-bezier(0.2, 0.9, 0.25, 1), opacity 0.35s ease; }
.zoom-leave-active { transition: transform 0.45s cubic-bezier(0.5, 0, 0.75, 0.4), opacity 0.2s ease 0.25s; }
.zoom-enter-from, .zoom-leave-to { opacity: 0; transform: scale(0.1); }
</style>
