# Backend Engineering Route Implementation Plan

> **Status: complete.** All 20 tasks executed, 101 topics across 15 stages plus
> the capstone, integrated into the landing page and documented. What differed
> from the plan is recorded in section 10 of the design spec.

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a fourth PrepOps track at `backend/index.html` — a 15-stage, 101-topic practical learning route for backend engineering, plus a capstone project spec — integrated into the landing page, README and CLAUDE.md.

**Architecture:** One self-contained HTML file, no build step, matching the repo's existing tracks. Content is data: a `stages` array of topic objects rendered by a single inline script into collapsible topic cards. Reuses the `aspnet-core` page's shell (colour tokens, hero, sticky toolbar, 760px column, highlight.js, back-to-top) with a new card renderer and DSA-style progress ticks.

**Tech Stack:** Plain HTML5, CSS custom properties, vanilla ES2020, highlight.js 11.9.0 from cdnjs, Google Fonts (Bricolage Grotesque, Atkinson Hyperlegible, JetBrains Mono), `localStorage`. No package manager, no framework, no build.

## Global Constraints

Copied verbatim from the spec and `CLAUDE.md`. Every task's requirements implicitly include this section.

- Path: `backend/index.html`. Track name: **Backend engineering**. Kicker: **Server-side craft**.
- Colour tokens: `--t-backend:#0F766E` / `--t-backend-soft:#D7F0ED` (light), `#5ED3C4` / `#10332F` (dark). Page accent tokens: `--accent`, `--accent-soft`, `--accent-ink`, `--on-accent`.
- localStorage keys: `backend:done:v1` (ticked topic ids), `backend:filter:v1` (toolbar filter). Namespaced because all `harsh07may.github.io` sites share one origin.
- **Never rename a published topic or stage `id`.** They are anchors and progress keys; renaming silently discards a reader's ticks and breaks inbound links.
- Code policy: language-neutral pseudocode by default. Real code only where concrete: **Node primary**, **C# secondary**, plus real SQL, YAML, Dockerfile, nginx, shell. No framework is assumed by the route.
- `code` entries are inserted with `textContent`, so snippet source is written raw and unescaped. Code inside `what`, `how`, `tradeoff`, `use`, `exercise` is HTML and MUST escape `<`, `>`, `&` as `&lt;`, `&gt;`, `&amp;`.
- Content rules: sentence case headings, no emoji in content, no all-caps labels. Cross-reference other topics and tracks by name, never by number.
- Accuracy: nothing is labelled as a retrieved source unless it was actually opened. Node snippets are run before committing. Pseudocode is labelled as pseudocode.
- Never attribute a question or practice to a named company without a source that says so.
- A runtime error (for example an undefined constant) leaves the page empty below the toolbar, so the page is opened in a browser after every stage is added.
- Attribution footer on every commit:
  `Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>` and
  `Claude-Session: https://claude.ai/code/session_018xuWqRy7N5nyjcvrnEgasH`
- Do NOT open a pull request. Commit and push to `claude/dazzling-mccarthy-0px7mf` only.

## Verification cycle

This repo has no test framework. The test cycle for every task is these four commands plus a browser check. "Run the checks" in any step below means exactly this:

```bash
cd /home/user/interview-prep
python3 -c "import html.parser; [html.parser.HTMLParser().feed(open(f).read()) for f in ['index.html','aspnet-core/index.html','react-typescript/index.html','dsa/index.html','backend/index.html']]; print('html ok')"
```

```bash
cd /home/user/interview-prep && python3 -c "import re; s=open('backend/index.html').read(); [open(f'/tmp/b{i}.js','w').write(b) for i,b in enumerate(re.findall(r'<script>(.*?)</script>', s, re.S))]" && for f in /tmp/b*.js; do node --check "$f" || exit 1; done && echo "script ok"
```

```bash
cd /home/user/interview-prep && grep -o 'id:"[^"]*"' backend/index.html | sort | uniq -d
```
Expected: prints nothing. Any output is a duplicate stage or topic id and MUST be fixed before committing.

```bash
cd /home/user/interview-prep && node -e "
const fs=require('fs'); const s=fs.readFileSync('backend/index.html','utf8');
const m=s.match(/<script>([\s\S]*?)<\/script>/)[1];
const lit=m.slice(m.indexOf('const stages'), m.indexOf('const DEPTH'));
const stages=new Function(lit+'; return stages;')();
let n=0, bad=[];
for (const st of stages) for (const t of st.topics||[]) { n++;
  for (const f of ['id','name','depth','line','what','use','concepts','how']) if (!t[f]) bad.push(t.id+': missing '+f);
  if (!['understand','build'].includes(t.depth)) bad.push(t.id+': bad depth');
}
console.log('topics:',n); if(bad.length){console.error(bad.join('\n'));process.exit(1)} console.log('fields ok');
"
```
Expected: a topic count and `fields ok`. This is the structural test — it fails loudly on a topic missing a required field, which is the most likely content mistake.

Browser check (after every content task): `python3 -m http.server 8000`, open the stage, confirm it renders, all four filters work, a tick survives a reload, and the page is readable at phone width in light and dark mode.

---

### Task 1: Page shell, renderer and one exemplar topic

The critical task. It establishes every interface the fifteen content tasks consume. It ends with exactly one topic rendered end to end, so the renderer is proven before any bulk content is written.

**Files:**
- Create: `backend/index.html`

**Interfaces:**
- Consumes: nothing.
- Produces: the `stages` array contract that Tasks 2-16 append to, and the CSS classes they rely on.

  Stage object:
  ```js
  { num:1, id:"foundations", phase:1, name:"Runtime foundations",
    intro:"One line, sentence case, no trailing period omitted.",
    topics:[ /* topic objects */ ],
    after:[ ["Short claim","What you can now do"] ],
    sources:`<p>HTML. Says which pages were opened and which are references.</p>` }
  ```

  Topic object. `id`, `name`, `depth`, `core`, `line`, `what`, `use`, `concepts`, `how` are REQUIRED. `code`, `tradeoff`, `pitfall`, `exercise`, `refs` are optional:
  ```js
  { id:"idempotency", name:"Idempotency and safe retries",
    depth:"build", core:true,
    line:"Make a repeated request produce one effect, not two.",
    what:`<p>HTML, two to four sentences.</p>`,
    use:`<p>HTML, one concrete scenario.</p>`,
    concepts:[ ["Idempotency key","A client-supplied id that names the attempt, not the request."] ],
    how:`<p>HTML.</p>`,
    code:[ {label:"Node", lang:"js", code:`raw, unescaped`} ],
    tradeoff:`<p>HTML.</p>`,
    pitfall:"Plain text, one or two sentences.",
    exercise:`<p>HTML.</p>`,
    refs:[ ["RFC 9110","https://www.rfc-editor.org/rfc/rfc9110"] ] }
  ```

  `lang` accepts `js`, `cs`, `sql`, `yaml`, `bash`, `json`, `dockerfile`, `nginx`, `text`. `dockerfile`, `nginx` and `text` render unhighlighted on purpose — highlight.js 11.9.0's common bundle does not carry the first two, and mis-highlighted pseudocode is worse than plain.

- [x] **Step 1: Copy the shell from the React track**

The React track is the newest and cleanest of the two data-driven pages.

```bash
cd /home/user/interview-prep && mkdir -p backend && cp react-typescript/index.html backend/index.html
```

- [x] **Step 2: Replace the head**

In `backend/index.html`, replace `<title>`, both meta descriptions, the og tags and the favicon emoji:

```html
<title>Backend engineering route</title>
<meta name="description" content="A practical backend engineering route: runtime and networking foundations, API design, databases, testing, security, caching, concurrency, messaging, reliability, observability, deployment and the distributed systems ideas you actually meet.">
<meta property="og:title" content="Backend engineering route">
<meta property="og:description" content="Fifteen stages from the request lifecycle to sharding, each topic with what it is, where you use it, how to build it and an exercise.">
<meta property="og:type" content="website">
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🧱</text></svg>">
```

Keep the existing `<link>` font tags unchanged — the same three families are used.

- [x] **Step 3: Recolour all three colour blocks**

Replace the accent tokens in the light `:root`, the `@media (prefers-color-scheme: dark)` block and `:root[data-theme="dark"]`. Light:

```css
--accent:#0F766E; --accent-soft:#D7F0ED; --accent-ink:#0B5952; --on-accent:#FFFFFF;
```

Dark (both dark blocks):

```css
--accent:#5ED3C4; --accent-soft:#10332F; --accent-ink:#A5E8DE; --on-accent:#0F141C;
```

Leave `--fresher`, `--mid`, `--senior`, `--trap-bg`, `--trap-line`, `--code-bg` and the hljs token colours alone; the level pills are repurposed as depth pills and the trap box as the pitfall box.

- [x] **Step 4: Add the new CSS**

Append to the `<style>` block, after the existing `.lvl` rules. These are the only new classes; everything else reuses the copied shell.

```css
/* Stages */
.stage{margin:72px 0 0;padding-top:40px;border-top:2px solid var(--line)}
.stage:first-child{border-top:none;padding-top:8px;margin-top:40px}
.stage-progress{font-size:.9rem;color:var(--muted);margin:10px 0 0}

/* Topic card: tick column + collapsible card */
.topic{display:grid;grid-template-columns:auto 1fr;gap:10px;align-items:start;margin:0 0 10px}
.t-check{padding-top:17px;cursor:pointer;display:grid;place-items:center}
.t-check input{position:absolute;opacity:0;width:0;height:0}
.t-check span{display:block;width:21px;height:21px;border:2px solid var(--line);border-radius:6px;background:var(--surface)}
.t-check input:checked + span{background:var(--accent);border-color:var(--accent)}
.t-check input:checked + span::after{content:"";display:block;width:5px;height:10px;margin:3px auto 0;border-right:2.5px solid var(--on-accent);border-bottom:2.5px solid var(--on-accent);transform:rotate(45deg)}
.t-check input:focus-visible + span{outline:3px solid var(--accent);outline-offset:3px}
.topic.done .t-name{text-decoration:line-through;text-decoration-color:var(--muted)}
.topic.done details.topic-card{opacity:.72}

details.topic-card{background:var(--surface);border:1px solid var(--line);border-radius:14px;min-width:0;transition:border-color .15s}
details.topic-card[open]{border-color:var(--accent)}
details.topic-card > summary{list-style:none;cursor:pointer;padding:14px 18px;display:grid;grid-template-columns:1fr auto;gap:4px 14px;align-items:start}
details.topic-card > summary::-webkit-details-marker{display:none}
.t-name{font-weight:700;font-size:1.05rem;line-height:1.35}
.t-line{color:var(--muted);font-size:.95rem;grid-column:1}
.t-meta{display:flex;align-items:center;gap:8px;grid-row:1 / span 2}
.t-body{padding:0 18px 18px;border-top:1px dashed var(--line)}
.t-h{font-family:var(--display);font-weight:700;font-size:1rem;letter-spacing:.005em;color:var(--accent-ink);margin:18px 0 0}
.t-body p{margin:8px 0 0}
.t-body ul,.t-body ol{margin:8px 0 0;padding-left:1.3em}
.t-body li{margin:4px 0}
.t-use{background:var(--accent-soft);border-radius:12px;padding:2px 16px 14px;margin:16px 0 0}
.t-use .t-h{margin-top:14px}
dl.t-concepts{margin:8px 0 0;display:grid;grid-template-columns:minmax(130px,210px) 1fr;gap:0}
dl.t-concepts dt,dl.t-concepts dd{padding:9px 0;border-top:1px solid var(--line);margin:0}
dl.t-concepts dt{font-weight:700;padding-right:14px}
@media (max-width:560px){dl.t-concepts{grid-template-columns:1fr}dl.t-concepts dd{border-top:none;padding-top:0}}
figure.snip{margin:14px 0 0}
figure.snip figcaption{font-size:.8rem;font-weight:700;color:var(--muted);margin:0 0 4px}
figure.snip pre{margin:0}

/* Depth and core pills reuse the level pill shape */
.lvl.build{background:var(--senior-bg);color:var(--senior)}
.lvl.understand{background:var(--accent-soft);color:var(--accent-ink)}
.lvl.core{background:var(--mid-bg);color:var(--mid)}
@media (max-width:560px){.t-meta{flex-wrap:wrap;justify-content:flex-end}}
```

- [x] **Step 5: Replace the hero**

Six phase dots, not fifteen stages — fifteen would be illegible. Replace the whole `<header class="wrap hero">` block:

```html
<header class="wrap hero">
  <a class="home-link" href="../">All tracks</a>
  <h1>Backend engineering route</h1>
  <p>What a backend engineer needs to build, debug, operate and scale a real service. Fifteen stages in order, each topic with what it is, where you use it, how to build it, and something to build.</p>

  <nav class="track" aria-label="Route phases">
    <ol>
      <li class="done"><a href="#foundations"><span class="dot">1</span>Foundations</a></li>
      <li class="done"><a href="#apis"><span class="dot">2</span>Build the service</a></li>
      <li class="done"><a href="#security"><span class="dot">3</span>Secure and fast</a></li>
      <li class="done"><a href="#concurrency"><span class="dot">4</span>Async and integrate</a></li>
      <li class="done"><a href="#reliability"><span class="dot">5</span>Operate</a></li>
      <li class="done"><a href="#distributed"><span class="dot">6</span>Scale</a></li>
    </ol>
  </nav>
  <p class="track-note">Six phases, fifteen stages. Work in order: each stage assumes the ones before it. Short on time? Filter to <em>Core only</em> for the spine of the route.</p>
</header>
```

- [x] **Step 6: Replace the toolbar**

Four filters instead of three levels, and an overall progress readout:

```html
<div class="toolbar">
  <div class="wrap">
    <div class="chips" role="group" aria-label="Filter topics">
      <button class="chip" data-filter="all" aria-pressed="true">All topics</button>
      <button class="chip" data-filter="core" aria-pressed="false">Core only</button>
      <button class="chip" data-filter="build" aria-pressed="false">Build</button>
      <button class="chip" data-filter="understand" aria-pressed="false">Understand</button>
    </div>
    <span class="spacer"></span>
    <span class="overall-progress" aria-live="polite"></span>
    <button class="ghost" id="toggleAll">Open all topics</button>
  </div>
</div>
```

Add to the `<style>` block: `.overall-progress{font-size:.9rem;color:var(--muted)}`

- [x] **Step 7: Replace the "How to use this page" section**

```html
<section class="howto">
  <h2>How to use this page</h2>
  <div class="levels">
    <span class="lvl core">Core · the spine, do these first</span>
    <span class="lvl build">Build · implement it yourself</span>
    <span class="lvl understand">Understand · know the trade-off, reach for a library</span>
  </div>
  <p>Every topic has the same parts: what it is and why it matters, a concrete case, the core concepts, how to build it, the trade-offs, the pitfall people hit, and an exercise. Tick a topic when you have built or can explain it.</p>
  <p>Code is pseudocode unless a snippet is labelled with a language. Real snippets are Node, with C# where a second language makes the point clearer, plus real SQL and config. Nothing here assumes a particular framework. The capstone at the end composes the exercises into one service.</p>
  <nav class="jump" id="jump" aria-label="Jump to stage"></nav>
</section>
```

- [x] **Step 8: Replace the footer**

```html
<footer><p>That's the route. The capstone is the real test: build the order service end to end, then come back and tick the topics you had to reread.</p><p class="foot-links"><a href="../">All tracks</a><a href="../dsa/">DSA patterns</a><a href="https://github.com/harsh07may/interview-prep">Source on GitHub</a></p></footer>
```

- [x] **Step 9: Replace the main script with the renderer**

Delete the entire first inline `<script>` (the React `topics` data and renderer) and write this in its place. This is the whole engine; Tasks 2-16 only add stage objects to `stages`.

```js
const stages = [
  { num:1, id:"foundations", phase:1, name:"Runtime foundations",
    intro:"What a running backend is made of, before any framework.",
    topics:[
      { id:"request-path", name:"The path of a request, end to end",
        depth:"understand", core:true,
        line:"Every hop between a client's click and your handler, and what can go wrong at each.",
        what:`<p>A request crosses name resolution, a TCP connection, a TLS handshake, usually a proxy or load balancer, your process, and whatever your code calls, before anything comes back. Most production confusion comes from not knowing which hop failed: a 504 from a proxy and a timeout in your client look the same from a log line but have different fixes.</p><p>This topic is the map. Each hop is covered in depth later, and this is where you learn which stage covers which.</p>`,
        use:`<p>An endpoint "times out" for one customer. Knowing the path tells you what to ask: does DNS resolve for them, does the TLS handshake complete, did the proxy log the request at all, did your handler start, did the query finish. Four of those five rule out your code entirely.</p>`,
        concepts:[
          ["Resolution","The client turns a hostname into an address. Cached at several layers, with its own TTL, which is why a DNS change does not take effect everywhere at once."],
          ["Connection","A TCP handshake costs a round trip, TLS costs one or two more. Reusing connections is the single cheapest latency win, covered in the networking stage."],
          ["The edge","A proxy or load balancer terminates TLS, picks a backend, and applies its own timeouts and body limits — separately from yours."],
          ["Your process","One request occupies a thread, a task, or an event-loop turn. What that costs decides how many you can serve at once."],
          ["Downstream","Your database, cache and the APIs you call. Each is a hop with its own timeout, and each can fail while the rest succeed."]
        ],
        how:`<p>Trace one real request through your own stack and write down the hops with a measured number against each. Curl's timing fields give you the client-side split without any tooling:</p>`,
        code:[ {label:"Where the time actually goes", lang:"bash", code:`curl -w '
dns      %{time_namelookup}s
connect  %{time_connect}s
tls      %{time_appconnect}s
sent     %{time_pretransfer}s
first    %{time_starttransfer}s
total    %{time_total}s
' -o /dev/null -s https://example.com/api/orders`} ],
        pitfall:"Treating a slow response as an application problem before splitting the timing. A handshake that takes 300ms on every call, because connections are not being reused, is invisible in application metrics and obvious in these numbers.",
        exercise:`<p>Run the command above against one endpoint of a service you work on, from a machine outside its network. Write down the six numbers. Then run it twice in a row on a connection that is reused and note which numbers collapse to zero. Keep the figures — the caching and performance stage asks you to compare against them.</p>`,
        refs:[ ["curl: time_* variables","https://curl.se/docs/manpage.html"] ] }
    ],
    after:[ ["Read a request's timing","You can split a slow request into hops and say which one is slow."] ],
    sources:`<p>References, not retrieved while writing: the curl manual for the timing variables. The hop-by-hop behaviour described here is standard and is covered in depth, with sources, in the networking and the edge stage.</p>` }
];

const DEPTH = { understand:"Understand", build:"Build" };
const chev = '<svg class="chev" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M6 9l6 6 6-6"/></svg>';
const content = document.getElementById('content');
const esc = (s) => String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');

function renderTopic(t){
  const wrap = document.createElement('div');
  wrap.className = 'topic';
  wrap.id = t.id;
  wrap.dataset.topic = t.id;
  wrap.dataset.depth = t.depth;
  wrap.dataset.core = String(!!t.core);

  let body = `<h4 class="t-h">What and why</h4>${t.what}`;
  body += `<div class="t-use"><h4 class="t-h">In practice</h4>${t.use}</div>`;
  body += `<h4 class="t-h">Core concepts</h4><dl class="t-concepts">`
        + t.concepts.map(([k,v]) => `<dt>${k}</dt><dd>${v}</dd>`).join('') + `</dl>`;
  body += `<h4 class="t-h">How to build it</h4>${t.how}`;
  (t.code || []).forEach(() => { body += `<figure class="snip"><figcaption></figcaption><pre><code></code></pre></figure>`; });
  if (t.tradeoff) body += `<h4 class="t-h">Trade-offs</h4>${t.tradeoff}`;
  if (t.pitfall) body += `<div class="trap"><p><span class="label">Pitfall:</span> ${t.pitfall}</p></div>`;
  if (t.exercise) body += `<details class="reveal"><summary>Exercise</summary>${t.exercise}</details>`;
  if (t.refs) body += `<p class="src">References: ` + t.refs.map(([l,u]) => `<a href="${u}" target="_blank" rel="noopener">${l}</a>`).join(', ') + `</p>`;

  const pills = (t.core ? `<span class="lvl core">Core</span>` : '')
              + `<span class="lvl ${t.depth}">${DEPTH[t.depth]}</span>`;
  wrap.innerHTML =
    `<label class="t-check"><input type="checkbox" aria-label="Mark ${esc(t.name)} as done"><span></span></label>`
  + `<details class="topic-card"><summary><span class="t-name">${t.name}</span>`
  + `<span class="t-meta">${pills}${chev}</span><span class="t-line">${t.line}</span></summary>`
  + `<div class="t-body">${body}</div></details>`;

  (t.code || []).forEach((c, i) => {
    const fig = wrap.querySelectorAll('figure.snip')[i];
    fig.querySelector('figcaption').textContent = c.label;
    const el = fig.querySelector('code');
    el.textContent = c.code;
    if (c.lang) el.dataset.lang = c.lang;
  });
  return wrap;
}

stages.forEach(s => {
  const sec = document.createElement('section');
  sec.className = 'stage';
  sec.id = s.id;
  sec.dataset.stage = String(s.num);
  sec.innerHTML = `<p class="phase-title">Stage ${s.num}</p><h2 class="topic-title">${s.name}</h2>`
                + `<p class="topic-intro">${s.intro}</p><p class="stage-progress" aria-live="polite"></p>`;
  if (s.custom) sec.insertAdjacentHTML('beforeend', s.custom);
  (s.topics || []).forEach(t => sec.appendChild(renderTopic(t)));
  if (s.after) {
    const a = document.createElement('section');
    a.className = 'revise';
    a.innerHTML = `<h2>After this stage you can</h2><p>If you can do these, move on.</p><dl>`
                + s.after.map(([k,v]) => `<dt>${k}</dt><dd>${v}</dd>`).join('') + `</dl>`;
    sec.appendChild(a);
  }
  if (s.sources) {
    const d = document.createElement('details');
    d.className = 'sources';
    d.innerHTML = `<summary>Sources for this stage</summary>${s.sources}`;
    sec.appendChild(d);
  }
  content.appendChild(sec);
});

document.getElementById('jump').innerHTML =
  stages.map(s => `<a href="#${s.id}">${s.num}. ${s.name}</a>`).join('');

// Progress: ticked topic ids, in this browser only.
const KEY = "backend:done:v1";
let done = new Set();
try { done = new Set(JSON.parse(localStorage.getItem(KEY) || "[]")); } catch (e) {}
const saveDone = () => { try { localStorage.setItem(KEY, JSON.stringify([...done])); } catch (e) {} };

function refreshProgress(){
  const all = document.querySelectorAll('.topic');
  const n = [...all].filter(el => done.has(el.dataset.topic)).length;
  const overall = document.querySelector('.overall-progress');
  if (overall) overall.textContent = `${n} of ${all.length} done`;
  document.querySelectorAll('.stage').forEach(st => {
    const t = st.querySelectorAll('.topic');
    const k = [...t].filter(el => done.has(el.dataset.topic)).length;
    const el = st.querySelector('.stage-progress');
    if (el) el.textContent = `${k} of ${t.length} done in this stage`;
  });
}

document.querySelectorAll('.topic').forEach(el => {
  const box = el.querySelector('input[type=checkbox]');
  box.checked = done.has(el.dataset.topic);
  el.classList.toggle('done', box.checked);
  box.addEventListener('change', () => {
    box.checked ? done.add(el.dataset.topic) : done.delete(el.dataset.topic);
    el.classList.toggle('done', box.checked);
    saveDone(); refreshProgress();
  });
});

// Filter: all | core | build | understand
const chips = document.querySelectorAll('.chip');
function applyFilter(mode){
  chips.forEach(c => c.setAttribute('aria-pressed', String(c.dataset.filter === mode)));
  document.querySelectorAll('.topic').forEach(el => {
    const show = mode === 'all' ? true
               : mode === 'core' ? el.dataset.core === 'true'
               : el.dataset.depth === mode;
    el.classList.toggle('hidden', !show);
  });
  document.querySelectorAll('.stage').forEach(st => {
    st.classList.toggle('hidden', st.querySelectorAll('.topic:not(.hidden)').length === 0);
  });
  try { localStorage.setItem('backend:filter:v1', mode); } catch (e) {}
}
chips.forEach(c => c.addEventListener('click', () => applyFilter(c.dataset.filter)));
let savedFilter = 'all';
try { savedFilter = localStorage.getItem('backend:filter:v1') || 'all'; } catch (e) {}
if (!['all','core','build','understand'].includes(savedFilter)) savedFilter = 'all';
applyFilter(savedFilter);
refreshProgress();

// Open / close all
const toggle = document.getElementById('toggleAll');
toggle.addEventListener('click', () => {
  const open = toggle.dataset.open !== 'true';
  document.querySelectorAll('.topic:not(.hidden) details.topic-card').forEach(d => d.open = open);
  toggle.dataset.open = String(open);
  toggle.textContent = open ? 'Close all topics' : 'Open all topics';
});

// Back to top
const toTop = document.getElementById('toTop');
const reduceMotion = window.matchMedia && window.matchMedia('(prefers-reduced-motion: reduce)').matches;
function updateToTop(){
  const show = window.scrollY > 700;
  toTop.classList.toggle('show', show);
  toTop.tabIndex = show ? 0 : -1;
}
window.addEventListener('scroll', updateToTop, { passive: true });
updateToTop();
toTop.addEventListener('click', () => {
  window.scrollTo({ top: 0, behavior: reduceMotion ? 'auto' : 'smooth' });
  const first = document.querySelector('.hero h1');
  if (first) { first.setAttribute('tabindex', '-1'); first.focus({ preventScroll: true }); }
});

// Open a collapsed card when a link or the URL points at it.
(function(){
  function openTo(id){
    const el = id && document.getElementById(id);
    if (!el) return;
    const card = el.querySelector(':scope > details.topic-card');
    if (card) card.open = true;
    for (let d = el.closest('details'); d; d = d.parentElement && d.parentElement.closest('details')) d.open = true;
  }
  document.addEventListener('click', e => {
    const a = e.target.closest('a[href^="#"]');
    if (a) openTo(decodeURIComponent(a.getAttribute('href').slice(1)));
  });
  window.addEventListener('hashchange', () => openTo(decodeURIComponent(location.hash.slice(1))));
  if (location.hash) {
    const id = decodeURIComponent(location.hash.slice(1));
    openTo(id);
    const t = document.getElementById(id);
    if (t) t.scrollIntoView();
  }
})();
```

- [x] **Step 10: Replace the highlight.js language map**

Replace the second inline script's `map` and `langFor` with this. There is no default language: an unlabelled snippet is pseudocode and must stay unhighlighted.

```js
(function(){
  if (!window.hljs) return;
  const map = { js:'javascript', cs:'csharp', sql:'sql', yaml:'yaml', bash:'bash', json:'json' };
  document.querySelectorAll('pre code').forEach(el => {
    const lang = map[el.dataset.lang];
    if (!lang) return;                 // dockerfile, nginx, text and pseudocode stay plain
    el.classList.add('language-' + lang);
    try { hljs.highlightElement(el); } catch (e) {}
  });
})();
```

- [x] **Step 11: Run the checks**

Run all four verification commands. Expected: `html ok`, `script ok`, no duplicate ids, `topics: 1` and `fields ok`.

- [x] **Step 12: Browser check**

```bash
cd /home/user/interview-prep && python3 -m http.server 8000
```

Open `http://localhost:8000/backend/`. Confirm: the hero renders six phase dots; stage 1 renders one card; the card opens; the pitfall box, exercise disclosure and the bash snippet all appear, the snippet highlighted; ticking the box strikes the name and the two progress readouts change; a reload keeps the tick; `Core only` keeps the card and `Understand` keeps it while `Build` hides the card and the whole stage; dark mode is legible; nothing scrolls horizontally at 390px wide.

- [x] **Step 13: Commit**

```bash
cd /home/user/interview-prep && git add backend/index.html && git commit -F - <<'MSG'
Add the backend engineering route shell and renderer

A fourth PrepOps track at backend/index.html. Reuses the React track's
shell and recolours it teal, but renders a new topic card: what and why,
a concrete case, core concepts, how to build it, trade-offs, the pitfall,
and an exercise.

Brings the DSA track's progress ticks into the data-driven engine, and
replaces the level filter with four: all, core only, build, understand.
One topic is included so the renderer is proven end to end.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_018xuWqRy7N5nyjcvrnEgasH
MSG
git push
```

---

## Tasks 2-16: the content stages

Each content task adds exactly one stage object to the `stages` array in
`backend/index.html`, immediately before the closing `];`, and nothing else.
The topic object contract is defined in Task 1's **Interfaces** block — reread it
before starting any of these tasks; it is the only place the field names and the
`lang` values are specified.

**Every content task has the same five steps.** They are written out once here
rather than repeated fifteen times, because they are identical in every task:

- [x] **Step A: Write the stage object**

Append one stage object to `stages`, using the `id`, `name`, `depth` and `core`
values from that task's table verbatim. Write every required field for every
topic: `line`, `what`, `use`, `concepts`, `how`. Add `code`, `tradeoff`,
`pitfall`, `exercise` and `refs` where the task's requirements call for them.

- [x] **Step B: Run every runnable snippet**

Any snippet with `lang:"js"` is run under Node before it goes in the file, from
the scratchpad directory, and its real output is what the prose claims. Any
snippet with `lang:"sql"` is run against a real engine if one is available in the
container (`psql`, `sqlite3`); if not, it is marked illustrative in the prose.
A snippet that is pseudocode carries no `lang` at all, and the prose says it is
pseudocode.

- [x] **Step C: Run the checks**

Run all four commands from the **Verification cycle** section above.
Expected: `html ok`, `script ok`, no duplicate ids, `fields ok`, and a `topics:`
count equal to the running total in that task's heading.

- [x] **Step D: Browser check**

Serve the page and open the new stage. Confirm every topic card renders and
opens, every snippet is present and highlighted as its language (or deliberately
plain), the concepts list is a two-column grid on desktop and one column at
phone width, the "After this stage you can" block renders, and all four filters
behave. A stage that renders nothing below the toolbar means a JavaScript error:
open the console and fix it before committing.

- [x] **Step E: Commit and push**

```bash
cd /home/user/interview-prep && git add backend/index.html && git commit -F - <<'MSG'
Add stage N of the backend route: <stage name>

<One paragraph: what the stage covers and the ordering reason it sits here.>

<One paragraph: the notable judgement calls, for example what was deliberately
framed as "usually do not do this", and which snippets were run.>

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_018xuWqRy7N5nyjcvrnEgasH
MSG
git push
```

### Content quality bar

Applies to every topic in every content task. A topic that fails any of these is
not done:

1. `what` is two to four sentences and names the problem the topic solves. Not a
   definition of the term.
2. `use` is one specific scenario with a number, a failure, or a decision in it.
   "Useful for scalability" fails. "A payment retry charges the customer twice"
   passes.
3. `concepts` is three to six pairs. Each value is one sentence that would let a
   reader use the term correctly, not a gloss of it.
4. `how` tells the reader what to actually do, in order, and names the real tools
   people use. Where two approaches exist, `tradeoff` says when to pick each.
5. `pitfall` is the mistake engineers actually make, stated plainly. Not a trick.
6. `exercise` is present where building the thing teaches it, and is sized to an
   evening. Where an exercise would be busywork, omit it rather than invent one.
7. No topic restates another. If two topics overlap, the later one says "covered
   in <stage name>" and moves on.
8. Nothing is attributed to a company. Nothing is labelled as a retrieved source
   unless it was opened.

---

### Task 2: Stage 1 — Runtime foundations (running total: 8 topics)

**Files:** Modify: `backend/index.html` (the `stages` array)

**Interfaces:** Consumes the topic contract from Task 1. Produces stage id `foundations`.

Task 1 already shipped `request-path`. This task adds the remaining seven to the
same stage object and fills in `after` and `sources` properly.

| id | name | depth | core |
|---|---|---|---|
| `request-path` | The path of a request, end to end | understand | yes (already written) |
| `http-semantics` | HTTP semantics: methods, status codes, headers | build | yes |
| `concurrency-models` | Server concurrency models and what one request costs | understand | no |
| `linux-basics` | Linux for backend engineers: processes, signals, file descriptors | understand | no |
| `config-secrets` | Configuration and secrets | build | yes |
| `layering` | Layering and dependency boundaries | build | yes |
| `serialization` | Serialization and data contracts | build | no |
| `statelessness` | Statelessness and where state actually leaks | build | no |

Stage-specific requirements:

- `http-semantics`: must state which methods are safe, which are idempotent, and
  that the two are different. Must cover 400 vs 422, 401 vs 403, 409, 429, and
  why returning 200 with an error body breaks clients and caches. Cite RFC 9110.
- `concurrency-models`: thread-per-request, event loop, and worker pool, with the
  memory and context-switch cost of each, and the rule that a blocking call on an
  event loop stalls every other request. Must point forward to the concurrency
  stage by name for async mechanics, and not duplicate it.
- `linux-basics`: processes and exit codes, SIGTERM vs SIGKILL, file descriptors
  and `ulimit -n`, and what `ss`, `lsof`, `top` and `/proc/<pid>/limits` tell you.
  Must point forward to graceful shutdown in the reliability stage by name.
- `config-secrets`: config from the environment, the difference between config and
  secrets, why secrets in images and repos are the common breach, and secret
  managers. Include a `bash`/`yaml` snippet of a twelve-factor style config
  surface and one example of a startup-time validation failure.
- `layering`: handler, service, repository; where validation lives; why a database
  row type reaching a handler makes both untestable; dependency injection as the
  seam tests use. One pseudocode snippet showing the dependency direction.
- `serialization`: UTC always, decimals not floats for money, null vs absent vs
  empty, additive-only field changes, and the cost of large payloads. Include a
  `js` snippet that demonstrates the float money bug, run under Node, with its
  real output in the prose.
- `statelessness`: in-process caches, sticky sessions, in-process schedulers, and
  files written to local disk, each with the failure it causes when a second
  instance appears. Must point forward to horizontal scaling by name.

### Task 3: Stage 2 — Networking and the edge (running total: 12)

**Files:** Modify: `backend/index.html`

**Interfaces:** Produces stage id `network`.

| id | name | depth | core |
|---|---|---|---|
| `tls-certs` | TLS, certificates and where encryption ends | understand | yes |
| `connections` | Connections: keep-alive, pooling and HTTP/2 multiplexing | build | yes |
| `reverse-proxy` | Reverse proxies and load balancers at the edge | build | no |
| `realtime` | Real-time: WebSockets, SSE and long polling | understand | no |

Stage-specific requirements:

- `tls-certs`: handshake round trips, the chain and why an intermediate is the
  usual breakage, SNI, expiry and renewal, and TLS termination at the edge meaning
  traffic behind it may be plaintext. Include the `bash` one-liner that prints a
  certificate's expiry (`openssl s_client ... | openssl x509 -noout -dates`).
  Point to service-to-service auth in the security stage for mTLS.
- `connections`: keep-alive, the cost of a fresh handshake per call, client-side
  connection pools and their size, HTTP/1.1 head-of-line blocking, what HTTP/2
  multiplexing changes, and HTTP/3 in one honest paragraph. Must say explicitly
  that domain sharding is now an anti-pattern.
- `reverse-proxy`: TLS termination, `X-Forwarded-For` and why trusting it blindly
  is a spoofing hole, two layers of timeout and body limits, and 502 vs 503 vs 504
  as a diagnosis table. Include a real `nginx` config block (unhighlighted by
  design) with proxy timeouts and `client_max_body_size`.
- `realtime`: long polling, SSE and WebSockets compared on direction, proxy
  support, reconnection and auth-on-upgrade. `tradeoff` must say SSE is enough for
  most server-push cases and that WebSockets cost you statefulness at the edge.

### Task 4: Stage 3 — Designing APIs (running total: 21)

**Files:** Modify: `backend/index.html`

**Interfaces:** Produces stage id `apis`.

| id | name | depth | core |
|---|---|---|---|
| `resource-modelling` | Resource modelling and URL design | build | yes |
| `validation` | Validation at the boundary | build | yes |
| `error-model` | Error model design | build | yes |
| `collections` | Collection endpoints: pagination, filtering and sorting | build | yes |
| `versioning` | Versioning and backward compatibility | build | no |
| `idempotency` | Idempotency and safe retries | build | yes |
| `rate-limiting` | Rate limiting and quotas | build | no |
| `api-style` | Choosing an API style: REST, gRPC and GraphQL | understand | no |
| `openapi` | OpenAPI that stays true | build | no |

Stage-specific requirements:

- `error-model`: a concrete `json` error envelope with a stable machine-readable
  code, and `application/problem+json` named. Must state that error codes are API
  surface and cannot change once clients branch on them.
- `collections`: offset vs cursor with the actual failure of offset (rows shift
  under a paginating client, so items are skipped or repeated). Include the real
  `sql` for a keyset query (`WHERE (created_at, id) < ($1, $2) ORDER BY created_at DESC, id DESC LIMIT $3`)
  and explain why the tiebreaker column is mandatory.
- `idempotency`: an idempotency key names the attempt, not the request. Include the
  `sql` for a unique-constraint-backed key table and a `js` request handler that
  returns the stored response on a repeat. Must cover what to do when the first
  attempt is still in flight, which is the part everyone gets wrong.
- `rate-limiting`: token bucket and sliding window, where the counter lives, 429
  with `Retry-After`, and per-key versus per-tenant limits. Include the Redis
  commands for an atomic token bucket and point to the Redis topic by name.
- `api-style`: one substantial comparison. GraphQL must include the N+1 resolver
  problem and dataloader batching; gRPC must include deadlines and streaming.
  `tradeoff` must give a default recommendation and the signals to deviate.

### Task 5: Stage 4 — Data modelling and the database (running total: 31)

**Files:** Modify: `backend/index.html`

**Interfaces:** Produces stage id `database`.

| id | name | depth | core |
|---|---|---|---|
| `sql-fluency` | SQL you need fluently | build | yes |
| `relational-modelling` | Relational modelling and deliberate denormalization | build | yes |
| `keys-constraints` | Keys, constraints and integrity in the database | build | yes |
| `indexes` | Indexes | build | yes |
| `query-plans` | Reading query plans | build | yes |
| `transactions` | Transactions and isolation anomalies | build | yes |
| `locking` | Locking, deadlocks and lost updates | build | no |
| `db-pooling` | Connection pooling and pool exhaustion | build | no |
| `migrations` | Zero-downtime schema migrations | build | no |
| `orm-n-plus-one` | ORMs and the N+1 problem | build | no |

Stage-specific requirements:

- `sql-fluency` comes first because indexes and plans depend on it. Joins,
  aggregation with `GROUP BY` and `HAVING`, and one window function worked through
  (`ROW_NUMBER() OVER (PARTITION BY ...)` for "latest row per group"). Real SQL.
- `relational-modelling`: normalization stated as its one practical rule — one
  fact in one place — then the cases for deliberate duplication and what you owe
  in return (a way to keep copies true). Do not drill 1NF/2NF/3NF.
- `keys-constraints`: natural vs surrogate, UUID v4 versus v7 versus bigint and
  the index-locality consequence, foreign keys, `UNIQUE`, `CHECK`, and the
  argument that a constraint in the database is the only one that cannot be
  bypassed. Real SQL.
- `indexes`: B-tree shape, composite column order and the leftmost-prefix rule,
  covering indexes, selectivity, and the write cost of every extra index. Include
  a `sql` example where column order decides whether the index is used.
- `query-plans`: `EXPLAIN ANALYZE` output read line by line, with a seq scan
  versus index scan, estimated versus actual rows, and what a bad estimate means.
  Include real plan output; mark it illustrative if no engine is available.
- `transactions`: ACID stated usefully, then the four anomalies, then a table of
  isolation levels against which anomalies each prevents. Must state Postgres'
  default is read committed and that it is not repeatable read — verify and cite.
- `locking`: row locks, `SELECT ... FOR UPDATE`, deadlock cause and the ordering
  rule that avoids them, lost updates, and optimistic (version column) versus
  pessimistic locking with the choice rule. Real SQL for both.
- `db-pooling`: pool size arithmetic, the pool as the real concurrency limit,
  exhaustion presenting as latency rather than errors, `max_connections`, and
  transaction-mode poolers such as pgbouncer with what they break (session state,
  prepared statements).
- `migrations`: expand/contract in its five ordered steps, with the rule that
  schema and code deploy separately, plus the specific traps: adding a NOT NULL
  column, renaming, and an index build that locks writes (`CREATE INDEX CONCURRENTLY`).

### Task 6: Stage 5 — The data layer in production (running total: 35)

**Files:** Modify: `backend/index.html`

**Interfaces:** Produces stage id `dataops`.

| id | name | depth | core |
|---|---|---|---|
| `store-choice` | Choosing a data store | understand | yes |
| `bulk-writes` | Batching, bulk writes and server-side cursors | build | no |
| `soft-deletes-audit` | Soft deletes, audit trails and time-based data | build | no |
| `replicas-lag` | Read replicas and replication lag | build | no |

Stage-specific requirements:

- `store-choice`: relational as the default and the burden of proof on anything
  else. Document, key-value, search and time-series each get the signal that
  justifies them. Graph and wide-column get one honest line each. No feature tour.
- `bulk-writes`: multi-row insert versus `COPY`, chunking a large update so no
  transaction runs for minutes, and server-side cursors for reading more rows than
  fit in memory. Distinguish from HTTP streaming, which is in the storage stage.
- `soft-deletes-audit`: `deleted_at` and the three things it breaks (unique
  constraints, every query that forgets the filter, and foreign keys), partial
  unique indexes as the fix, append-only audit tables, and storing time ranges.
- `replicas-lag`: read-your-writes violations as the concrete bug, measuring lag,
  routing rules, and why "read from a replica" is not a free performance win.

### Task 7: Stage 6 — Testing and quality (running total: 41)

**Files:** Modify: `backend/index.html`

**Interfaces:** Produces stage id `testing`.

| id | name | depth | core |
|---|---|---|---|
| `test-levels` | What to test at which level | build | yes |
| `integration-tests` | Integration tests against real dependencies | build | yes |
| `test-doubles` | Test doubles, and what not to mock | build | yes |
| `flaky-tests` | Flaky tests and determinism | build | no |
| `contract-testing` | Contract testing between services | understand | no |
| `load-testing` | Load, stress and soak testing | understand | no |

Stage-specific requirements:

- `test-levels`: a decision rule, not the pyramid as a slogan. Must cover the
  error and authorization paths people skip, and testing the HTTP boundary.
- `integration-tests`: a real database rather than a mock, Testcontainers or an
  ephemeral schema, and a transaction rolled back per test for isolation. Include
  a `js` snippet of per-test rollback.
- `test-doubles`: fake, stub, mock and spy distinguished usefully, then the rule
  that matters — do not mock what you own, do not assert on calls when you can
  assert on state.
- `flaky-tests`: the four causes (wall clock, randomness, shared state, test
  ordering) with the fix for each: inject the clock, seed the generator, isolate
  the fixture, randomise the order to expose the dependency.
- `contract-testing`: must open by saying this pays off with several teams and is
  overhead for one, then explain consumer-driven contracts and the broker.
- `load-testing`: load, stress and soak distinguished by what each answers,
  open versus closed models, and reading the result as percentiles against a
  throughput target. Point to the performance stage by name for percentiles.

### Task 8: Stage 7 — Auth, authorization and security (running total: 50)

**Files:** Modify: `backend/index.html`

**Interfaces:** Produces stage id `security`.

| id | name | depth | core |
|---|---|---|---|
| `sessions-vs-tokens` | Sessions vs tokens, and cookie flags | build | yes |
| `passwords` | Passwords and credential handling | build | yes |
| `jwt` | Tokens in practice: JWT validation, expiry, refresh and revocation | build | yes |
| `oauth-oidc` | OAuth 2.1 and OpenID Connect | understand | no |
| `authz-models` | Authorization models: RBAC, ABAC, ownership and tenant isolation | build | yes |
| `service-auth` | Service-to-service auth | understand | no |
| `untrusted-input` | Untrusted input: injection, SSRF, traversal and deserialization | build | yes |
| `cors-headers` | CORS, security headers and browser-facing concerns | build | no |
| `data-protection` | Data protection: encryption, key rotation, PII and logs | build | no |

Stage-specific requirements:

- `sessions-vs-tokens`: a real recommendation (server sessions for first-party web
  apps), `HttpOnly`, `Secure`, `SameSite` and the domain/path scope, and CSRF with
  why `SameSite=Lax` is most of the defence and the token is the rest.
- `passwords`: Argon2id first, bcrypt as acceptable, parameters that matter, per-user
  salts handled by the library, never a reversible scheme, and a reset flow with
  single-use expiring tokens and no account enumeration.
- `jwt`: verify signature *and* algorithm *and* issuer *and* audience *and*
  expiry; the `alg: none` and algorithm-confusion attacks; why a stateless token
  cannot be revoked, and the three real answers (short expiry, a revocation list,
  reference tokens). Refresh rotation and reuse detection. Include a `js` snippet
  of correct verification showing the options that are easy to omit.
- `oauth-oidc`: who the four parties are, the authorization code flow with PKCE
  walked through, OIDC adding identity on top, and that OAuth 2.1 requires PKCE
  and drops implicit. **Verify the PKCE requirement against the specification and
  cite it.** Must state plainly that this is for delegated access, not a way to
  log users into your own first-party app by default.
- `authz-models`: authentication is who, authorization is what, and the bug that
  actually ships — an object-level check missing, so `/orders/{id}` returns another
  tenant's order. RBAC, ABAC, ownership checks, and tenant isolation enforced at
  the query layer rather than per-handler. Include the pseudocode of a
  tenant-scoped repository.
- `untrusted-input`: parameterized queries with the real `sql` and the string
  concatenation that is wrong beside it; SSRF with the allowlist-plus-metadata-IP
  defence; path traversal; and deserialization of untrusted input. Reference the
  OWASP Top 10 as reading, not as a topic.
- `cors-headers`: what CORS actually protects, preflight, why `Access-Control-Allow-Origin: *`
  plus credentials is refused by browsers, and that CORS is not authorization.
  Then CSP, HSTS and `X-Content-Type-Options` in one paragraph each.
- `data-protection`: TLS in transit, encryption at rest and what it does not
  protect against, envelope encryption and key rotation, PII minimisation, and the
  concrete list of things that must never reach a log: tokens, passwords, card
  numbers, full request bodies on auth endpoints.

### Task 9: Stage 8 — Caching and performance (running total: 59)

**Files:** Modify: `backend/index.html`

**Interfaces:** Produces stage id `caching`.

| id | name | depth | core |
|---|---|---|---|
| `cache-layers` | Cache layers and the CDN | understand | yes |
| `cache-strategies` | Strategies: cache-aside, write-through, write-behind | build | yes |
| `cache-keys` | Keys, TTLs and invalidation | build | no |
| `stampedes` | Stampedes and hot keys | build | no |
| `http-caching` | HTTP caching and ETags | build | yes |
| `redis` | Redis beyond caching | build | no |
| `bottleneck` | Finding the bottleneck: profiling and percentiles | build | no |
| `memory-gc` | Memory, GC and leaks | understand | no |
| `tail-latency` | Why p99 goes bad before CPU does | understand | no |

Stage-specific requirements:

- `cache-strategies`: cache-aside as the default with its real code path, and
  write-behind stated as rare and risky rather than presented as an equal.
- `cache-keys`: key naming with a version prefix so a format change is a
  deploy-time invalidation, TTL as the primary correctness tool, and the fact that
  precise invalidation is the hard part. Include the key-versioning pattern.
- `stampedes`: the concrete failure (a hot key expires, a thousand requests hit
  the database at once), then the three defences: a lock or single-flight, TTL
  jitter, and early refresh. Negative caching for misses. Include a `js`
  single-flight snippet, run under Node.
- `http-caching`: `Cache-Control` directives that matter, `ETag` and
  `If-None-Match` returning 304, `Last-Modified`, and why `private` versus
  `public` decides whether a CDN may store an authenticated response. Include a
  real request/response pair as an unhighlighted snippet.
- `redis`: strings, hashes, lists, sets, sorted sets and streams with the job each
  is for, `SETNX`/`SET NX` and `INCR` as atomic primitives, TTL semantics, Lua for
  multi-step atomicity, and the single-threaded model meaning one slow command
  blocks everyone (`KEYS` in production).
- `bottleneck`: measure before changing, percentiles not averages, where to put
  timers, and a CPU profile versus a wait profile. Must name the tools generically.
- `memory-gc`: heap versus stack, what a pause costs, and the four leak shapes
  that actually happen: an unbounded cache, a retained closure, an unremoved
  listener, and an ever-growing module-level collection. How to read RSS versus
  heap used.
- `tail-latency`: queueing stated practically — as utilisation approaches one,
  wait time explodes, so a server at 90% CPU has a bad p99 while its average looks
  fine. Little's law as one usable sentence, no derivation.

### Task 10: Stage 9 — Concurrency and async work (running total: 65)

**Files:** Modify: `backend/index.html`

**Interfaces:** Produces stage id `concurrency`.

| id | name | depth | core |
|---|---|---|---|
| `async-io` | Async I/O in your code: what actually yields, and how pools starve | build | yes |
| `background-jobs` | Background jobs: moving work out of the request | build | yes |
| `scheduled-work` | Scheduled work, overlapping runs and singletons | build | no |
| `shared-state` | Shared state: races, atomic updates and compare-and-swap | build | yes |
| `distributed-locks` | Distributed locks and leases | understand | no |
| `backpressure` | Concurrency limits and backpressure | build | no |

Stage-specific requirements:

- `async-io`: async is about waiting, not speed; only I/O yields, so a CPU-bound
  loop in an async function blocks everything; `await` in a loop serialises work
  that could be concurrent. Include a `js` snippet contrasting sequential `await`
  in a loop with `Promise.all`, run under Node with its real timing in the prose.
  Pool starvation and the sync-over-async deadlock, with the C# form named.
- `background-jobs`: the rule for what leaves the request (anything the caller
  does not need the result of, anything slow, anything that may need retrying),
  the job record as the source of truth, and status reporting back to the caller.
- `scheduled-work`: cron drift, a run that overlaps its predecessor, missed runs
  after downtime, and the lease or leader election that keeps a singleton single
  across instances. Point to distributed locks by name.
- `shared-state`: the read-modify-write race shown concretely, then the fix order
  that matters — prefer an atomic database update (`UPDATE ... SET n = n + 1`) or a
  unique constraint over any lock; a version column for compare-and-swap; in-process
  locks only for in-process state. Real SQL.
- `distributed-locks`: must open by saying you usually do not need one, and that a
  constraint or an atomic update is correct where a lock is merely probable. Then
  leases with expiry, fencing tokens, clock dependence, and the honest note that
  Redlock's safety is contested. Point to clocks and ordering by name.
- `backpressure`: an unbounded queue is a latency bomb; bound the queue, bound
  concurrency per dependency, shed load with 429 or 503 rather than collapse.
  Include a `js` bounded-concurrency snippet, run under Node.

### Task 11: Stage 10 — Messaging and event-driven architecture (running total: 73)

**Files:** Modify: `backend/index.html`

**Interfaces:** Produces stage id `messaging`.

| id | name | depth | core |
|---|---|---|---|
| `broker-choice` | Queue vs pub/sub vs log, and delivery semantics | understand | yes |
| `consumer-idempotency` | At-least-once and consumer idempotency | build | yes |
| `ordering-partitions` | Ordering, partitioning and consumer groups | build | no |
| `dlq-retries` | Retries, dead-letter queues and poison messages | build | yes |
| `outbox` | The outbox pattern and dual writes | build | no |
| `event-schemas` | Event schemas and versioning | build | no |
| `sagas` | Sagas and compensation | understand | no |
| `read-models` | Read models and projections | understand | no |

Stage-specific requirements:

- `broker-choice`: work queue, fan-out pub/sub and a replayable log as three
  different shapes, with a named example of each. Delivery semantics: at-most-once,
  at-least-once, and the statement that exactly-once delivery is not available —
  what you get is at-least-once delivery with idempotent processing.
- `consumer-idempotency`: the concrete duplicate (a payment charged twice), a
  processed-message table with a unique constraint, natural idempotency through
  upserts, and the ordering rule — commit the side effect and the dedupe record
  together or make the side effect safe to repeat. Real SQL.
- `ordering-partitions`: ordering only within a partition, choosing the partition
  key so related events stay ordered, consumer groups and rebalancing, and the
  trade-off between parallelism and ordering.
- `dlq-retries`: immediate retry versus delayed retry, a bounded retry count, the
  DLQ as a queue someone must actually watch, and the poison message that blocks a
  partition forever. Must state that a DLQ with no alert is a silent data-loss hole.
- `outbox`: the dual-write problem stated precisely — a database commit and a
  publish cannot be atomic — then the outbox table written in the same
  transaction and relayed after, with CDC named as the alternative mechanism and
  when it is preferable. Real SQL plus the relay loop as pseudocode.
- `event-schemas`: events are a published contract; additive changes only; carry
  enough state that a consumer need not call back, and the trade-off against
  staleness; an event id and a timestamp on every event.
- `sagas`: must open by asking whether these steps should be one service and one
  transaction, since that is usually the right answer. Then a saga as a sequence
  with a compensating action per step, orchestration versus choreography, and the
  fact that compensation is not rollback.
- `read-models`: a projection built from events for a query shape the write model
  serves badly, with staleness as the cost. CQRS scoped to that. Event sourcing
  gets an explicit warning paragraph: it makes schema change and debugging harder,
  and is right mainly where the audit log *is* the product.

### Task 12: Stage 11 — Storage and external integrations (running total: 79)

**Files:** Modify: `backend/index.html`

**Interfaces:** Produces stage id `storage`.

| id | name | depth | core |
|---|---|---|---|
| `file-uploads` | File uploads | build | yes |
| `object-storage` | Object storage and pre-signed URLs | build | no |
| `streaming-payloads` | Streaming large payloads, imports and exports | build | no |
| `third-party-apis` | Consuming third-party APIs | build | no |
| `receiving-webhooks` | Receiving webhooks | build | yes |
| `sending-webhooks` | Sending webhooks | build | no |

Stage-specific requirements:

- `file-uploads`: multipart versus direct binary, a size limit enforced at the
  proxy *and* the app, content type determined by sniffing the bytes rather than
  trusting the extension or the declared type, filename sanitisation, and never
  serving user uploads from your own origin.
- `object-storage`: store the bytes in object storage and the metadata in your
  database; pre-signed URLs so bytes never cross your service; the upload flow in
  ordered steps (request a URL, upload directly, confirm, mark the record ready);
  short expiry and a content-length condition.
- `streaming-payloads`: never load a whole file or result set into memory; streams
  and backpressure; an export streamed row by row with a server-side cursor; an
  import chunked with per-row error reporting and a resumable offset. Include a
  `js` streaming snippet, run under Node.
- `third-party-apis`: separate sandbox and production credentials, a deadline on
  every call, the provider's own rate limits and idempotency keys, webhooks versus
  polling, and recording real responses so tests do not hit the provider. Point to
  the reliability stage by name for retries rather than restating them.
- `receiving-webhooks`: verify the signature over the raw body before parsing it,
  constant-time comparison, a timestamp window and a replay cache, respond 2xx
  fast then process asynchronously, and treat delivery as at-least-once so the
  handler must be idempotent. Include a `js` HMAC verification snippet, run under
  Node, and note the common bug of verifying against a re-serialised body.
- `sending-webhooks`: subscriptions, signing what you send, retry with backoff and
  a visible delivery log, disabling a persistently failing endpoint, and never
  trusting a subscriber URL without SSRF checks. Point to SSRF by name.

### Task 13: Stage 12 — Reliability and fault tolerance (running total: 84)

**Files:** Modify: `backend/index.html`

**Interfaces:** Produces stage id `reliability`.

| id | name | depth | core |
|---|---|---|---|
| `timeouts-retries` | When a dependency fails: timeouts, retries, jitter and budgets | build | yes |
| `circuit-breakers` | Circuit breakers, bulkheads and degraded modes | build | no |
| `lifecycle-probes` | Readiness, liveness and graceful shutdown | build | yes |
| `error-handling` | Error handling that aids diagnosis | build | yes |
| `backups-rollback` | Backups, restore drills and rollback | build | no |

Stage-specific requirements:

- `timeouts-retries`: opens with partial failure — a call that neither succeeds
  nor fails. Every outbound call has a timeout, and a timeout shorter than the
  caller's. Retry only idempotent operations. Exponential backoff with full jitter,
  with the formula. A retry budget, because retries multiply load exactly when the
  dependency is already failing. Include a `js` backoff-with-jitter snippet, run
  under Node.
- `circuit-breakers`: the three states and the threshold that moves between them,
  why a breaker protects the *dependency* as much as you, bulkheads as a separate
  pool per dependency, and a degraded mode that returns stale or partial data
  instead of failing.
- `lifecycle-probes`: liveness answers "restart me", readiness answers "send me
  traffic", and conflating them causes restart loops. Graceful shutdown in ordered
  steps: fail readiness, stop accepting, drain in-flight work with a deadline, close
  pools, exit. SIGTERM and the grace period. Point to the Kubernetes topic by name.
- `error-handling`: expected versus unexpected errors, catch at a boundary rather
  than everywhere, never swallow, attach context as the error travels, and return a
  stable code to the client while logging the detail. Must state that an empty
  `catch` is the single most expensive line in a codebase to debug.
- `backups-rollback`: a backup you have never restored is not a backup; a restore
  drill with a measured time; RPO and RTO as two different numbers; point-in-time
  recovery; and rolling back code when the migration cannot be rolled back, which
  is why expand/contract exists. Point to migrations by name.

### Task 14: Stage 13 — Observability and debugging (running total: 89)

**Files:** Modify: `backend/index.html`

**Interfaces:** Produces stage id `observability`.

| id | name | depth | core |
|---|---|---|---|
| `structured-logs` | Structured logs and correlation IDs | build | yes |
| `metrics` | Metrics: golden signals, RED/USE and histograms | build | yes |
| `tracing` | Distributed tracing and context propagation | understand | no |
| `alerting-slos` | Alerting, SLOs, runbooks and postmortems | understand | no |
| `debugging-prod` | Debugging production systematically | build | yes |

Stage-specific requirements:

- `structured-logs`: one JSON object per event, a correlation id generated at the
  edge and propagated through every call and job, the levels and what each is for,
  sampling high-volume lines, and the list of fields that must never be logged.
  Include a `js` snippet of a logger carrying request context.
- `metrics`: latency, traffic, errors and saturation; RED for services and USE for
  resources; counters, gauges and histograms, with the point that an average
  latency hides everything and a histogram gives you percentiles. Prometheus and
  Grafana named as the usual toolchain. Must warn about label cardinality.
- `tracing`: a trace as a tree of spans across services, context propagation via
  the `traceparent` header, and OpenTelemetry as the vendor-neutral API.
  **Verify the header name against the W3C Trace Context specification and cite
  it.** Must say honestly that for a single service, structured logs with a
  correlation id get you most of the way, and tracing earns its cost at the second
  or third service.
- `alerting-slos`: alert on symptoms users feel, not causes; every alert needs an
  action and a runbook or it should be a dashboard; pick two or three SLIs and a
  target you would actually act on; error budgets as the honest version of
  "reliable enough"; blameless postmortems with actions that have owners.
- `debugging-prod`: a method, in order — establish what changed, confirm the
  symptom with a measurement, bisect the request path hop by hop, form one
  hypothesis and test it, and only then change something. Must state that the
  first instinct, restarting, destroys the evidence.

### Task 15: Stage 14 — Deployment and production engineering (running total: 96)

**Files:** Modify: `backend/index.html`

**Interfaces:** Produces stage id `deploy`.

| id | name | depth | core |
|---|---|---|---|
| `containerising` | Containerising a service properly | build | yes |
| `pipeline` | Pipeline: build once, promote the artifact | build | yes |
| `release-strategies` | Release strategies and feature flags | build | no |
| `kubernetes-objects` | The Kubernetes objects you actually touch | understand | no |
| `limits-autoscaling` | Resource limits, autoscaling and OOMKills | build | no |
| `iac` | Infrastructure as code | understand | no |
| `cloud-iam` | Cloud primitives and least-privilege IAM | understand | no |

Stage-specific requirements:

- `containerising`: a real multi-stage `dockerfile` (unhighlighted by design) with
  a pinned base, dependency layer caching, a non-root user, and an entrypoint that
  receives signals as PID 1 so SIGTERM reaches the process. Must point to graceful
  shutdown by name.
- `pipeline`: build the artifact once and promote the same digest through
  environments; the ordered stages; environment parity and config injected at
  deploy rather than baked in; secrets never in build logs.
- `release-strategies`: rolling, blue-green and canary compared on rollback speed
  and cost, feature flags decoupling deploy from release, and the rule that a flag
  without a removal date becomes permanent branching.
- `kubernetes-objects`: Deployment, Service, Ingress, ConfigMap and Secret, probes
  and `terminationGracePeriodSeconds`, and Service as the discovery mechanism.
  Include a real `yaml` Deployment fragment with probes and resources. Must stay at
  "operate your own service" scope.
- `limits-autoscaling`: requests versus limits, a CPU limit causing throttling
  rather than failure, a memory limit causing an OOMKill with no stack trace, HPA
  scaling on a signal that actually correlates with load, and the fact that a
  service with a connection-pool bottleneck does not scale by adding replicas.
  Point to pooling by name.
- `iac`: declarative desired state, a plan before an apply, state as the thing
  that must not be lost or shared carelessly, and drift. Concepts only; name
  Terraform and the alternatives. Not a tutorial.
- `cloud-iam`: roles attached to workloads rather than long-lived keys,
  least-privilege as a default-deny posture, network boundaries, and managed
  versus self-hosted databases as a real trade-off. No service tour.

### Task 16: Stage 15 — Distributed systems you'll actually meet (running total: 101)

**Files:** Modify: `backend/index.html`

**Interfaces:** Produces stage id `distributed`.

| id | name | depth | core |
|---|---|---|---|
| `service-boundaries` | Monolith, services and the distributed-monolith trap | understand | yes |
| `sharding` | Horizontal scaling, sharding and partition keys | understand | no |
| `consistency-models` | Consistency models and what eventual consistency costs you | understand | no |
| `replication-quorum` | Replication, leader election and quorum as things you operate | understand | no |
| `clocks-ordering` | Clocks, ordering and why timestamps lie | understand | no |

Stage-specific requirements:

- `service-boundaries`: a modular monolith as the correct default; vertical versus
  horizontal scaling; the distributed monolith — services that must deploy
  together, share a database, or make a synchronous chain per request — with the
  boundary test being data ownership. Must reference the planned system design
  track by name as the place that goes further.
- `sharding`: scale reads with replicas before sharding anything; choosing a
  partition key so related data lands together and no key is hot; resharding as
  the hard part; consistent hashing explained in one worked paragraph.
- `consistency-models`: strong versus eventual stated as what a reader can
  observe, the concrete costs (read-your-writes, monotonic reads), CAP folded in
  as the honest one-paragraph version without theorem worship, and multi-region as
  a trade-off paragraph rather than a how-to.
- `replication-quorum`: synchronous versus asynchronous replication and the
  durability-versus-latency trade, failover and the split-brain risk, quorum as
  a majority with the point that a two-node cluster is worse than one, and
  consensus named as something you configure and watch, not implement.
- `clocks-ordering`: wall clock versus monotonic clock and which to use for a
  duration, NTP skew, `updated_at` as an unreliable ordering key across machines,
  token expiry failing under skew, and sequence numbers or a logical clock as the
  fix. Point to distributed locks by name.

---

### Task 17: The capstone

**Files:** Modify: `backend/index.html`

**Interfaces:** Consumes every stage id above, by name, in the milestone table.
Produces the stage object with id `capstone` and a `custom` HTML block.

The capstone is a stage with `custom` HTML instead of `topics`, so it renders as
prose. It has no progress ticks and no `after` block.

- [x] **Step 1: Add the capstone CSS**

```css
.cap{margin:14px 0 0}
.cap h3{font-family:var(--display);font-weight:700;font-size:1.25rem;margin:28px 0 6px}
.cap h4{font-family:var(--display);font-weight:700;font-size:1.02rem;margin:18px 0 4px}
.cap p{margin:8px 0 0}
.cap ul,.cap ol{margin:8px 0 0;padding-left:1.3em}
.cap li{margin:5px 0}
.cap .ms{background:var(--surface);border:1px solid var(--line);border-radius:14px;padding:16px 18px;margin:14px 0 0}
.cap .ms h4{margin-top:0}
.cap .ms .uses{font-size:.9rem;color:var(--muted);margin-top:8px}
.cap .ms .check{background:var(--accent-soft);border-radius:10px;padding:10px 14px;margin-top:10px}
.cap .ms .check p{margin:0}
```

- [x] **Step 2: Write the capstone stage object**

```js
{ num:16, id:"capstone", phase:6, name:"Capstone: build the order service",
  intro:"One service that exercises the whole route, in eight milestones.",
  custom:`<div class="cap"> ... </div>`,
  sources:`<p>...</p>` }
```

The `custom` HTML must contain, in this order:

1. **The brief.** An order service for a small shop: create an order, pay for it,
   fulfil it, and let a customer read their orders. One relational database, one
   cache, one queue, one object store. Stated constraints: it must survive a
   duplicate client retry, a restart mid-request, and a dependency being down.
2. **What to leave out**, explicitly: no microservices, no Kubernetes until
   milestone 8, no event sourcing, no multi-region. Each with one line on why.
3. **Eight milestone cards** (`div.ms`), each with: an `h4` title, two or three
   sentences of what to build, a `p.uses` line naming the topics it exercises by
   name, and a `div.check` with one acceptance check the learner can actually run.
   The milestones:

   | # | Milestone | Acceptance check |
   |---|---|---|
   | 1 | Schema, constraints and the first endpoints | A test proves the database rejects an invalid order without the application checking |
   | 2 | Validation, the error model and collection endpoints | A cursor-paginated list returns no duplicate or skipped row while rows are inserted during paging |
   | 3 | Idempotent order creation | The same idempotency key sent twice, concurrently, creates one order and returns the same response body twice |
   | 4 | Authentication and tenant-scoped authorization | A test proves tenant A cannot read tenant B's order by id |
   | 5 | Caching the read path, with correct invalidation | A load test shows the cache hit, and an update is visible on the next read |
   | 6 | Payment as a background job, with an outbox | Killing the worker mid-job loses no job and charges nothing twice |
   | 7 | Timeouts, retries, breaker, logs, metrics and probes | With the payment provider stubbed to hang, the endpoint returns a degraded response within its timeout, and the metric and log show why |
   | 8 | Container, pipeline and a zero-downtime deploy | A rolling deploy under load drops no request, and a migration runs with the old code still live |

4. **A closing paragraph** in a `.closing` block: what to do after — rerun the
   route's filters and tick what you can now defend, and reread the stages whose
   exercises you had to look things up for.

- [x] **Step 3: Update the hero and the capstone's phase**

The capstone is phase 6 alongside `distributed`. No hero change is needed; confirm
the sixth dot still links to `#distributed`.

- [x] **Step 4: Run the checks**

All four verification commands. The field check skips the capstone because it has
no `topics`; expected total is still `topics: 101`.

- [x] **Step 5: Browser check**

Confirm the capstone renders as prose with eight milestone cards, no tick boxes
appear in it, and the overall progress still reads `n of 101 done`.

- [x] **Step 6: Commit and push**

Commit message subject: `Add the backend route capstone: the order service`.

---

### Task 18: Landing page integration

**Files:**
- Modify: `index.html` (colour blocks, the tile grid, the count script, head, footer)

**Interfaces:** Consumes `backend/index.html`'s `depth:"build"` / `depth:"understand"`
literals, which the count script matches as text.

- [x] **Step 1: Add the colour tokens to all three colour blocks**

Light `:root`:

```css
--t-backend:#0F766E; --t-backend-soft:#D7F0ED;
```

Both dark blocks:

```css
--t-backend:#5ED3C4; --t-backend-soft:#10332F;
```

- [x] **Step 2: Add the tile class and rebalance the spans**

```css
.track.backend{--acc:var(--t-backend);--soft:var(--t-backend-soft)}
```

Change `.note{grid-column:span 2}` to `.note{grid-column:span 3}` so that four
track tiles, the planned tile and three notes all tile cleanly two-per-row on a
six-column grid.

- [x] **Step 3: Generalise the count script**

Replace the keyset branch in the script at the bottom of `index.html` with a
table, so a third track kind does not need another conditional. Existing tiles
keep their exact behaviour.

```js
const TRACK_KINDS = {
  level: { keys:["fresher","mid","senior"],
    pat: k => 'level:"' + k + '"',
    label: n => `Questions by level: ${n.fresher} fresher, ${n.mid} mid, ${n.senior} senior` },
  difficulty: { keys:["easy","med","hard"],
    pat: k => 'class="diff ' + k + '"',
    label: n => `Problems by difficulty: ${n.easy} easy, ${n.med} medium, ${n.hard} hard` },
  depth: { keys:["build","understand"],
    pat: k => 'depth:"' + k + '"',
    label: n => `Topics by depth: ${n.build} to build, ${n.understand} to understand` }
};

document.querySelectorAll(".track[data-src]").forEach(async (tile) => {
  try {
    const res = await fetch(tile.dataset.src);
    if (!res.ok) return;
    const src = await res.text();
    const kind = TRACK_KINDS[tile.dataset.kind || "level"];
    const n = {};
    kind.keys.forEach(k => {
      n[k] = (src.match(new RegExp(kind.pat(k), "g")) || []).length;
    });
    n.total = kind.keys.reduce((sum, k) => sum + n[k], 0);
    if (!n.total) return;
    tile.querySelectorAll("[data-count]").forEach(el => { el.textContent = n[el.dataset.count]; });
    tile.querySelectorAll(".mix-bar [data-level]").forEach(el => {
      el.style.flex = String(n[el.dataset.level] || 0);
    });
    tile.querySelector(".mix-bar").setAttribute("aria-label", kind.label(n));
  } catch (e) { /* keep static values */ }
});
```

- [x] **Step 4: Add the tile**

Insert after the React tile and before the planned `sys` tile. The static counts
are the fallback when the page is opened from disk; replace `NN`/`MM` with the
real counts from the field check in Task 16.

```html
<section class="tile track backend" data-src="backend/" data-kind="depth" aria-labelledby="t-backend">
  <p class="kicker">Server-side craft</p>
  <h2 id="t-backend">Backend engineering</h2>
  <p class="blurb">A learning route, not a question bank: the request path, API design, databases, testing, security, caching, concurrency, messaging, reliability, observability and deployment, ending in one service you build.</p>
  <ul class="topics">
    <li><a href="backend/#foundations">Foundations</a></li>
    <li><a href="backend/#apis">APIs</a></li>
    <li><a href="backend/#database">Databases</a></li>
    <li><a href="backend/#testing">Testing</a></li>
    <li><a href="backend/#security">Security</a></li>
    <li><a href="backend/#caching">Caching and performance</a></li>
    <li><a href="backend/#concurrency">Concurrency</a></li>
    <li><a href="backend/#messaging">Messaging</a></li>
    <li><a href="backend/#reliability">Reliability</a></li>
    <li><a href="backend/#observability">Observability</a></li>
    <li><a href="backend/#deploy">Deployment</a></li>
    <li><a href="backend/#capstone">Capstone</a></li>
  </ul>
  <div class="mix">
    <div class="mix-bar" role="img" aria-label="Topics by depth: NN to build, MM to understand">
      <span data-level="build" style="background:var(--senior);flex:NN"></span>
      <span data-level="understand" style="background:var(--fresher);flex:MM"></span>
    </div>
    <ul class="mix-legend">
      <li><i style="background:var(--senior)"></i><b data-count="build">NN</b> to build</li>
      <li><i style="background:var(--fresher)"></i><b data-count="understand">MM</b> to understand</li>
    </ul>
    <div class="cta-row">
      <a class="cta" href="backend/">Start the route</a>
      <p class="total"><span data-count="total">101</span> topics across 15 stages</p>
    </div>
  </div>
</section>
```

- [x] **Step 5: Update the head and footer**

```html
<title>PrepOps: ASP.NET Core, DSA, React and backend engineering</title>
```

Update both meta descriptions to mention the backend route, and change the footer
version line to: `Framework details target .NET 8 and .NET 10. Python examples were tested on Python 3.12, Node examples on Node 22. Checked in November 2026.`

- [x] **Step 6: Verify the counts are live**

```bash
cd /home/user/interview-prep && python3 -m http.server 8000
```

Open `http://localhost:8000/`. Confirm the backend tile's numbers match the field
check's total, the mix bar has two segments, the other three tiles' counts are
unchanged from before this task, and the grid has no orphaned tile at desktop,
tablet and phone widths.

- [x] **Step 7: Commit and push**

Commit message subject: `Add the backend engineering tile to the landing page`.

---

### Task 19: README and CLAUDE.md

**Files:**
- Modify: `README.md`
- Modify: `CLAUDE.md`

- [x] **Step 1: README table row**

Insert after the React row:

```markdown
| [Backend engineering](backend/) | A practical route rather than a question bank: runtime and networking foundations, API design, databases, testing, security, caching and performance, concurrency, messaging, storage and integrations, reliability, observability, deployment, distributed systems, and a capstone service | Live, 101 topics across 15 stages |
```

- [x] **Step 2: README usage section**

Add after the DSA section:

```markdown
**Backend engineering**
- A learning route, not interview questions. Fifteen stages in order; each stage assumes the ones before it.
- Every topic has the same parts: what it is and why, a concrete case, core concepts, how to build it, the trade-offs, the pitfall, and an exercise.
- Filter to **Core only** for the spine of the route, or to **Build** and **Understand** to separate what you should implement yourself from what you should know the trade-off for.
- Tick topics off as you build or can explain them. The capstone composes the exercises into one order service.
- Code is pseudocode unless labelled; real snippets are Node, with C# where a second language helps, plus real SQL and config.
```

- [x] **Step 3: README structure block and accuracy note**

Add `backend/index.html          backend engineering route` to the structure
block, and a sentence to the accuracy section: the backend route's Node snippets
were run on Node 22, its SQL is Postgres dialect, and its source labels say which
pages were opened.

- [x] **Step 4: CLAUDE.md table row**

```markdown
| `backend/index.html` | Backend engineering route: 15 stages, 101 topics, plus a capstone spec | `backend:done:v1` (ticked topic ids), `backend:filter:v1` (toolbar filter) |
```

- [x] **Step 5: CLAUDE.md "How the backend route works" section**

Add after the "React and TypeScript track differences" section. It must document:
the `stages`/topic data shape with every field and which are required; that topic
and stage ids are anchors and progress keys and must never be renamed; the four
filter values and that `core` and `depth` are independent; the `lang` values and
that `dockerfile`, `nginx` and unlabelled snippets are deliberately unhighlighted;
that `code` is inserted with `textContent` so snippets are raw while `what`/`how`
are HTML and must be escaped; the capstone being a `custom` stage with no ticks;
and how to add a stage in four ordered steps.

- [x] **Step 6: CLAUDE.md check commands and colours**

Add `backend/index.html` to the HTML-parse list, add the id-duplicate and field
checks from this plan's **Verification cycle** section, and add `--t-backend` to
the track colours sentence.

- [x] **Step 7: Verify**

```bash
cd /home/user/interview-prep && python3 -c "import html.parser; [html.parser.HTMLParser().feed(open(f).read()) for f in ['index.html','aspnet-core/index.html','react-typescript/index.html','dsa/index.html','backend/index.html']]; print('html ok')" && grep -rn "claude.ai/artifact" --include=*.html --include=*.md --exclude=CLAUDE.md . || echo "clean"
```

Then run every command block in CLAUDE.md's "Checks before committing" section
verbatim and confirm each one passes as written.

- [x] **Step 8: Commit and push**

Commit message subject: `Document the backend engineering route`.

---

### Task 20: Final review pass

**Files:** Modify: whatever the review finds.

- [x] **Step 1: Duplication sweep**

```bash
cd /home/user/interview-prep && grep -o 'name:"[^"]*"' backend/index.html | sed 's/name:"//;s/"//' | sort
```

Read the 117 names together (101 topics plus 16 stages). For any two that could plausibly cover the same
ground, open both and confirm the later one defers to the earlier by name instead
of restating it. Fix any restatement.

- [x] **Step 2: Forward-reference check**

```bash
cd /home/user/interview-prep && grep -o 'covered in [^.<]*' backend/index.html | sort -u
```

Every cross-reference must name a stage or topic that exists, by name, and must
not point backwards to something the reader has not reached in a way that implies
a prerequisite. Fix any reference to a renamed or absent topic.

- [x] **Step 3: Link check**

```bash
cd /home/user/interview-prep && grep -o 'https\?://[^"]*' backend/index.html | sort -u | while read u; do code=$(curl -s -o /dev/null -w '%{http_code}' -L --max-time 15 "$u"); echo "$code $u"; done
```

Expected: `200` for every URL. Anything 4xx or 5xx is replaced or removed. A
timeout is retried once before being treated as a failure; if the host is
unreachable from this container, the link stays but the stage's `sources` must not
claim the page was opened.

- [x] **Step 4: Accuracy audit**

List every claim labelled as cited and confirm the page was actually retrieved in
this work. Any claim that was not gets relabelled as a reference not retrieved.
Specifically confirm: the OAuth 2.1 PKCE requirement, Postgres' default isolation
level, and the W3C `traceparent` header name.

- [x] **Step 5: Accessibility and responsive pass**

At 390px, 768px and 1280px, in light and dark: no horizontal page scroll, the
concepts list collapses to one column on phones, every interactive control is
reachable by keyboard with a visible focus ring, and the tick checkbox has its
accessible label.

- [x] **Step 6: Update the spec's status**

Tick every box in the spec's section 8 build order, and change the spec's status
line from `approved design, implementation pending` to `implemented`.

- [x] **Step 7: Commit and push**

Commit message subject: `Review pass over the backend engineering route`.

---

## Self-review

Run against the spec after the plan is written.

**Spec coverage.** Every spec section maps to a task: §2 decisions → Task 1
(colour, keys, shape) and Task 18 (landing colour); §3 page design → Task 1
steps 4-10; §4 inventory → Tasks 2-16, one stage each, ids matching the spec
exactly; capstone → Task 17; §5 exclusions → enforced by the content quality bar
item 7 and Task 20 step 1; §6 accuracy → Step B of every content task plus Task 20
step 4; §7 integration → Tasks 18 and 19; §8 build order → the task sequence,
updated in Task 20 step 6; §9 verification → the Verification cycle section.

**Placeholder scan.** `NN`/`MM` in Task 18 step 4 are deliberate: the real counts
are not known until Task 16 completes, and step 4 says where to get them. The
`...` in Task 17 step 2's stage object is the body the same step then specifies in
four numbered parts. No other placeholders.

**Type consistency.** Field names are defined once in Task 1's Interfaces block
and used unchanged throughout: `id`, `name`, `depth`, `core`, `line`, `what`,
`use`, `concepts`, `how`, `code`, `tradeoff`, `pitfall`, `exercise`, `refs`.
Stage fields: `num`, `id`, `phase`, `name`, `intro`, `topics`, `after`, `sources`,
`custom`. Functions: `renderTopic`, `applyFilter`, `refreshProgress`, `saveDone`,
`esc`. `localStorage` keys appear as `backend:done:v1` and `backend:filter:v1`
only. Stage ids used in the hero, the landing tile and the cross-references are
the same fifteen plus `capstone`: `foundations`, `network`, `apis`, `database`,
`dataops`, `testing`, `security`, `caching`, `concurrency`, `messaging`,
`storage`, `reliability`, `observability`, `deploy`, `distributed`, `capstone`.

**Counts.** 8+4+9+10+4+6+9+9+6+8+6+5+5+7+5 = 101 topics, which matches the spec
and every running total in the task headings.
