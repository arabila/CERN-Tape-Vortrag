<script setup lang="ts">
// v5 vs v6 frontend: one XRootD-SSI process for everything vs. two gRPC services.
// Based on "Splitting the CTA gRPC Frontend" (P. Ferreira) and "gRPC Frontend deployment
// in production at CERN" (J. Leduc), CTA Workshop 2026.
const v6Edges = [
  { id: 'wf', d: 'M140 76 L140 140', dur: 1.6, delay: 0 },
  { id: 'ad', d: 'M420 76 L420 140', dur: 1.6, delay: 0.8 },
  { id: 'wf-cat', d: 'M140 268 L140 300', dur: 1.2, delay: 0.4 },
  { id: 'ad-sch', d: 'M420 268 L420 300', dur: 1.2, delay: 1.1 },
]
</script>

<template>
  <div class="cmp">
    <section class="panel old">
      <header><span class="ver">v5</span><b>Ein Frontend für alles</b></header>
      <svg viewBox="0 0 560 370">
        <defs>
          <marker id="fc-a-old" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0 0 L10 5 L0 10 z" fill="#94a3b8" /></marker>
        </defs>
        <g class="client">
          <rect x="40" y="20" width="200" height="56" rx="12" />
          <text x="140" y="44" text-anchor="middle" class="t">EOS / dCache</text>
          <text x="140" y="62" text-anchor="middle" class="s">Workflow-Events</text>
        </g>
        <g class="client">
          <rect x="320" y="20" width="200" height="56" rx="12" />
          <text x="420" y="44" text-anchor="middle" class="t">cta-admin</text>
          <text x="420" y="62" text-anchor="middle" class="s">Operatoren</text>
        </g>
        <path d="M140 76 L240 146" class="edge" marker-end="url(#fc-a-old)" />
        <path d="M420 76 L320 146" class="edge" marker-end="url(#fc-a-old)" />
        <text x="168" y="118" class="elabel">WFE</text>
        <text x="368" y="118" class="elabel" text-anchor="end">Admin</text>

        <g class="mono-box">
          <rect x="130" y="148" width="300" height="104" rx="14" />
          <text x="280" y="180" text-anchor="middle" class="t big">cta-frontend</text>
          <text x="280" y="200" text-anchor="middle" class="s">XRootD-SSI</text>
          <g class="pill">
            <rect x="165" y="214" width="230" height="24" rx="12" />
            <text x="280" y="230" text-anchor="middle">1 Prozess · 1 Config · 1 Auth</text>
          </g>
        </g>

        <path d="M230 252 L170 300" class="edge" marker-end="url(#fc-a-old)" />
        <path d="M330 252 L390 300" class="edge" marker-end="url(#fc-a-old)" />
        <g class="db">
          <path d="M110 306 a60 10 0 0 0 120 0 v40 a60 10 0 0 1 -120 0 z" />
          <ellipse cx="170" cy="306" rx="60" ry="10" />
          <text x="170" y="338" text-anchor="middle" class="s">Catalogue</text>
        </g>
        <g class="db">
          <path d="M330 306 a60 10 0 0 0 120 0 v40 a60 10 0 0 1 -120 0 z" />
          <ellipse cx="390" cy="306" rx="60" ry="10" />
          <text x="390" y="338" text-anchor="middle" class="s">Scheduler-DB</text>
        </g>
      </svg>
    </section>

    <div class="arrow">
      <svg viewBox="0 0 40 40"><path d="M6 20 H32 M24 11 L34 20 L24 29" /></svg>
    </div>

    <section class="panel new">
      <header><span class="ver">v6</span><b>Zwei getrennte gRPC-Dienste</b></header>
      <svg viewBox="0 0 560 370">
        <defs>
          <marker id="fc-a-new" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0 0 L10 5 L0 10 z" fill="#7357ff" /></marker>
        </defs>
        <g class="client">
          <rect x="40" y="20" width="200" height="56" rx="12" />
          <text x="140" y="44" text-anchor="middle" class="t">EOS / dCache</text>
          <text x="140" y="62" text-anchor="middle" class="s">Workflow-Events</text>
        </g>
        <g class="client">
          <rect x="320" y="20" width="200" height="56" rx="12" />
          <text x="420" y="44" text-anchor="middle" class="t">cta / cta-admin</text>
          <text x="420" y="62" text-anchor="middle" class="s">Operatoren</text>
        </g>

        <path v-for="e in v6Edges" :id="`fc-${e.id}`" :key="e.id" :d="e.d" class="edge on" marker-end="url(#fc-a-new)" />
        <path d="M200 268 L380 300" class="edge on thin" marker-end="url(#fc-a-new)" />
        <path d="M360 268 L180 300" class="edge on thin" marker-end="url(#fc-a-new)" />
        <text x="150" y="112" class="elabel on">gRPC</text>
        <text x="430" y="112" class="elabel on">gRPC</text>
        <circle v-for="e in v6Edges" :key="`dot-${e.id}`" r="5" class="dot">
          <animateMotion :dur="`${e.dur}s`" :begin="`${e.delay}s`" repeatCount="indefinite" :path="e.d" />
        </circle>

        <g class="svc">
          <rect x="40" y="142" width="200" height="94" rx="14" />
          <text x="140" y="168" text-anchor="middle" class="t">cta-workflow-api</text>
          <text x="140" y="186" text-anchor="middle" class="s">archive · retrieve · delete</text>
          <g class="auth">
            <rect x="70" y="200" width="140" height="24" rx="12" />
            <text x="140" y="216" text-anchor="middle">mTLS oder JWT</text>
          </g>
        </g>
        <g class="svc">
          <rect x="320" y="142" width="200" height="94" rx="14" />
          <text x="420" y="168" text-anchor="middle" class="t">cta-admin-api</text>
          <text x="420" y="186" text-anchor="middle" class="s">alle Admin-Befehle</text>
          <g class="auth">
            <rect x="345" y="200" width="150" height="24" rx="12" />
            <text x="420" y="216" text-anchor="middle">Kerberos und/oder JWT</text>
          </g>
        </g>

        <!-- blocked: a workflow endpoint cannot run admin commands -->
        <g class="block">
          <path d="M240 189 L320 189" />
          <circle cx="280" cy="189" r="11" />
          <path d="M275 184 L285 194 M285 184 L275 194" class="x" />
        </g>

        <g class="runtime">
          <rect x="40" y="244" width="480" height="24" rx="8" />
          <text x="280" y="260" text-anchor="middle">gemeinsame CTA-Runtime · TOML · Health-Probes · Metriken</text>
        </g>

        <g class="db">
          <path d="M80 306 a60 10 0 0 0 120 0 v40 a60 10 0 0 1 -120 0 z" />
          <ellipse cx="140" cy="306" rx="60" ry="10" />
          <text x="140" y="338" text-anchor="middle" class="s">Catalogue</text>
        </g>
        <g class="db">
          <path d="M360 306 a60 10 0 0 0 120 0 v40 a60 10 0 0 1 -120 0 z" />
          <ellipse cx="420" cy="306" rx="60" ry="10" />
          <text x="420" y="338" text-anchor="middle" class="s">Scheduler-DB</text>
        </g>
      </svg>
    </section>

    <div class="facts">
      <div class="fact"><b>Weniger Rechte</b><span>Ein Workflow-Endpunkt kann keine Admin-Befehle ausführen und umgekehrt.</span></div>
      <div class="fact"><b>Getrennt betreiben</b><span>Eigene Pakete, Konfiguration und Skalierung je Rolle.</span></div>
      <div class="fact"><b>CERN: seit 11.10.2026 nur gRPC</b><span>Ausfallsicherheit über DNS-Alias und Client-Deadline.</span></div>
    </div>
  </div>
</template>

<style scoped>
.cmp { height: 100%; display: grid; grid-template-columns: 1fr 44px 1fr; grid-template-rows: 1fr auto; column-gap: 6px; row-gap: 12px; }
.panel { display: flex; flex-direction: column; min-height: 0; padding: 12px 14px 6px; border-radius: var(--radius); border: 1px solid var(--line); background: var(--surface); box-shadow: var(--shadow); }
.panel.new { border-color: #d8cfff; }
.panel header { display: flex; align-items: center; gap: 10px; }
.panel header b { font-size: 17px; color: var(--ink); }
.ver { padding: 3px 9px; border-radius: 999px; font: 700 12px 'JetBrains Mono', monospace; color: var(--muted); background: var(--surface-2); }
.new .ver { color: #fff; background: var(--violet); }
.panel svg { flex: 1; min-height: 0; width: 100%; }
.arrow { display: flex; align-items: center; }
.arrow svg { width: 40px; height: 40px; }
.arrow path { fill: none; stroke: var(--violet); stroke-width: 3; stroke-linecap: round; stroke-linejoin: round; }

.t { font: 650 15px Inter, sans-serif; fill: var(--ink); }
.t.big { font-size: 18px; }
.s { font: 500 12px 'JetBrains Mono', monospace; fill: var(--muted); }
.client rect { fill: var(--surface-2); stroke: #d5dbe4; stroke-dasharray: 4 4; }
.edge { fill: none; stroke: #b6c0ce; stroke-width: 2; }
.edge.on { stroke: #b9aaff; }
.edge.thin { stroke-width: 1.5; opacity: 0.7; }
.elabel { font: 600 11px 'JetBrains Mono', monospace; fill: var(--muted); }
.elabel.on { fill: var(--violet); }
.mono-box rect { fill: #f3f5f8; stroke: #b6c0ce; stroke-width: 1.5; }
.pill rect { fill: #fff; stroke: #d5dbe4; }
.pill text { font: 600 12px Inter, sans-serif; fill: var(--ink-2); }
.db path, .db ellipse { fill: #fff; stroke: #b6c0ce; stroke-width: 1.5; }
.new .db path, .new .db ellipse { stroke: #b9aaff; }

.svc rect { fill: var(--violet-soft); stroke: var(--violet); stroke-width: 1.5; }
.svc .t { fill: #3b23c9; }
.auth rect { fill: #fff; stroke: #c9bdff; }
.auth text { font: 600 11.5px Inter, sans-serif; fill: #3b23c9; }
.block path { stroke: var(--rose); stroke-width: 1.5; stroke-dasharray: 3 4; fill: none; }
.block circle { fill: #fff; stroke: var(--rose); stroke-width: 1.5; }
.block .x { stroke: var(--rose); stroke-width: 2; stroke-dasharray: none; stroke-linecap: round; }
.runtime rect { fill: #f4f1ff; stroke: #d8cfff; }
.runtime text { font: 600 11.5px 'JetBrains Mono', monospace; fill: #5b45d9; }
.dot { fill: var(--violet); }

.facts { grid-column: 1 / -1; display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; }
.fact { display: flex; flex-direction: column; gap: 3px; padding: 10px 14px; border-radius: 12px; background: var(--violet-soft); box-shadow: inset 3px 0 0 var(--violet); }
.fact b { font-size: 14.5px; color: #3b23c9; }
.fact span { font-size: 13px; line-height: 1.35; color: var(--ink-2); }
</style>
