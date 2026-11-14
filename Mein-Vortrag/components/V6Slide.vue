<script setup lang="ts">
import { computed, ref } from 'vue'

// v5 → v6: each click swaps one component for its v6 replacement.
// Click 2 and 4 open an old-vs-new diagram (frontend, scheduler DB) as a window that
// grows out of its table row; the next click collapses it back into the row.
const props = defineProps<{ step: number }>()

// step → [rows done, view]
const PLAN: [number, 'table' | 'fe' | 'sch'][] = [
  [0, 'table'], [1, 'table'], [1, 'fe'], [2, 'table'], [2, 'sch'], [3, 'table'], [4, 'table'], [5, 'table'], [6, 'table'],
]
const state = computed(() => PLAN[Math.min(Math.max(props.step, 0), PLAN.length - 1)])
const done = computed(() => state.value[0])
const view = computed(() => state.value[1])

// Transform origin of a detail window: the centre of the v6 cell in the given table row.
// The offsets are relative to .body; the window overhangs it by 14px / 6px (see ZoomWindow).
const newCells = ref<HTMLElement[]>([])
const origin = (row: number) => {
  const cell = newCells.value[row]
  const line = cell?.parentElement
  if (!cell || !line)
    return `65% ${18 + row * 14}%`
  return `${cell.offsetLeft + cell.offsetWidth / 2 + 14}px ${line.offsetTop + line.offsetHeight / 2 + 6}px`
}

const rows = [
  { area: 'Frontend', old: 'XRootD-SSI-Frontend', neu: 'gRPC: Workflow-API und Admin-API', detail: 'getrennte Dienste mit mTLS, JWT oder Kerberos' },
  { area: 'Scheduler-DB', old: 'Objectstore', neu: 'PostgreSQL', detail: 'einfachere Transaktionen, Batching, bereit für Run-4' },
  { area: 'Tape Daemon', old: 'Prozesse per fork()', neu: 'Ein Prozess, Multi-Drive', detail: 'einfacher, mit Schutz vor Datenverlust' },
  { area: 'Konfiguration', old: '3 verschiedene Formate', neu: 'TOML in jedem Dienst', detail: '--config-strict stoppt bei Tippfehlern' },
  { area: 'Betrieb', old: 'Text-Logs', neu: 'JSON-Logs, Health-Probes, Metriken', detail: 'OpenTelemetry nicht mehr experimentell' },
  { area: 'Werkzeuge', old: 'cta-admin + cta-ops-*', neu: 'ein Kommando: cta', detail: 'gebaut auf dem neuen Python-SDK' },
]
</script>

<template>
  <SlideFrame :section="3" title="Was sich mit v6 ändert" kicker="Neu in CTA v6 · Q4 2026">
    <div class="body">
    <div class="wrap">
      <div class="cols">
        <span />
        <span class="col v5">CTA v5</span>
        <span />
        <span class="col v6">CTA v6</span>
      </div>
      <div v-for="(r, i) in rows" :key="r.area" class="row" :class="{ done: done > i, now: done === i + 1 }">
        <div class="area">{{ r.area }}</div>
        <div class="old">{{ r.old }}</div>
        <svg class="arrow" viewBox="0 0 60 20"><path d="M2 10 H52 M44 3 L54 10 L44 17" /></svg>
        <div ref="newCells" class="new">
          <template v-if="done > i">
            <b>{{ r.neu }}</b>
            <span>{{ r.detail }}</span>
          </template>
        </div>
      </div>
      <div class="status card">
        <span class="pulse" />
        <span>Dev-Releases sind im Test. Die <b>erste stabile v6</b> wird vor Jahresende erwartet, mit Upgrade-Anleitung.</span>
      </div>
    </div>

    <ZoomWindow :open="view === 'fe'" :origin="origin(0)" tag="Frontend" title="Vorher und nachher: aus einem Frontend werden zwei gRPC-Dienste">
      <FrontendCompare />
    </ZoomWindow>
    <ZoomWindow :open="view === 'sch'" :origin="origin(1)" tag="Scheduler-DB" title="Vorher und nachher: vom Objectstore zu PostgreSQL">
      <SchedulerCompare />
    </ZoomWindow>
    </div>
  </SlideFrame>
</template>

<style scoped>
.body { position: relative; height: 100%; }
.wrap { height: 100%; display: flex; flex-direction: column; gap: 8px; }
.cols, .row { display: grid; grid-template-columns: 150px 1fr 70px 1.5fr; align-items: center; gap: 12px; }
.col { font: 600 12px 'JetBrains Mono', monospace; letter-spacing: 0.1em; text-transform: uppercase; color: var(--muted); }
.col.v6 { color: var(--violet); }
.row { flex: 1; min-height: 0; }
.area { font-size: 15px; font-weight: 650; color: var(--ink); }
.old {
  position: relative;
  padding: 11px 14px;
  border: 1px solid var(--line);
  border-radius: 12px;
  background: var(--surface);
  font-size: 15px;
  color: var(--ink-2);
  transition: opacity 0.4s, color 0.4s;
}
.old::after {
  content: '';
  position: absolute;
  left: 12px;
  right: 12px;
  top: 50%;
  height: 2px;
  background: var(--rose);
  transform: scaleX(0);
  transform-origin: left;
  transition: transform 0.45s ease;
}
.row.done .old { opacity: 0.55; color: var(--muted); }
.row.done .old::after { transform: scaleX(1); }

.arrow { width: 60px; height: 20px; }
.arrow path { fill: none; stroke: var(--violet); stroke-width: 2.5; stroke-linecap: round; stroke-linejoin: round; stroke-dasharray: 80; stroke-dashoffset: 80; transition: stroke-dashoffset 0.5s ease 0.2s; }
.row.done .arrow path { stroke-dashoffset: 0; }

.new {
  display: flex;
  flex-direction: column;
  justify-content: center;
  min-height: 48px;
  padding: 8px 14px;
  border: 1.5px dashed var(--line);
  border-radius: 12px;
  transition: background 0.4s, border-color 0.4s;
}
.row.done .new { border-style: solid; border-color: #d8cfff; background: var(--violet-soft); animation: slide-in 0.5s cubic-bezier(0.2, 0.9, 0.3, 1.1) 0.35s both; }
.row.now .new { border-color: var(--violet); box-shadow: 0 0 0 4px rgba(115, 87, 255, 0.12); }
.new b { font-size: 16px; color: #3b23c9; }
.new span { margin-top: 2px; font-size: 13px; color: var(--ink-2); }

.status { display: flex; align-items: center; gap: 12px; margin-top: 6px; padding: 12px 18px; font-size: 15px; color: var(--ink-2); }
.status b { color: var(--violet); }
.pulse { flex: none; width: 10px; height: 10px; border-radius: 50%; background: var(--green); animation: pulse 1.6s ease-in-out infinite; }

@keyframes slide-in { from { opacity: 0; transform: translateX(-16px); } to { opacity: 1; transform: none; } }
@keyframes pulse { 0%, 100% { box-shadow: 0 0 0 0 rgba(22, 163, 74, 0.4); } 50% { box-shadow: 0 0 0 6px rgba(22, 163, 74, 0); } }
</style>
