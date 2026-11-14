<script setup lang="ts">
// Colour key for the tiers in the Diamondback scatter plots
import { TIER_COLORS } from './diamondbackData'
</script>

<template>
  <div class="legend">
    <span class="lab">Farbe = Ebene im Slot</span>
    <span v-for="(c, i) in TIER_COLORS" :key="i" class="t"><i :style="{ background: c }" />T{{ i + 1 }}</span>
    <span class="lab"><i class="dash" />Median</span>
  </div>
</template>

<style scoped>
.legend { display: flex; flex-wrap: wrap; align-items: center; gap: 6px 12px; }
.lab { display: inline-flex; align-items: center; gap: 6px; font: 600 11.5px 'JetBrains Mono', monospace; color: var(--muted); }
.t { display: inline-flex; align-items: center; gap: 5px; font: 700 12px 'JetBrains Mono', monospace; color: var(--ink-2); }
i { width: 10px; height: 10px; border-radius: 50%; }
.dash { width: 16px; height: 0; border-radius: 0; border-top: 2px dashed var(--ink-2); }
</style>
