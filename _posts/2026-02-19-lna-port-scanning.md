---
title: "Browser-Based Port Scanning in the Age of LNA"
date: 2026-02-19T13:00:00-01:00
categories:
  - Tools
tags:
  - Web Sec
  - Research
  - LNA
toc: True
toc_label: "LNA"
toc_icon: "web"
---

Is it possible ? Yes.
How ?

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

Because in this case… timing is everything.

## Background

### The Pop-Up

Have you seen this prompt recently?

<a href="https://github.com/user-attachments/assets/4d1181e4-836a-454d-8ffd-f5f9d87864c9" class="image-popup">
  <img src="https://github.com/user-attachments/assets/4d1181e4-836a-454d-8ffd-f5f9d87864c9">
</a>


If you use a modern browser, there’s a good chance you’ve either seen it, or you’re about to.

This dialog was introduced as part of **Local Network Access (LNA)**. The premise is straightforward. Public websites should not be able to silently reach into your local network and talk to private resources, such as your router admin panel, your local dev server, your NAS, a Raspberry Pi on `192.168.x.x`. LNA draws a hard boundary between public IP space and private/loopback address space, and requires explicit user permission before a cross-boundary fetch is allowed to proceed.


### What LNA is meant to do

Under the hood, this boundary is enforced by a function such as `URLLoaderThrottle` (in Chrome and Chromium). When a public origin tries to access a of a less public IP the following sequence plays out:

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
  width: 20px;
}
.lna-flow .flow-dot {
  width: 14px;
  height: 14px;
  border-radius: 50%;
  border: 2px solid #3a3a50;
  background: #18181f;
  flex-shrink: 0;
  margin-top: 4px;
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
  padding-bottom: 24px;
}
.lna-flow li:last-child .flow-body {
  padding-bottom: 0;
}
.lna-flow .flow-title {
  font-size: 16px;
  font-weight: 600;
  color: #ffffff;
  line-height: 1.3;
}
.lna-flow .flow-desc {
  font-size: 13px;
  color: #9a9ab8;
  margin-top: 3px;
  line-height: 1.5;
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



Importantly  :
The fetch promise remains in a `pending` state throughout the entire pause. From the page's perspective, the request is simply waiting. From a timing perspective, that wait is information.


### LNA enabled browsers
As of February 2026, LNA is available on the following browsers :

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
  font-size: 12px;
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
  padding: 13px 14px;
  font-size: 15px;
  color: #d4d4e8;
  vertical-align: middle;
}
.lna-browsers .browser-name {
  color: #ffffff;
  font-weight: 600;
  white-space: nowrap;
}
.lna-browsers .version {
  color: #9a9ab8;
  font-size: 13px;
}
.lna-browsers .status-full {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 14px;
  color: #39ff7e;
}
.lna-browsers .status-full::before {
  content: '';
  width: 7px; height: 7px;
  border-radius: 50%;
  background: #39ff7e;
  box-shadow: 0 0 6px rgba(57,255,126,0.6);
  flex-shrink: 0;
}
.lna-browsers .status-trial {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 14px;
  color: #ff9f43;
}
.lna-browsers .status-trial::before {
  content: '';
  width: 7px; height: 7px;
  border-radius: 50%;
  background: #ff9f43;
  box-shadow: 0 0 6px rgba(255,159,67,0.6);
  flex-shrink: 0;
}
.lna-browsers .status-none {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 14px;
  color: #6b6b88;
}
.lna-browsers .status-none::before {
  content: '';
  width: 7px; height: 7px;
  border-radius: 50%;
  background: #6b6b88;
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


## The Finding 
### Port Scanning 

Here is the crux of the finding as part of this research. To decide _whether to show you the prompt at all_, the browser first has to check whether the private target's port is actually reachable. It does this by opening a TCP connection, before any permission is granted or denied.

The implication is quiet but significant. The port state has already been determined before the user has taken any action :
* Port **Closed** : the TCP handshake gets an RST almost instantly, and the fetch rejects in milliseconds. No LNA prompt is required.
* Port **Open** : the TCP handshake succeeds, the browser defers the request, the prompt appears, and the fetch sits in a pending state for as long as the user takes to decide, or indefinitely if they ignore it.

<style>
  .mermaid svg text,
  .mermaid svg .nodeLabel,
  .mermaid svg .edgeLabel,
  .mermaid svg .cluster-label,
  .mermaid svg span {
    font-family: 'Share Tech', sans-serif !important;
    font-size: 16px !important;
  }
</style>

<script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
<script>
mermaid.initialize({
  startOnLoad: true,
  theme: 'dark',
  flowchart: { useMaxWidth: false },
  themeVariables: {
    fontFamily: 'Share Tech, sans-serif',
    fontSize: '16px',
    primaryColor: '#111115',
    primaryBorderColor: '#444458',
    primaryTextColor: '#ffffff',
    lineColor: '#4a4a6a',
    edgeLabelBackground: '#0c0c0e'
  }
});
</script>

<div style="display:flex; justify-content:center; margin:1.5em 0;">
<div class="mermaid" style="min-width:500px;">
%%{init: {'theme': 'dark', 'themeVariables': {'fontFamily': 'Share Tech, sans-serif', 'fontSize': '16px'}}}%%
graph TD;
    id(Page Loaded) --> id1(Internal Resource Access Initiated)
    id1 --> id2(Port Probed)
    id2 -->|Port Open| id3(User Prompted)
    id2 -->|Port Closed| id4(Preflight fails - Immediately)
    id3 -->|Allow| id5(Preflight Sent)
    id3 -->|Deny| id6(Preflight fails CORS Error)
</div>
</div>

This creates a trivially observable timing differential:

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


The malicious page does not need the user to click Allow or Block. The timing delta between an immediate rejection and a prolonged pending state is sufficient to fingerprint port state.


### State of LNA Prompt vs. Port State

State of LNA Prompt vs. Port State

<div style="display:flex; justify-content:center; margin:1.5em 0;">

| Prompt State | Port State |
| --- | --- |
| "Allow" | Port can be probed |
| "Block" | Port cannot be probed |
| Pending prompt | Port can be probed |

</div>


Based on this  its safe to say that LNA may be intended to protect knowledge of such port states when the user "Blocks" LNA.


## Proof of Concept

### Basic Test

Start a local HTTP server on any port : 
```bash
python3 -m http.server 30000
```

Open a browser's developer console from any public-origin page. Paste the following. Importantlly, when the LNA prompt appears, **do not click Allow or Block**. Simply observe the console output.
```js
// Target port to probe
const port = 30000;

// Allows us to cancel the request after a timeout
const c = new AbortController();

// Record the start time
const t = Date.now();

// Abort the request after 2 seconds
setTimeout(() => c.abort(), 2000);

// If the request times out (AbortError), the port is open — closed ports fail instantly
fetch(`http://localhost:${port}`, { mode: "no-cors", signal: c.signal })
  .catch(e => console.log(e.name === "AbortError" ? `open (${Date.now() - t}ms)` : "closed"));
```

### Scaling to the Full Port Range
An optimized version can sweep the full 65,535 TCP port range using batched, concurrent fetches with tuned abort timing can be found here : 
https://raw.githubusercontent.com/notnotnotveg/notnotnotveg.github.io/refs/heads/master/raw-html-tests/scanner-all.html

**[Source](https://raw.githubusercontent.com/notnotnotveg/notnotnotveg.github.io/refs/heads/master/raw-html-tests/scanner-all.html) 
[Live demo](https://wiki.notveg.ninja/raw-html-tests/scanner-all.html)**

An example run:

<a href="https://github.com/user-attachments/assets/6222080b-187e-465e-98e5-098d777ea20f" class="image-popup">
  <img src="https://github.com/user-attachments/assets/6222080b-187e-465e-98e5-098d777ea20f">
</a>


### Implications 

Port scanning via browsers is not new. What LNA changes is the _quality_ of the signal. The LNA probe is a deliberate TCP handshake with a binary outcome, producing a clean, reliable timing split.

### Browser & User Fingerprinting
Different OS and software configurations expose different port signatures. Combined with existing fingerprinting signals, a port map assembled in seconds meaningfully contributes to cross-session tracking.

### Internal Network Reconnaissance
The graver scenario is enterprise. An employee whose browser bridges the public internet and an internal network is an ideal unwitting proxy. A malicious page can use LNA timing to probe RFC 1918 ranges to map live hosts and open services, with no installation, no privileges, and no interaction beyond a page load.

### The Prompt Is Not the Defence
It is worth stating directly: LNA's prompt was never intended to prevent port discovery. It was intended to prevent unauthorized data exchange with local resources. The port-state leakage described here is a side-effect of the mechanism used to decide whether to show the prompt, and not a flaw in the prompt itself.

## Summary 

LNA's TCP probe, the mechanism that decides _whether to show the prompt_, runs before the user sees anything. That probe leaks port state. Open ports stall; closed ports reject instantly. The delta is measurable from JavaScript with no special permissions.

Allow, Block, or ignore : it doesn't matter. By the time the prompt appears, the scan is already done.







