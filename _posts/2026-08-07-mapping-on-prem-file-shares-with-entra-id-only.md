---
title: "Mapping On-Prem File Shares with Entra ID Only"
date: 2026-08-07
categories: [Azure]
tags: [azure]
image: /assets/img/blog/azurezon.png
---

<style>
  :root {
    --bg: #0b0f14;
    --panel: #121821;
    --panel-2: #161d28;
    --border: #232b38;
    --text: #dbe2ea;
    --text-dim: #93a1b3;
    --accent: #4fc3f7;
    --accent-2: #7c5cff;
    --green: #35d07f;
    --amber: #ffb648;
    --red: #ff5c72;
    --code-bg: #0d1117;
    --maxw: 880px;
  }

  * { box-sizing: border-box; }

  html { scroll-behavior: smooth; }

  body {
    margin: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    background: radial-gradient(1200px 600px at 50% -200px, #16202c 0%, var(--bg) 60%);
    color: var(--text);
    line-height: 1.7;
  }

  a { color: var(--accent); text-decoration: none; }
  a:hover { text-decoration: underline; }

  .wrap { max-width: var(--maxw); margin: 0 auto; padding: 0 24px; }

  /* Hero */
  header.hero {
    padding: 72px 0 40px;
    border-bottom: 1px solid var(--border);
  }
  .eyebrow {
    display: inline-flex; align-items: center; gap: 8px;
    font-size: 13px; letter-spacing: 0.06em; text-transform: uppercase;
    color: var(--accent); font-weight: 600; margin-bottom: 18px;
  }
  .eyebrow::before { content: "⚙"; }
  h1.title {
    font-size: clamp(28px, 5vw, 44px);
    line-height: 1.15;
    margin: 0 0 16px;
    background: linear-gradient(90deg, #ffffff, #b9c6d6);
    -webkit-background-clip: text; background-clip: text; color: transparent;
  }
  .subtitle { font-size: 18px; color: var(--text-dim); max-width: 700px; margin: 0 0 24px; }
  .tags { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 28px; }
  .tag {
    font-size: 12.5px; padding: 6px 12px; border-radius: 999px;
    background: var(--panel-2); border: 1px solid var(--border); color: var(--text-dim);
  }
  .meta-bar {
    display: flex; flex-wrap: wrap; gap: 24px; font-size: 14px; color: var(--text-dim);
    padding-top: 20px; border-top: 1px dashed var(--border);
  }
  .meta-bar strong { color: var(--text); }

  /* TOC */
  nav.toc {
    background: var(--panel);
    border: 1px solid var(--border);
    border-radius: 14px;
    padding: 22px 26px;
    margin: 40px 0;
  }
  nav.toc h2 { margin: 0 0 12px; font-size: 15px; text-transform: uppercase; letter-spacing: 0.05em; color: var(--text-dim); }
  nav.toc ol { margin: 0; padding-left: 20px; columns: 2; column-gap: 32px; }
  nav.toc li { margin-bottom: 8px; break-inside: avoid; font-size: 15px; }

  /* Sections */
  section { padding: 44px 0; border-bottom: 1px solid var(--border); }
  section:last-of-type { border-bottom: none; }

  h2.section-title {
    font-size: 28px; margin: 0 0 8px;
    display: flex; align-items: center; gap: 12px;
  }
  .section-num {
    display: inline-flex; align-items: center; justify-content: center;
    width: 36px; height: 36px; border-radius: 10px; flex: none;
    background: linear-gradient(135deg, var(--accent), var(--accent-2));
    color: #0b0f14; font-weight: 800; font-size: 16px;
  }
  h3 { font-size: 20px; margin: 32px 0 12px; color: #eef2f7; }
  p { color: var(--text); }
  .lede { color: var(--text-dim); font-size: 16px; margin-top: 0; }

  /* Callouts */
  .callout {
    border-radius: 12px; padding: 18px 20px; margin: 20px 0;
    border: 1px solid var(--border); background: var(--panel);
    display: flex; gap: 14px;
  }
  .callout .icon { font-size: 20px; flex: none; line-height: 1.4; }
  .callout.warning { border-color: rgba(255,182,72,0.4); background: rgba(255,182,72,0.06); }
  .callout.danger  { border-color: rgba(255,92,114,0.4); background: rgba(255,92,114,0.06); }
  .callout.info    { border-color: rgba(79,195,247,0.4); background: rgba(79,195,247,0.06); }
  .callout.success { border-color: rgba(53,208,127,0.4); background: rgba(53,208,127,0.06); }
  .callout strong { display: block; margin-bottom: 4px; }

  /* Step cards */
  .steps { counter-reset: step; margin: 24px 0; }
  .step {
    position: relative;
    padding: 22px 24px 22px 66px;
    background: var(--panel);
    border: 1px solid var(--border);
    border-radius: 14px;
    margin-bottom: 16px;
  }
  .step::before {
    counter-increment: step;
    content: counter(step);
    position: absolute; left: 18px; top: 20px;
    width: 32px; height: 32px; border-radius: 50%;
    background: linear-gradient(135deg, var(--accent), var(--accent-2));
    color: #0b0f14; font-weight: 800; font-size: 15px;
    display: flex; align-items: center; justify-content: center;
  }
  .step h4 { margin: 0 0 8px; font-size: 16.5px; color: #eef2f7; }
  .step p { margin: 6px 0; font-size: 15px; }

  /* Code blocks */
  pre[class*="language-"] {
    border-radius: 10px !important;
    border: 1px solid var(--border);
    font-size: 13.5px !important;
    margin: 12px 0 !important;
  }
  code { font-family: "SF Mono", SFMono-Regular, Menlo, Consolas, monospace; }
  p code, li code, td code {
    background: var(--code-bg); border: 1px solid var(--border);
    padding: 2px 6px; border-radius: 6px; font-size: 0.88em; color: #ffcb6b;
  }

  /* Tables */
  .table-wrap { overflow-x: auto; margin: 20px 0; border-radius: 12px; border: 1px solid var(--border); }
  table { border-collapse: collapse; width: 100%; font-size: 14.5px; }
  th, td { text-align: left; padding: 12px 16px; border-bottom: 1px solid var(--border); vertical-align: top; }
  th { background: var(--panel-2); color: var(--text-dim); font-size: 13px; text-transform: uppercase; letter-spacing: 0.03em; }
  tr:last-child td { border-bottom: none; }
  tr:hover td { background: rgba(255,255,255,0.02); }

  .badge { display: inline-block; padding: 3px 10px; border-radius: 999px; font-size: 12.5px; font-weight: 700; }
  .badge.high   { background: rgba(255,92,114,0.15); color: var(--red); }
  .badge.medium { background: rgba(255,182,72,0.15); color: var(--amber); }
  .badge.low    { background: rgba(53,208,127,0.15); color: var(--green); }
  .badge.dash   { background: rgba(147,161,179,0.15); color: var(--text-dim); }

  /* Checklist */
  .checklist { list-style: none; margin: 16px 0; padding: 0; }
  .checklist li {
    display: flex; align-items: flex-start; gap: 12px;
    padding: 10px 0; border-bottom: 1px dashed var(--border);
    font-size: 15px;
  }
  .checklist li:last-child { border-bottom: none; }
  .check-yes, .check-no {
    flex: none; width: 22px; height: 22px; border-radius: 6px;
    display: flex; align-items: center; justify-content: center;
    font-size: 13px; font-weight: 800; margin-top: 2px;
  }
  .check-yes { background: rgba(53,208,127,0.15); color: var(--green); }
  .check-no  { background: rgba(147,161,179,0.15); color: var(--text-dim); }

  /* Diagram box */
  .diagram-box {
    background: var(--panel); border: 1px solid var(--border);
    border-radius: 14px; padding: 20px; margin: 20px 0;
  }
  .diagram-caption { text-align: center; font-size: 13.5px; color: var(--text-dim); margin-top: 10px; }

  /* Custom visual flow diagram */
  .flow-wrap { margin: 24px 0; }
  .flow-scenario {
    background: var(--panel); border: 1px solid var(--border);
    border-radius: 16px; padding: 24px 26px; margin-bottom: 18px;
  }
  .flow-scenario.dimmed { background: var(--panel-2); border-style: dashed; }
  .flow-scenario-title {
    font-size: 14px; color: var(--text-dim); margin-bottom: 20px;
    display: flex; align-items: center; gap: 8px;
  }
  .flow-scenario-title strong { color: #eef2f7; }
  .flow-row {
    display: flex; align-items: center; gap: 8px;
  }
  .flow-node {
    flex: 0 0 168px;
    text-align: center;
    background: var(--code-bg);
    border: 1px solid var(--border);
    border-radius: 16px;
    padding: 20px 12px;
  }
  .flow-node.highlight { border-color: rgba(124,92,255,0.5); box-shadow: 0 0 0 1px rgba(124,92,255,0.2) inset; }
  .flow-node.cloud { border-color: rgba(79,195,247,0.5); box-shadow: 0 0 0 1px rgba(79,195,247,0.2) inset; }
  .flow-node .flow-icon { font-size: 34px; display: block; margin-bottom: 10px; }
  .flow-node .flow-title { font-weight: 700; font-size: 14.5px; color: #eef2f7; }
  .flow-node .flow-sub { font-size: 12px; color: var(--text-dim); margin-top: 5px; }

  .flow-connector { flex: 1; min-width: 90px; padding: 0 8px; }
  .flow-connector .flow-label {
    text-align: center; font-size: 12.5px; font-weight: 600; color: var(--text);
    margin-bottom: 8px; white-space: nowrap;
  }
  .flow-connector .flow-badge-icon { font-size: 16px; margin-right: 6px; }
  .flow-connector .flow-line {
    height: 3px; border-radius: 2px;
    background: linear-gradient(90deg, var(--accent), var(--accent-2));
    position: relative;
  }
  .flow-connector .flow-line.dashed {
    background: repeating-linear-gradient(90deg, var(--accent-2), var(--accent-2) 8px, transparent 8px, transparent 14px);
  }
  .flow-connector .flow-line::after {
    content: "▶"; position: absolute; right: -2px; top: -8px;
    color: var(--accent-2); font-size: 13px;
  }
  .flow-connector .flow-sublabel {
    text-align: center; font-size: 11px; color: var(--text-dim); margin-top: 8px;
  }
  .flow-connector .v-arrow { display: none; text-align: center; }
  .flow-connector .v-arrow-glyph { font-size: 22px; color: var(--accent-2); margin: 4px 0; }

  @media (max-width: 680px) {
    .flow-row { flex-direction: column; }
    .flow-node { width: 100%; }
    .flow-connector { width: 100%; padding: 10px 0; }
    .flow-connector .h-arrow { display: none; }
    .flow-connector .v-arrow { display: block; }
  }

  /* Big-picture hero overview graphic */
  .hero-visual {
    margin: 30px 0 6px;
    background: linear-gradient(160deg, rgba(79,195,247,0.10), rgba(124,92,255,0.06));
    border: 1px solid var(--border);
    border-radius: 20px;
    padding: 22px 18px 10px;
  }
  .hero-visual svg { width: 100%; height: auto; display: block; overflow: visible; }
  .hero-visual .caption {
    text-align: center; font-size: 13.5px; color: var(--text-dim);
    margin: 4px 0 6px; padding: 0 12px;
  }
  .hero-node {
    width: 100%; height: 100%; box-sizing: border-box; border-radius: 22px;
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    text-align: center; gap: 5px; padding: 10px;
    border: 1px solid var(--border);
    box-shadow: 0 12px 28px rgba(0,0,0,0.35);
  }
  .hero-node .hero-icon { font-size: 36px; line-height: 1; }
  .hero-node .hero-label { font-weight: 700; font-size: 13.5px; color: #eef2f7; }
  .hero-node .hero-sub { font-size: 11px; color: var(--text-dim); margin-top: 5px; }
  .hero-node--cloud  { background: linear-gradient(160deg, rgba(79,195,247,0.20), rgba(79,195,247,0.03)); border-color: rgba(79,195,247,0.45); }
  .hero-node--server { background: linear-gradient(160deg, rgba(124,92,255,0.20), rgba(124,92,255,0.03)); border-color: rgba(124,92,255,0.45); }
  .hero-node--users  { background: linear-gradient(160deg, rgba(53,208,127,0.20), rgba(53,208,127,0.03)); border-color: rgba(53,208,127,0.45); }
  .hero-badge {
    display: flex; align-items: center; justify-content: center; gap: 6px;
    width: 100%; height: 100%; box-sizing: border-box;
    background: var(--code-bg); border: 1px solid var(--border); border-radius: 999px;
    padding: 6px 10px; font-size: 11.5px; white-space: nowrap; color: var(--text);
    box-shadow: 0 6px 16px rgba(0,0,0,0.4);
  }
  .hero-edge { fill: none; stroke-width: 3.5; stroke-linecap: round; }
  .hero-edge--kerberos { stroke: var(--accent); }
  .hero-edge--pku2u { stroke: var(--green); }
  .hero-edge--sync { stroke: var(--accent-2); stroke-dasharray: 2 11; animation: hero-flow 1.4s linear infinite; }
  @keyframes hero-flow { to { stroke-dashoffset: -130; } }
  @media (max-width: 680px) {
    .hero-node .hero-icon { font-size: 26px; }
    .hero-node .hero-label { font-size: 11.5px; }
    .hero-node .hero-sub { display: none; }
    .hero-badge { font-size: 10px; padding: 5px 8px; }
  }

  /* Architecture legend / illustrated cards */
  .arch-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(230px, 1fr)); gap: 16px; margin: 22px 0; }
  .arch-card {
    background: var(--panel); border: 1px solid var(--border); border-radius: 14px; padding: 20px;
  }
  .arch-card .emoji { font-size: 28px; margin-bottom: 10px; display: block; }
  .arch-card h4 { margin: 0 0 6px; font-size: 15.5px; }
  .arch-card p { font-size: 13.5px; color: var(--text-dim); margin: 0; }

  /* Reference cards */
  .ref-card {
    background: var(--panel); border: 1px solid var(--border); border-radius: 14px;
    padding: 18px 20px; margin-bottom: 14px;
  }
  .ref-card .ref-title { font-weight: 700; color: #eef2f7; font-size: 15.5px; }
  .ref-card .ref-meta { font-size: 13px; color: var(--text-dim); margin: 4px 0 10px; }

  /* Footer */
  footer { padding: 40px 0 80px; color: var(--text-dim); font-size: 13.5px; }
  footer hr { border: none; border-top: 1px solid var(--border); margin-bottom: 20px; }

  ::selection { background: rgba(79,195,247,0.25); }

  @media (max-width: 640px) {
    nav.toc ol { columns: 1; }
    .step { padding-left: 56px; }
  }
</style>

<div class="wrap">
  <header class="hero">
    <div class="eyebrow">Field-tested runbook</div>
    <h1 class="title">Mapping On-Prem File Shares with Entra ID Only</h1>
    <p class="subtitle">
      No on-prem Active Directory. No Kerberos KDC. Just a Windows Server 2025 box,
      Azure Arc, and a decades-old authentication protocol most people have forgotten exists.
      Here's the fully validated, step-by-step build — including the lab evidence that proves it actually works.
    </p>
    <div class="tags">
      <span class="tag">Azure Arc</span>
      <span class="tag">Microsoft Entra ID</span>
      <span class="tag">PKU2U</span>
      <span class="tag">Windows Server 2025</span>
      <span class="tag">Azure Files</span>
      <span class="tag">Azure File Sync</span>
      <span class="tag">Intune</span>
    </div>
    <div class="meta-bar">
      <span><strong>Storage account:</strong> datasharetesting</span>
      <span><strong>Tenant model:</strong> cloud-only Entra ID</span>
      <span><strong>Status:</strong> Lab-validated ✅</span>
    </div>

    <div class="hero-visual">
      <svg viewBox="0 0 900 400" role="img" aria-label="Overview diagram: Azure Files in the cloud, an on-prem server, and Windows 11 users, connected through Entra Kerberos, PKU2U, and Azure File Sync">
        <path class="hero-edge hero-edge--kerberos" d="M450,150 C560,190 645,200 705,232" />
        <path class="hero-edge hero-edge--pku2u" d="M222,300 C350,346 545,346 673,300" />
        <path class="hero-edge hero-edge--sync" d="M410,150 C330,196 235,196 168,232" />

        <foreignObject x="260" y="148" width="170" height="40">
          <div xmlns="http://www.w3.org/1999/xhtml" class="hero-badge">🔄 Azure File Sync</div>
        </foreignObject>
        <foreignObject x="470" y="148" width="170" height="40">
          <div xmlns="http://www.w3.org/1999/xhtml" class="hero-badge">🔑 Entra Kerberos</div>
        </foreignObject>
        <foreignObject x="350" y="318" width="200" height="40">
          <div xmlns="http://www.w3.org/1999/xhtml" class="hero-badge">🔐 PKU2U (on-prem only)</div>
        </foreignObject>

        <foreignObject x="375" y="10" width="150" height="140">
          <div xmlns="http://www.w3.org/1999/xhtml" class="hero-node hero-node--cloud">
            <span class="hero-icon">☁️</span>
            <span class="hero-label">Azure Files</span>
            <span class="hero-sub">datasharetesting</span>
          </div>
        </foreignObject>

        <foreignObject x="70" y="232" width="150" height="140">
          <div xmlns="http://www.w3.org/1999/xhtml" class="hero-node hero-node--server">
            <span class="hero-icon">🖥️</span>
            <span class="hero-label">On-Prem Server</span>
            <span class="hero-sub">Arc-connected, no domain</span>
          </div>
        </foreignObject>

        <foreignObject x="680" y="232" width="150" height="140">
          <div xmlns="http://www.w3.org/1999/xhtml" class="hero-node hero-node--users">
            <span class="hero-icon">💻</span>
            <span class="hero-label">Windows 11 Users</span>
            <span class="hero-sub">Entra-joined, cloud-only</span>
          </div>
        </foreignObject>
      </svg>
      <p class="caption">Same files everywhere: users reach Azure directly over Entra Kerberos, reach the on-prem server directly over PKU2U when they're on that network, and Azure File Sync quietly keeps the server and the cloud share identical in the background.</p>
    </div>
  </header>

  <nav class="toc">
    <h2>Contents</h2>
    <ol>
      <li><a href="#overview">Architecture Overview</a></li>
      <li><a href="#part1">Part 1 — On-Prem Access via Arc + PKU2U</a></li>
      <li><a href="#part2">Part 2 — Bridging to Azure Files with File Sync</a></li>
      <li><a href="#evidence">Validation Evidence</a></li>
      <li><a href="#prior-art">Known Prior Art / References</a></li>
      <li><a href="#risk">Risk Assessment / Long-Term Viability</a></li>
    </ol>
  </nav>

  <section id="overview">
    <h2 class="section-title"><span class="section-num">1</span>Architecture Overview</h2>
    <p class="lede">Three pieces work together so that both on-prem-reachable and fully remote users can get to the same data — without a domain controller anywhere in the picture.</p>

    <div class="flow-wrap">

      <div class="flow-scenario">
        <div class="flow-scenario-title">① Users who <strong>can reach</strong> the on-prem network</div>
        <div class="flow-row">
          <div class="flow-node">
            <span class="flow-icon">💻</span>
            <div class="flow-title">Windows 11</div>
            <div class="flow-sub">Intune-managed, cloud-only</div>
          </div>
          <div class="flow-connector">
            <div class="h-arrow">
              <div class="flow-label"><span class="flow-badge-icon">🔐</span>PKU2U + Entra ID</div>
              <div class="flow-line"></div>
              <div class="flow-sublabel">no domain, no KDC — validated</div>
            </div>
            <div class="v-arrow">
              <span class="flow-badge-icon">🔐</span>
              <div class="flow-label">PKU2U + Entra ID</div>
              <div class="v-arrow-glyph">↓</div>
              <div class="flow-sublabel">no domain, no KDC — validated</div>
            </div>
          </div>
          <div class="flow-node highlight">
            <span class="flow-icon">🏢</span>
            <div class="flow-title">AZ-Filez-01</div>
            <div class="flow-sub">On-prem server · sharetesting2</div>
          </div>
        </div>
      </div>

      <div class="flow-scenario">
        <div class="flow-scenario-title">② Users who <strong>cannot reach</strong> the on-prem network</div>
        <div class="flow-row">
          <div class="flow-node">
            <span class="flow-icon">💻</span>
            <div class="flow-title">Windows 11</div>
            <div class="flow-sub">Intune-managed, cloud-only</div>
          </div>
          <div class="flow-connector">
            <div class="h-arrow">
              <div class="flow-label"><span class="flow-badge-icon">🔑</span>Entra Kerberos</div>
              <div class="flow-line"></div>
              <div class="flow-sublabel">standard, Microsoft-supported</div>
            </div>
            <div class="v-arrow">
              <span class="flow-badge-icon">🔑</span>
              <div class="flow-label">Entra Kerberos</div>
              <div class="v-arrow-glyph">↓</div>
              <div class="flow-sublabel">standard, Microsoft-supported</div>
            </div>
          </div>
          <div class="flow-node cloud">
            <span class="flow-icon">☁️</span>
            <div class="flow-title">Azure Files</div>
            <div class="flow-sub">sharetesting2 · synced copy</div>
          </div>
        </div>
      </div>

      <div class="flow-scenario">
        <div class="flow-scenario-title">③ All users — separate, always-cloud share</div>
        <div class="flow-row">
          <div class="flow-node">
            <span class="flow-icon">💻</span>
            <div class="flow-title">Windows 11</div>
            <div class="flow-sub">Intune-managed, cloud-only</div>
          </div>
          <div class="flow-connector">
            <div class="h-arrow">
              <div class="flow-label"><span class="flow-badge-icon">🔑</span>Entra Kerberos</div>
              <div class="flow-line"></div>
              <div class="flow-sublabel">standard, Microsoft-supported</div>
            </div>
            <div class="v-arrow">
              <span class="flow-badge-icon">🔑</span>
              <div class="flow-label">Entra Kerberos</div>
              <div class="v-arrow-glyph">↓</div>
              <div class="flow-sublabel">standard, Microsoft-supported</div>
            </div>
          </div>
          <div class="flow-node cloud">
            <span class="flow-icon">☁️</span>
            <div class="flow-title">Azure Files</div>
            <div class="flow-sub">sharetesting · primary share</div>
          </div>
        </div>
      </div>

      <div class="flow-scenario dimmed">
        <div class="flow-scenario-title">Behind the scenes — keeping both copies identical</div>
        <div class="flow-row">
          <div class="flow-node">
            <span class="flow-icon">🏢</span>
            <div class="flow-title">AZ-Filez-01</div>
            <div class="flow-sub">Local folder: sharetesting2</div>
          </div>
          <div class="flow-connector">
            <div class="h-arrow">
              <div class="flow-label"><span class="flow-badge-icon">🔄</span>Azure File Sync</div>
              <div class="flow-line dashed"></div>
              <div class="flow-sublabel">HTTPS 443, continuous</div>
            </div>
            <div class="v-arrow">
              <span class="flow-badge-icon">🔄</span>
              <div class="flow-label">Azure File Sync</div>
              <div class="v-arrow-glyph">↕</div>
              <div class="flow-sublabel">HTTPS 443, continuous</div>
            </div>
          </div>
          <div class="flow-node cloud">
            <span class="flow-icon">☁️</span>
            <div class="flow-title">Azure Files</div>
            <div class="flow-sub">sharetesting2 · cloud endpoint</div>
          </div>
        </div>
      </div>

    </div>

    <div class="arch-grid">
      <div class="arch-card">
        <span class="emoji">☁️</span>
        <h4>sharetesting</h4>
        <p>Direct Azure Files access for cloud-only Windows 11 devices. Mapped via Intune, authenticated with Entra Kerberos. No server involved.</p>
      </div>
      <div class="arch-card">
        <span class="emoji">🏢</span>
        <h4>AZ-Filez-01</h4>
        <p>Non-domain-joined Windows Server 2025, connected to Azure Arc. Local <code>sharetesting2</code> share reachable via PKU2U + Entra ID — no AD, no Kerberos KDC. Lab-validated below.</p>
      </div>
      <div class="arch-card">
        <span class="emoji">🔄</span>
        <h4>sharetesting2 (Azure)</h4>
        <p>A File Sync cloud endpoint mirroring the server's local share, so users who can't reach the on-prem box get the same data straight from Azure.</p>
      </div>
    </div>
  </section>

  <section id="part1">
    <h2 class="section-title"><span class="section-num">2</span>Part 1 — On-Prem Access via Azure Arc + PKU2U</h2>
    <p class="lede">This is the interesting part: mapping a drive to a server with zero Active Directory, using an authentication protocol most admins have never had a reason to touch.</p>

    <h3>Prerequisites</h3>
    <div class="table-wrap">
      <table>
        <thead><tr><th>Requirement</th><th>Detail</th></tr></thead>
        <tbody>
          <tr><td>Server OS</td><td>Windows Server 2025, <strong>not domain-joined</strong> (workgroup)</td></tr>
          <tr><td>Server connectivity</td><td>Connected to <strong>Azure Arc</strong> (<code>azcmagent connect</code>)</td></tr>
          <tr><td>Extension</td><td><code>AADLoginForWindows</code> deployed to the Arc machine — Entra-joins the server (does <strong>not</strong> by itself enable SMB auth)</td></tr>
          <tr><td>Tenant match</td><td>Server and client must be <strong>Entra-joined to the same tenant</strong></td></tr>
          <tr><td>Client</td><td>Windows 11, Entra-joined (cloud-only), Intune-managed</td></tr>
          <tr><td>Name resolution</td><td>Real DNS or hosts entry — <strong>PKU2U does not work against a bare IP</strong></td></tr>
        </tbody>
      </table>
    </div>

    <h3>How it actually works</h3>
    <p><code>AADLoginForWindows</code> only handles interactive/RDP sign-in via Azure RBAC (VM Administrator/User Login roles) — it registers nothing in the SMB stack. What actually authorizes a mapped drive with no domain and no Kerberos KDC is:</p>
    <ul>
      <li><strong>PKU2U</strong> (Negotiate SSP <code>NegoExtender</code>) — lets two Entra-joined, non-domain machines authenticate an SMB session using online (Entra) identities.</li>
      <li><strong>NTFS ACLs keyed to Entra identities</strong> (<code>AzureAD\&lt;upn&gt;</code>) — authorization is still local NTFS; there's no Azure RBAC governing file-level access on a self-hosted server, unlike Azure Files.</li>
    </ul>

    <div class="callout warning">
      <div class="icon">⚠️</div>
      <div>
        <strong>Support status</strong>
        This combination (Arc + AADLoginForWindows + PKU2U for SMB) is <strong>not an officially documented, supported Microsoft workflow</strong> — it's an engineered combination of supported building blocks. Treat as validated-in-lab, not vendor-guaranteed. If the server must later join an on-prem AD domain, this entire path is unavailable.
      </div>
    </div>

    <h3>Setup steps</h3>
    <div class="steps">

      <div class="step">
        <h4>Connect the server to Azure Arc and deploy AADLoginForWindows</h4>
        <p>Confirm the join state before doing anything else:</p>
        <pre><code class="language-powershell">dsregcmd /status
# AzureAdJoined : YES / DomainJoined : NO</code></pre>
      </div>

      <div class="step">
        <h4>Enable PKU2U on both the server and every Windows 11 client</h4>
        <pre><code class="language-powershell">New-Item -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\pku2u" -Force
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\pku2u" -Name "AllowOnlineID" -Value 1 -Type DWord</code></pre>
        <p>Deploy this to Windows 11 clients at scale via <strong>Intune → Settings Catalog</strong> rather than by hand.</p>
      </div>

      <div class="step">
        <h4>Create the share and folder on the server</h4>
        <pre><code class="language-powershell">New-Item -Path "C:\Shares\sharetesting2" -ItemType Directory -Force
New-SmbShare -Name "sharetesting2" -Path "C:\Shares\sharetesting2" -FullAccess "Everyone"</code></pre>
        <p>Share permissions are left open — access is controlled entirely by NTFS ACLs in the next step (standard pattern).</p>
      </div>

      <div class="step">
        <h4>Grant NTFS access to each Entra identity that needs it</h4>
        <pre><code class="language-powershell">icacls "C:\Shares\sharetesting2" /grant "AzureAD\&lt;user-upn&gt;:(OI)(CI)M"</code></pre>
        <p>Confirm resolution — <code>icacls</code> should show the resolved display name, not a raw SID:</p>
        <pre><code class="language-powershell">icacls "C:\Shares\sharetesting2"</code></pre>
      </div>

      <div class="step">
        <h4>Map the drive from a Windows 11 client</h4>
        <p>Via Intune drive-mapping policy/script:</p>
        <pre><code class="language-powershell">net use Z: \\az-filez-01\sharetesting2</code></pre>
      </div>

      <div class="step">
        <h4>Verify it authenticated via PKU2U, not NTLM or domain Kerberos</h4>
        <pre><code class="language-powershell">Get-SmbConnection</code></pre>
        <p><code>UserName</code>/<code>Credential</code> should show <code>AzureAD\&lt;DisplayName&gt;</code>. On the server's Security event log, filter <code>4624</code> events and check <code>AuthPackage</code> — expect <code>NegoExtender</code> (this is how PKU2U-based logons are labeled in the LSA).</p>
      </div>

    </div>
  </section>

  <section id="part2">
    <h2 class="section-title"><span class="section-num">3</span>Part 2 — Bridging to Azure Files with Azure File Sync</h2>
    <p class="lede">Gives the same data a second access path — directly from Azure — for users who can't reach the on-prem server over the network.</p>

    <div class="table-wrap">
      <table>
        <thead><tr><th>Item</th><th>Value</th></tr></thead>
        <tbody>
          <tr><td>Storage account</td><td><code>datasharetesting</code> (existing)</td></tr>
          <tr><td>Azure file share (cloud endpoint)</td><td><code>sharetesting2</code> — same name as the local server share, for clarity</td></tr>
          <tr><td>Server endpoint (local path)</td><td><code>C:\Shares\sharetesting2</code> on <code>AZ-Filez-01</code></td></tr>
          <tr><td>Cloud tiering</td><td><strong>Disabled</strong> — full local copy always kept on the server</td></tr>
          <tr><td>Region</td><td>Storage Sync Service in the <strong>same region</strong> as <code>datasharetesting</code></td></tr>
        </tbody>
      </table>
    </div>

    <div class="callout info">
      <div class="icon">ℹ️</div>
      <div>
        Azure File Sync is a <strong>separate mechanism from Azure Arc</strong>. It uses its own agent (Storage Sync Service + registered server), not the Arc extension from Part 1. Both can coexist on the same server.
      </div>
    </div>

    <h3>Prerequisites</h3>
    <ul>
      <li>Outbound <strong>HTTPS 443</strong> from the server to Azure (the sync engine uses REST calls over 443, not SMB).</li>
      <li>An Azure AD account with <strong>Contributor</strong> rights on the Storage Sync Service, used once to register the server.</li>
      <li>Latest <strong>Azure File Sync agent</strong> compatible with Windows Server 2025.</li>
    </ul>

    <h3>Setup steps</h3>
    <div class="steps">

      <div class="step">
        <h4>Create the cloud endpoint share</h4>
        <pre><code class="language-powershell">New-AzStorageShare -Name "sharetesting2" -Context (Get-AzStorageAccount -ResourceGroupName "&lt;rg&gt;" -Name "datasharetesting").Context</code></pre>
      </div>

      <div class="step">
        <h4>Create a Storage Sync Service</h4>
        <pre><code class="language-powershell">New-AzStorageSyncService -ResourceGroupName "&lt;rg&gt;" -Name "contoso-sync" -Location "&lt;region&gt;"</code></pre>
      </div>

      <div class="step">
        <h4>Install the Azure File Sync agent and register the server</h4>
        <p>Download from <a href="https://aka.ms/afs/agent" target="_blank" rel="noopener">aka.ms/afs/agent</a>, run the installer, and sign in interactively with the Contributor account when the Server Registration UI launches. Select the subscription, resource group, and Storage Sync Service. This creates a registered server resource in Azure — independent of the Arc connection from Part 1.</p>
      </div>

      <div class="step">
        <h4>Create a Sync Group and endpoints</h4>
        <pre><code class="language-powershell">$syncGroup = New-AzStorageSyncGroup -ResourceGroupName "&lt;rg&gt;" -StorageSyncServiceName "contoso-sync" -Name "sharetesting2-syncgroup"

New-AzStorageSyncCloudEndpoint -ResourceGroupName "&lt;rg&gt;" -StorageSyncServiceName "contoso-sync" `
  -SyncGroupName "sharetesting2-syncgroup" -StorageAccountResourceId "&lt;storage-account-resource-id&gt;" -AzureFileShareName "sharetesting2"

New-AzStorageSyncServerEndpoint -ResourceGroupName "&lt;rg&gt;" -StorageSyncServiceName "contoso-sync" `
  -SyncGroupName "sharetesting2-syncgroup" -ServerId "&lt;registered-server-id&gt;" -ServerFolder "C:\Shares\sharetesting2" `
  -CloudTiering $false</code></pre>
      </div>

      <div class="step">
        <h4>Monitor the initial sync</h4>
        <p>Since the local folder already has data, Azure File Sync performs a <strong>namespace merge</strong>, uploading existing files to the new cloud endpoint. Watch <em>Portal → Storage Sync Service → Sync Group → Server Endpoint → Health</em> until it reads <strong>"Healthy" / "Up to date"</strong>.</p>
      </div>

      <div class="step">
        <h4>Enable identity-based access on the new share</h4>
        <p>Microsoft Entra Kerberos is a storage-account-level setting, so <code>sharetesting2</code> inherits it automatically from <code>datasharetesting</code>. Assign RBAC (e.g. <strong>Storage File Data SMB Share Contributor</strong>) to the users/groups who need direct cloud access.</p>
        <div class="callout warning">
          <div class="icon">⚠️</div>
          <div>NTFS ACLs set locally are expected to replicate to Azure Files as Windows ACLs once synced — <strong>verify this per file/folder after the first sync</strong> rather than assuming it.</div>
        </div>
      </div>

      <div class="step">
        <h4>Map the drive for users who can't reach the on-prem server</h4>
        <pre><code class="language-powershell">New-PSDrive -Name Y -PSProvider FileSystem -Root "\\datasharetesting.file.core.windows.net\sharetesting2" -Persist</code></pre>
      </div>

    </div>

    <h3>Resulting access matrix</h3>
    <div class="table-wrap">
      <table>
        <thead><tr><th>User group</th><th>Path</th><th>Mechanism</th></tr></thead>
        <tbody>
          <tr><td>Cloud-only, on-prem network reachable</td><td><code>\\az-filez-01\sharetesting2</code></td><td>PKU2U + Entra ID</td></tr>
          <tr><td>Cloud-only, no on-prem network path</td><td><code>\\datasharetesting.file.core.windows.net\sharetesting2</code></td><td>Entra Kerberos</td></tr>
          <tr><td>Both groups</td><td>Same underlying data</td><td>Kept in sync by Azure File Sync</td></tr>
        </tbody>
      </table>
    </div>
  </section>

  <section id="evidence">
    <h2 class="section-title"><span class="section-num">4</span>Validation Evidence</h2>
    <p class="lede">Tested and confirmed on <code>AZ-Filez-01</code> — this isn't theoretical.</p>

    <ul class="checklist">
      <li><span class="check-yes">✓</span> Server &amp; client Entra-joined, same tenant</li>
      <li><span class="check-yes">✓</span> PKU2U enabled both ends</li>
      <li><span class="check-yes">✓</span> NTFS ACL resolved Entra UPN → cloud identity</li>
      <li><span class="check-yes">✓</span> <code>net use</code> succeeded, no credential prompt</li>
      <li><span class="check-yes">✓</span> <code>Get-SmbConnection</code> shows <code>AzureAD\&lt;user&gt;</code> identity</li>
      <li><span class="check-yes">✓</span> Server Security log <code>4624</code> — <code>AuthPackage = NegoExtender</code> (PKU2U)</li>
      <li><span class="check-yes">✓</span> Read + write access confirmed</li>
      <li><span class="check-yes">✓</span> Survives reboot (remembered mapping + manual reconnect)</li>
      <li><span class="check-yes">✓</span> Share permissions alone are insufficient — NTFS ACL is the real enforcement point</li>
    </ul>

    <div class="callout success">
      <div class="icon">✅</div>
      <div>
        <strong>Proof of NTFS enforcement</strong>
        With share permissions set to "Everyone" but <em>before</em> running <code>icacls</code>, the mapped drive connected but <code>dir</code>/<code>mkdir</code> returned <code>Access is denied</code>. After granting the ACL, both succeeded immediately — no re-mapping or reboot needed. Authorization is enforced entirely by the local NTFS ACL, not the share permission list.
      </div>
    </div>

    <h3>Still to validate</h3>
    <ul class="checklist">
      <li><span class="check-no">○</span> Negative test — a different Entra user not in the ACL is denied access</li>
      <li><span class="check-no">○</span> Intune-deployed drive mapping reproduces the same result unattended</li>
      <li><span class="check-no">○</span> Behavior after PRT refresh / longer elapsed time</li>
      <li><span class="check-no">○</span> Azure File Sync namespace merge completes cleanly and NTFS ACLs replicate as expected</li>
    </ul>
  </section>

  <section id="prior-art">
    <h2 class="section-title"><span class="section-num">5</span>Known Prior Art / References</h2>
    <p class="lede">There is <strong>no official Microsoft Learn documentation</strong> for the Arc + PKU2U SMB file-share combination used in Part 1. Here's what exists publicly.</p>

    <div class="ref-card">
      <div class="ref-title">Anthony Fontanez — "Internet-facing File Servers, with a dash of Entra Authentication"</div>
      <div class="ref-meta">July 2025 · <a href="https://anthonyfontanez.com/index.php/2025/07/27/internet-facing-file-servers-with-a-dash-of-entra-authentication/" target="_blank" rel="noopener">anthonyfontanez.com</a></div>
      <p>Identifies <strong>PKU2U</strong> as the mechanism enabling Entra-only SMB file share access. Confirms in the comments that both an Azure-hosted Server 2022+ VM and an <strong>on-prem Arc-attached Server 2025</strong> box were tested — but the on-prem Arc case isn't documented step-by-step, and there's no event-log-level proof (<code>4624</code>/<code>NegoExtender</code>) shown.</p>
    </div>

    <div class="ref-card">
      <div class="ref-title">Mindcore — "Modern Server Management: Azure Arc RDP with Entra ID Authentication"</div>
      <div class="ref-meta">August 2025 · <a href="https://blog.mindcore.dk/2025/08/modern-server-management-azure-arc-rdp-with-entra-id-authentication/" target="_blank" rel="noopener">blog.mindcore.dk</a></div>
      <p>Documents the Arc + <code>AADLoginForWindows</code> extension deployment itself, including the <code>mdmId=""</code> workaround needed on Arc machines. Explicitly labels the whole approach <strong>POC-only, not production-ready</strong>. Covers RDP sign-in only — no mention of PKU2U or file shares.</p>
    </div>

    <div class="ref-card">
      <div class="ref-title">Microsoft Q&amp;A thread</div>
      <div class="ref-meta">October 2024 · <a href="https://learn.microsoft.com/en-us/answers/questions/2109100/microsoft-entra-authentication-on-windows-server" target="_blank" rel="noopener">learn.microsoft.com</a></div>
      <p>Microsoft support staff confirm <code>AADLoginForWindows</code> was <strong>not officially supported</strong> on hybrid Arc machines at that time.</p>
    </div>

    <div class="callout info">
      <div class="icon">💬</div>
      <div>
        A reader comment on Fontanez's post (Nov 2025) describes asking multiple consultants and a Microsoft expert about mapping drives on an Entra-only tenant with no AD — none had a solution, and Microsoft was reportedly <strong>surprised</strong> the approach worked when shown it.
      </div>
    </div>

    <p>What appears original here: a single validated runbook combining on-prem Arc-attached WS2025, PKU2U-secured local SMB access for Intune-managed cloud-only clients, <strong>and</strong> Azure File Sync bridging the same data into Azure Files — with concrete validation evidence. This full combination, documented end-to-end with test evidence, doesn't appear to exist publicly elsewhere as of this writing.</p>
  </section>

  <section id="risk">
    <h2 class="section-title"><span class="section-num">6</span>Risk Assessment / Long-Term Viability</h2>
    <p class="lede">Be clear-eyed about what could break this — none of it requires Microsoft to "target" the approach specifically.</p>

    <div class="table-wrap">
      <table>
        <thead><tr><th>Risk</th><th>Likelihood</th><th>Notes</th></tr></thead>
        <tbody>
          <tr>
            <td>Customer security baseline disables PKU2U</td>
            <td><span class="badge high">High</span></td>
            <td>Microsoft Security Baselines and CIS Benchmarks commonly recommend <strong>disabling</strong> "Allow PKU2U authentication requests to use online identities." If a baseline is applied via Intune/GPO, this can silently break the solution with zero Microsoft involvement. <strong>Check for existing baselines before deployment.</strong></td>
          </tr>
          <tr>
            <td>Microsoft further restricts PKU2U</td>
            <td><span class="badge medium">Medium</span></td>
            <td>Not unprecedented — CVE-2021-25195 already led Microsoft to change PKU2U's default posture on domain/hybrid-joined devices.</td>
          </tr>
          <tr>
            <td><code>AADLoginForWindows</code> on Arc changes behavior</td>
            <td><span class="badge medium">Medium</span></td>
            <td>Explicitly unsupported/POC per community sources and Microsoft support. No compatibility guarantee for an undocumented use case.</td>
          </tr>
          <tr>
            <td>PKU2U protocol removed entirely</td>
            <td><span class="badge low">Low</span></td>
            <td>Deeply embedded in Windows' Negotiate/SPNEGO stack; used elsewhere (e.g. Entra-joined-to-Entra-joined RDP).</td>
          </tr>
          <tr>
            <td>Direction of travel is favorable</td>
            <td><span class="badge dash">—</span></td>
            <td>Microsoft is actively investing in cloud-only auth (Entra Kerberos for Azure Files, Entra sign-in for Arc VMs) — this isn't fighting the roadmap, just ahead of official support.</td>
          </tr>
        </tbody>
      </table>
    </div>

    <h3>Before shipping this to a customer</h3>
    <ul class="checklist">
      <li><span class="check-yes">✓</span> Do not present this as a Microsoft-supported architecture</li>
      <li><span class="check-yes">✓</span> Get written customer sign-off acknowledging the risk</li>
      <li><span class="check-yes">✓</span> Check for existing security baselines that touch PKU2U — the most likely failure mode</li>
      <li><span class="check-yes">✓</span> Define a rollback/exit plan up front (domain-join + Kerberos, or full migration to Azure Files)</li>
      <li><span class="check-yes">✓</span> Monitor Windows/Entra release notes as an ongoing task, not a one-time check</li>
    </ul>

    <div class="callout danger">
      <div class="icon">🚫</div>
      <div>
        None of this is officially supported by Microsoft CSS. Treat it as validated-internally, not vendor-endorsed — don't expect standard Microsoft support channels to troubleshoot issues with this specific combination.
      </div>
    </div>
  </section>

  <footer>
    <hr>
    Document reflects architecture and lab results validated in a real engagement. Part 2 (Azure File Sync) should be run through the same staged validation approach as Part 1 before relying on it for production access.
  </footer>
</div>
