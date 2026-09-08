<script setup>

</script>

<template>
  <div
      class="interop-diagram"
      role="img"
      aria-label="Kotlin classes are final by default, so Spring cannot create CGLIB proxies. The kotlin plugin.spring compiler plugin opens Spring annotated classes and methods, so proxying works again."
  >
    <svg viewBox="0 0 1200 470" preserveAspectRatio="xMidYMid meet" aria-hidden="true">
      <defs>
        <marker id="interop-arrow" markerWidth="9" markerHeight="9" refX="7" refY="4.5" orient="auto">
          <path d="M0 0 L9 4.5 L0 9z" fill="#7954f6"/>
        </marker>
        <marker id="interop-arrow-blocked" markerWidth="9" markerHeight="9" refX="7" refY="4.5" orient="auto">
          <path d="M0 0 L9 4.5 L0 9z" fill="#e5484d"/>
        </marker>
      </defs>

      <text class="section-label" x="40" y="30">KOTLIN</text>
      <text class="section-label" x="760" y="30">SPRING</text>

      <g class="card">
        <rect x="40" y="50" width="380" height="120" rx="16"/>
        <text class="card-title" x="70" y="90">class PetService</text>
        <text class="keyword" x="70" y="128">final</text>
        <text class="muted" x="150" y="128">by default</text>
      </g>

      <path class="wire blocked" d="M430 110 H745" marker-end="url(#interop-arrow-blocked)"/>
      <text class="wire-label blocked-label" x="500" y="92">cannot subclass</text>

      <g class="card card--blocked">
        <rect x="760" y="50" width="400" height="120" rx="16"/>
        <text class="card-title" x="790" y="90">CGLIB proxy</text>
        <text class="muted" x="790" y="128">@Transactional, @Configuration, ...</text>
      </g>

      <g class="plugin">
        <rect x="330" y="200" width="540" height="76" rx="14"/>
        <text class="plugin-title" x="600" y="250">kotlin("plugin.spring")</text>
      </g>
      <path class="wire" d="M230 176 V230 H320" marker-end="url(#interop-arrow)"/>
      <path class="wire" d="M880 230 H970 V176" marker-end="url(#interop-arrow)"/>

      <path class="wire" d="M600 290 V352" marker-end="url(#interop-arrow)"/>

      <g class="card card--fixed">
        <rect x="330" y="366" width="540" height="76" rx="14"/>
        <text class="keyword fixed" x="366" y="413">open</text>
        <text class="muted" x="466" y="413">for Spring annotated classes &amp; methods</text>
      </g>
    </svg>
  </div>
</template>

<style scoped>
.interop-diagram {
  --ink: #292531;
  --muted: #746d80;
  --purple: var(--slidev-theme-primary, #7954f6);
  --red: #e5484d;
  width: min(100%, 70rem);
  margin: 0.85rem auto 0.4rem;
}

svg {
  display: block;
  width: 100%;
  height: auto;
}

.section-label, .muted, .wire-label, .plugin-title, .keyword {
  font-family: 'JetBrains Mono', ui-monospace, monospace;
}

.section-label {
  fill: var(--muted);
  font-size: 13px;
  font-weight: 800;
  letter-spacing: .12em;
}

.card > rect {
  fill: rgb(121 84 246 / 5%);
  stroke: rgb(121 84 246 / 45%);
  stroke-width: 2;
}

.card--blocked > rect {
  fill: rgb(229 72 77 / 5%);
  stroke: rgb(229 72 77 / 45%);
}

.card--fixed > rect {
  fill: rgb(121 84 246 / 8%);
  stroke: rgb(121 84 246 / 55%);
}

.card-title {
  fill: var(--ink);
  font: 750 24px 'JetBrains Mono', ui-monospace, monospace;
}

.muted {
  fill: var(--muted);
  font-size: 17px;
}

.keyword {
  fill: var(--red);
  font-size: 26px;
  font-weight: 800;
}

.keyword.fixed {
  fill: var(--purple);
}

.wire {
  fill: none;
  stroke: var(--purple);
  stroke-width: 2.5;
  stroke-linecap: round;
}

.wire.blocked {
  stroke: var(--red);
  stroke-dasharray: 9 7;
}

.wire-label {
  fill: var(--purple);
  font-size: 15px;
  font-weight: 800;
}

.wire-label.blocked-label {
  fill: var(--red);
}

.plugin > rect {
  fill: rgb(121 84 246 / 12%);
  stroke: rgb(121 84 246 / 55%);
  stroke-width: 2;
}

.plugin-title {
  fill: var(--purple);
  font-size: 26px;
  font-weight: 800;
  text-anchor: middle;
}

html.dark .interop-diagram {
  --ink: #f2edf8;
  --muted: #b8afc5;
}

html.dark .card > rect {
  fill: rgb(121 84 246 / 13%);
}

html.dark .card--blocked > rect {
  fill: rgb(229 72 77 / 13%);
}

html.dark .plugin > rect {
  fill: rgb(121 84 246 / 20%);
}
</style>
