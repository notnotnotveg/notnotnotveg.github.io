---
title: "Placeholder"
date: 2010-02-19T13:00:00-01:00
categories:
  - Tools
tags:
  - Web Sec
  - Research
toc: True
toc_label: "LNA"
toc_icon: "web"
---

### Test 1
test

<div style="display:flex; justify-content:center; margin: 1.5em 0;">
  <iframe
    width="480"
    height="270"
    src="https://www.youtube.com/embed/c56t7upa8Bk"
    title="Hans Zimmer — Time"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    allowfullscreen
    style="border-radius: 8px; border: 1px solid #222230;">
  </iframe>
</div>

test

### Test 2
test
<style>
.lna-flow {
  list-style: none;
  padding: 0;
  margin: 1.5em 0;
  font-family: 'Share Tech', sans-serif;
}
.lna-flow li {
  display: flex;
  gap: 16px;
  position: relative;
}
.lna-flow li + li {
  margin-top: 0;
}
.lna-flow .flow-spine {
  display: flex;
  flex-direction: column;
  align-items: center;
  flex-shrink: 0;
  width: 16px;
}
.lna-flow .flow-dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  border: 2px solid #3a3a50;
  background: #18181f;
  flex-shrink: 0;
  margin-top: 3px;
}
.lna-flow .flow-dot.accent  { background: #e8ff47; border-color: #e8ff47; box-shadow: 0 0 8px rgba(232,255,71,0.5); }
.lna-flow .flow-dot.warning { background: #ff9f43; border-color: #ff9f43; box-shadow: 0 0 8px rgba(255,159,67,0.5); }
.lna-flow .flow-dot.danger  { background: #ff4757; border-color: #ff4757; box-shadow: 0 0 8px rgba(255,71,87,0.5); }
.lna-flow .flow-line {
  width: 2px;
  flex: 1;
  background: #222230;
  min-height: 16px;
}
.lna-flow .flow-body {
  padding-bottom: 20px;
}
.lna-flow li:last-child .flow-body {
  padding-bottom: 0;
}
.lna-flow .flow-title {
  font-size: 0.95em;
  font-weight: 600;
  color: #ffffff;
  line-height: 1.3;
}
.lna-flow .flow-desc {
  font-size: 0.78em;
  color: #6b6b88;
  margin-top: 2px;
  line-height: 1.4;
}
</style>

<ul class="lna-flow">
  <li>
    <div class="flow-spine"><div class="flow-dot"></div><div class="flow-line"></div></div>
    <div class="flow-body">
      <div class="flow-title">Request initiated</div>
      <div class="flow-desc">fetch() or XHR call is made to an internal IP or localhost</div>
    </div>
  </li>
  <li>
    <div class="flow-spine"><div class="flow-dot"></div><div class="flow-line"></div></div>
    <div class="flow-body">
      <div class="flow-title">IP resolves</div>
      <div class="flow-desc">Hostname resolved; address classified as private or loopback</div>
    </div>
  </li>
  <li>
    <div class="flow-spine"><div class="flow-dot accent"></div><div class="flow-line"></div></div>
    <div class="flow-body">
      <div class="flow-title">LocalNetworkAccessCheck triggers</div>
      <div class="flow-desc">A TCP connection is opened to the target IP:Port to determine reachability</div>
    </div>
  </li>
  <li>
    <div class="flow-spine"><div class="flow-dot warning"></div><div class="flow-line"></div></div>
    <div class="flow-body">
      <div class="flow-title">Request deferred — user prompted</div>
      <div class="flow-desc">IPC message sent to browser process; prompt appears if port is open</div>
    </div>
  </li>
  <li>
    <div class="flow-spine"><div class="flow-dot"></div><div class="flow-line"></div></div>
    <div class="flow-body">
      <div class="flow-title">User decides</div>
      <div class="flow-desc">Allow, Block, or ignore the prompt</div>
    </div>
  </li>
  <li>
    <div class="flow-spine"><div class="flow-dot danger"></div></div>
    <div class="flow-body">
      <div class="flow-title">Request resumes or fails</div>
      <div class="flow-desc">Preflight sent on Allow; CORS error on Block; fetch remains Pending if ignored</div>
    </div>
  </li>
</ul>

test

### Test 3
test
<style>
.lna-browsers {
  width: 100%;
  border-collapse: collapse;
  font-family: 'Share Tech', sans-serif;
  margin: 1.5em 0;
}
.lna-browsers thead tr {
  background: #18181f;
  border-bottom: 1px solid #e8ff47;
}
.lna-browsers thead th {
  padding: 10px 14px;
  font-size: 9px;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: #6b6b88;
  text-align: left;
  font-weight: 400;
}
.lna-browsers tbody tr {
  border-bottom: 1px solid #222230;
  transition: background 0.15s;
}
.lna-browsers tbody tr:last-child {
  border-bottom: none;
}
.lna-browsers tbody tr:hover {
  background: #111115;
}
.lna-browsers td {
  padding: 11px 14px;
  font-size: 0.88em;
  color: #d4d4e8;
  vertical-align: middle;
}
.lna-browsers .browser-name {
  color: #ffffff;
  font-weight: 600;
  white-space: nowrap;
}
.lna-browsers .version {
  color: #6b6b88;
  font-size: 0.82em;
}
.lna-browsers .status-full {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 0.8em;
  color: #39ff7e;
}
.lna-browsers .status-full::before {
  content: '';
  width: 6px; height: 6px;
  border-radius: 50%;
  background: #39ff7e;
  box-shadow: 0 0 6px rgba(57,255,126,0.6);
  flex-shrink: 0;
}
.lna-browsers .status-trial {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 0.8em;
  color: #ff9f43;
}
.lna-browsers .status-trial::before {
  content: '';
  width: 6px; height: 6px;
  border-radius: 50%;
  background: #ff9f43;
  box-shadow: 0 0 6px rgba(255,159,67,0.6);
  flex-shrink: 0;
}
.lna-browsers .status-none {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 0.8em;
  color: #3a3a50;
}
.lna-browsers .status-none::before {
  content: '';
  width: 6px; height: 6px;
  border-radius: 50%;
  background: #3a3a50;
  flex-shrink: 0;
}
</style>

<table class="lna-browsers">
  <thead>
    <tr>
      <th>Browser</th>
      <th>LNA Support</th>
      <th>Version</th>
      <th>Status — Feb 2026</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><span class="browser-name">Chrome / Chromium</span></td>
      <td><span class="status-full">Supported</span></td>
      <td><span class="version">Chrome 142+ &nbsp;·&nbsp; Sept 2025</span></td>
      <td>Full LNA enforcement</td>
    </tr>
    <tr>
      <td><span class="browser-name">Microsoft Edge</span></td>
      <td><span class="status-full">Supported</span></td>
      <td><span class="version">Edge 142+ &nbsp;·&nbsp; 2025</span></td>
      <td>Full LNA enforcement</td>
    </tr>
    <tr>
      <td><span class="browser-name">Firefox</span></td>
      <td><span class="status-trial">In trials</span></td>
      <td><span class="version">Nightly only</span></td>
      <td>In development</td>
    </tr>
    <tr>
      <td><span class="browser-name">Safari</span></td>
      <td><span class="status-none">Not supported</span></td>
      <td><span class="version">N/A</span></td>
      <td>Partial / different model</td>
    </tr>
  </tbody>
</table>
test

### Test 4
test

<style>
.lna-timing {
  width: 100%;
  background: #0a0a0f;
  border: 1px solid #222230;
  border-radius: 10px;
  padding: 20px 22px 14px;
  font-family: 'JetBrains Mono', monospace;
}
.lna-timing .header {
  font-family: 'Share Tech', sans-serif;
  font-size: 14px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #6b6b88;
  margin-bottom: 16px;
}
.lna-timing .row {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 10px;
}
.lna-timing .row-label {
  width: 80px;
  flex-shrink: 0;
  font-family: 'Share Tech', sans-serif;
  line-height: 1.5;
}
.lna-timing .row-label .state { display: block; font-size: 13px; color: #6b6b88; }
.lna-timing .row-label .value-open  { font-size: 16px; color: #ff9f43; font-weight: 700; }
.lna-timing .row-label .value-closed { font-size: 16px; color: #39ff7e; font-weight: 700; }
.lna-timing .track-wrap { flex: 1; position: relative; }
.lna-timing .track {
  width: 100%; height: 32px;
  background: #18181f;
  border-radius: 4px;
  position: relative;
  overflow: visible;
}
.lna-timing .bar-closed {
  position: absolute; left: 0; top: 0;
  width: 1.5%; height: 100%;
  background: rgba(57,255,126,0.18);
  border: 1px solid rgba(57,255,126,0.45);
  border-radius: 4px;
}
.lna-timing .bar-closed-label {
  position: absolute; left: calc(1.5% + 8px);
  top: 50%; transform: translateY(-50%);
  font-size: 13px; color: #39ff7e; white-space: nowrap;
}
.lna-timing .bar-open {
  position: absolute; left: 0; top: 0;
  width: calc(100% - 2px); height: 100%;
  background: linear-gradient(90deg, rgba(255,159,67,0.30) 0%, rgba(255,159,67,0.18) 50%, rgba(255,159,67,0.08) 100%);
  border: 1px solid rgba(255,159,67,0.45);
  border-radius: 4px;
}
.lna-timing .bar-open-label {
  position: absolute; left: 10px;
  top: 50%; transform: translateY(-50%);
  font-size: 13px; color: #ff9f43; white-space: nowrap;
}
.lna-timing .arrow {
  position: absolute; right: -10px; top: 50%; transform: translateY(-50%);
  width: 0; height: 0;
  border-top: 6px solid transparent;
  border-bottom: 6px solid transparent;
  border-left: 9px solid #ff9f43;
}
.lna-timing .badge {
  flex-shrink: 0; padding: 3px 10px;
  border-radius: 4px; font-size: 13px; font-weight: 700; white-space: nowrap;
}
.lna-timing .badge-open  { background: rgba(255,159,67,0.1); border: 1px solid rgba(255,159,67,0.35); color: #ff9f43; }
.lna-timing .badge-closed { background: rgba(57,255,126,0.1); border: 1px solid rgba(57,255,126,0.35); color: #39ff7e; }
.lna-timing .axis-row { display: flex; align-items: flex-start; gap: 10px; margin-top: 4px; }
.lna-timing .axis-spacer { width: 80px; flex-shrink: 0; }
.lna-timing .axis-badge-spacer { width: 66px; flex-shrink: 0; }
.lna-timing .axis { flex: 1; position: relative; height: 32px; }
.lna-timing .axis::before {
  content: ''; position: absolute;
  top: 0; left: 0; right: 0; height: 1px; background: #3a3a50;
}
.lna-timing .tick {
  position: absolute; top: 0;
  display: flex; flex-direction: column; align-items: center;
  transform: translateX(-50%);
}
.lna-timing .tick-line { width: 1px; height: 5px; background: #3a3a50; }
.lna-timing .tick-label { font-family: 'Share Tech', sans-serif; font-size: 11px; color: #3a3a50; margin-top: 2px; white-space: nowrap; }
.lna-timing .tick-label.threshold-label { color: rgba(255,255,255,0.2); }
.lna-timing .threshold-line {
  position: absolute; left: calc(1.5% + 80px + 10px);
  top: 0; bottom: 0; width: 1px;
  border-left: 1px dashed rgba(255,255,255,0.2);
  pointer-events: none;
}
.lna-timing .note {
  margin-top: 14px; padding-top: 10px;
  border-top: 1px solid #222230;
  font-family: 'Share Tech', sans-serif;
  font-size: 13px; color: #3a3a50; line-height: 1.5;
}
</style>

<div class="lna-timing">
  <div class="header">Request Duration — Observed from Page Context</div>
  <div style="position: relative;">
    <div class="threshold-line"></div>
    <div class="row">
      <div class="row-label">
        <span class="state">Port</span>
        <span class="value-open">OPEN</span>
      </div>
      <div class="track-wrap">
        <div class="track">
          <div class="bar-open"></div>
          <div class="bar-open-label">pending... (LNA prompt shown, awaiting user)</div>
          <div class="arrow"></div>
        </div>
      </div>
      <div class="badge badge-open">&gt;&gt; ms</div>
    </div>
    <div class="row">
      <div class="row-label">
        <span class="state">Port</span>
        <span class="value-closed">CLOSED</span>
      </div>
      <div class="track-wrap">
        <div class="track">
          <div class="bar-closed"></div>
          <div class="bar-closed-label">RST — rejected instantly</div>
        </div>
      </div>
      <div class="badge badge-closed">&lt; 5ms</div>
    </div>
  </div>
  <div class="axis-row">
    <div class="axis-spacer"></div>
    <div class="axis">
      <div class="tick" style="left:0%"><div class="tick-line"></div><div class="tick-label">0</div></div>
      <div class="tick" style="left:1.5%"><div class="tick-line"></div><div class="tick-label threshold-label">~5ms</div></div>
      <div class="tick" style="left:25%"><div class="tick-line"></div><div class="tick-label">250ms</div></div>
      <div class="tick" style="left:50%"><div class="tick-line"></div><div class="tick-label">500ms</div></div>
      <div class="tick" style="left:75%"><div class="tick-line"></div><div class="tick-label">750ms</div></div>
      <div class="tick" style="left:100%"><div class="tick-line"></div><div class="tick-label">≥1s+</div></div>
    </div>
    <div class="axis-badge-spacer"></div>
  </div>
  <div class="note">
    Any response beyond a few milliseconds indicates an open port — the 2s abort timeout used in the PoC is a practical scan cutoff, not the detection threshold.
  </div>
</div>

tst3
