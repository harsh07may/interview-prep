# CLAUDE.md

Everything needed to understand, edit and publish this repository. Read it fully before changing anything.

## What this repo is

Interview preparation tracks plus a landing page, served as a static site on GitHub Pages. No build step, no framework, no package manager: every page is one self-contained HTML file. The site is called PrepOps; the repo and its URL keep the name `interview-prep`.

| Path | What it is | localStorage key |
|---|---|---|
| `index.html` | Landing page: bento grid linking to all tracks | none (reads each track's source to count questions) |
| `aspnet-core/index.html` | ASP.NET Core track, 8 topics, about 180 questions | `aspnet:level:v1` (selected level filter) |
| `react-typescript/index.html` | React and TypeScript track: 10 topics, 154 questions, same data-driven format as ASP.NET Core | `react:level:v1` (selected level filter) |
| `dsa/index.html` | DSA patterns track: 7 stages, 25 patterns, 285 problems, Python | `dsa:done:v1` (array of solved problem slugs) |
| `backend/index.html` | Backend engineering route: 15 stages, 101 topics, plus a capstone spec. A learning route, not a question bank | `backend:done:v1` (ticked topic ids), `backend:filter:v1` (toolbar filter) |
| `README.md` | Public description | |
| `.nojekyll` | Tells GitHub Pages to serve files as-is. Do not delete. | |

Planned tracks: `system-design/`. They appear on the landing page as dashed "Planned" tiles until they exist.

All sites under `harsh07may.github.io` share one browser origin, so every localStorage key must be namespaced by track (`aspnet:`, `dsa:`, ...).

## How a track page works

Content is data, not markup. The script at the bottom of each track declares one array per topic and a `topics` list; the page renders everything from it: topic headers, part sections, question cards, the "night before" list, and sources.

### Topic

```js
{ num:1, id:"fundamentals", name:"ASP.NET Core fundamentals", intro:"One line.",
  parts: fundamentals, revise: topic2Revise, sources: topic2Sources }
```

- `num` is the displayed topic number. `id` is the anchor used by the progress track and the landing page's topic chips. Never rename an `id` once published; links break.
- A topic can use `custom: htmlString` instead of `parts` and `revise` for free-form pages (the interview guide does this).

### Part and question

```js
{ title:"Part title", intro:"One line.", qs:[
  { level:"fresher" | "mid" | "senior",
    q:"The question as it's usually asked",
    a:`<p>HTML answer. Escape < > & inside it.</p>`,
    code:`raw code, not escaped, shown after the answer`,
    follow:"Likely follow-up, with a short answer.",
    trap:"The common wrong answer.",
    src:`Source basis, see below.`,
    predict:true   // optional: shows code first, answer behind "Show the answer"
  } ] }
```

- `code` is inserted with `textContent`, so write it raw. Code inside `a` is HTML, so escape it (`&lt;`, `&gt;`, `&amp;`).
- Every `pre code` block is highlighted by highlight.js 11.9.0 (cdnjs). `langFor()` in the last script picks the language: SQL if it starts with `--`, `SELECT` or `WITH`; plain text for table schemas like `Employees(Id, Name)`, numbered or bulleted lists and route lists; C# otherwise. If a new kind of snippet comes out wrongly coloured, extend `langFor()`.
- `predict:true` cards put the `code` (a snippet, schema, or prompt) up front and hide `a`. Use them for output prediction, query writing, code review, and scenarios.
- Source-link constants (like `CWM`, `SQLQ`) are defined once near the top of the script. Define a constant before any topic uses it; an undefined one stops the whole page rendering.

### Adding a topic

1. Add the parts array, a `topicNRevise` list and `topicNSources` HTML before `const topics = [`.
2. Add the topic entry to `topics`.
3. Add a link to the progress track `<nav class="track">` and update the track note.
4. Update the landing page chips and the "N topics" text.

### React and TypeScript track differences

`react-typescript/index.html` was copied from the ASP.NET Core page and renders the same way, with these changes:

- Accent is the React blue (`--accent` family plus `--on-accent`, used for text on accent backgrounds).
- A question can set `lang:"js" | "ts" | "tsx" | "css"`. The last script's `langFor()` reads it from `data-lang` and defaults to TypeScript, which also highlights plain JavaScript and JSX. Set `lang:"js"` on plain JavaScript so it is not presented as TS.
- The topic list and jump nav are generated from `topics`; the `<nav class="track">` links are still written by hand.
- Topic 10 (`guide`) is a `custom` topic with the priority table and the 20-item list. Its priorities are editorial, not measured; keep that wording.
- Source constants (`EFFECT`, `NOEFFECT`, `COMPILER`, `MEMO`, `USEMEMO`, `SYNC`) point at react.dev pages seen only as documentation excerpts. Use `REPORTED` or `REPORTED_REACT` for widely reported patterns, and do not upgrade a label to "Cited" unless the page was actually retrieved.
- Output-prediction snippets must be run in Node before publishing. TypeScript snippets in topic 6 were type-checked against TypeScript 5.x.

## How the backend route works

`backend/index.html` is data-driven like the ASP.NET Core page, but it renders a
different card and is a **learning route rather than a question bank**. It was
copied from `react-typescript/index.html`, recoloured teal, and had its renderer
replaced. Keep that distinction when editing: it has no levels, no follow-ups
and no traps, and adding them would make it a fourth question bank.

### Data shape

One `stages` array. Every stage has `num`, `id`, `phase`, `name`, `intro`, and
either `topics` or `custom`, plus optional `after` and `sources`.

```js
{ id:"idempotency",                    // anchor AND progress key: never rename
  name:"Idempotency and safe retries",
  depth:"build",                       // "understand" | "build"  (required)
  core:true,                           // part of the 46-topic spine
  line:"One line shown on the closed card.",
  what:`<p>HTML.</p>`,                     // required
  use:`<p>HTML, one concrete case.</p>`,   // required
  concepts:[ ["Term","One sentence."] ],   // required, 3 or more pairs
  how:`<p>HTML.</p>`,                      // required
  code:[ {label:"Node", lang:"js", code:`raw, unescaped`} ],   // optional array
  tradeoff:`<p>HTML.</p>`,             // optional
  pitfall:"Plain text.",               // optional
  exercise:`<p>HTML.</p>`,             // optional
  refs:[ ["Label","https://..."] ] }   // optional; an empty array renders nothing
```

- **Never rename a published `id`.** It is the anchor and the `backend:done:v1`
  progress key, so renaming silently discards a reader's ticks.
- `code` entries are inserted with `textContent`, so snippet source is raw.
  Everything in `what`, `use`, `how`, `tradeoff` and `exercise` is HTML and must
  escape `<`, `>` and `&`.
- `core` and `depth` are independent axes. The toolbar filters are All topics,
  Core only, Build and Understand.
- The capstone is a `custom` stage with no `topics`, so it renders as prose with
  no tick boxes and an empty stage-progress line.

### Writing snippets

Three escaping mistakes each cost a rebuild while this page was written, so
check a stage on its own before inserting it:

1. A shell line continuation (a lone backslash at end of line) inside a JS
   template literal is an escape and silently joins the two lines. Double it.
2. A double quote inside a double-quoted string ends it. Use `&quot;` in prose.
3. A backtick in pseudocode ends the template literal. Remove it.

`langFor()` maps `js`, `cs`, `sql`, `yaml`, `bash` and `json`. Anything else,
including `dockerfile`, `nginx`, `text` and unlabelled pseudocode, renders
unhighlighted on purpose: highlight.js 11.9.0's common bundle does not carry the
first two, and mis-highlighted pseudocode reads worse than none. Keep snippet
lines to 92 characters, or they scroll sideways on the page.

### Adding a stage

1. Write the stage object as a standalone `{ ... }` literal in its own file.
2. Syntax-check it alone (`node --check` on `const stage = <literal>;`) before
   inserting, so a broken stage never reaches the page.
3. Append it to `stages` before the closing `];`.
4. Run the checks below, then open the page.

### Content rules specific to this route

- Every claim that can be run should be run, and the snippet should carry its
  real output. Several topics here were rewritten because testing contradicted
  the conventional advice: a wrongly ordered index does not stop Postgres using
  the index, and a current JWT library rejects both `alg:none` and algorithm
  confusion while happily accepting a token minted for another audience.
- Where a snippet could not be verified in the authoring environment, the
  stage's `sources` says so explicitly.
- Topics that argue against their own subject are deliberate, not hedging:
  distributed locks, sagas, event sourcing, contract testing and sharding all
  open by saying when not to use them.

## How the DSA page works

Unlike the ASP.NET Core page, `dsa/index.html` is plain static markup, not rendered from a data array. Its structure:

- **Stages** are `<section class="block stage" id="stage-N" data-stage>`, with a `.stage-progress` line filled in by script. The stage track in the header links to `#stage-0` to `#stage-6`; the curriculum map cards use `#map-0` to `#map-6`.
- **Patterns** are `<section class="pattern" id="p-..." data-pattern>` containing `.pt-what`, two `.pt-box` panels (recognition cues, variations), `.pt-intuition`, a Python template in `<pre><code class="language-python">`, and problems grouped under `h4.grp` headings (Canonical, Variations, Combinations and stretch).
- **Problems** are `<li class="prob" data-id="leetcode-slug">`. The `data-id` is the LeetCode URL slug and is the progress key, so never change one after publishing: people's ticks would silently disappear. Every slug must be unique on the page.
- Each problem shows why it's included (`.p-why`), pattern tags, an optional "Solve first" line, and a collapsed `details.p-insight` key insight. Stage 6 problems (`li.prob.mixed`) have a Hint and a "Reveal the pattern" disclosure instead, and no tags.
- Background material (how the sheet was built, the resource comparison, sources) lives in the collapsed `#about` section at the end. Keep anything not used while practising in collapsible sections.
- The curriculum map (`#map`) and Prerequisites (`#stage-0`) are collapsed by default: each section's contents sit in a `details.fold` whose `summary` holds the heading and lead. A small script at the end opens a collapsed section when a link or the URL hash points into it (for example `#stage-0` or `#pre-heaps`), so keep that script if you move these sections.
- Syntax highlighting: highlight.js 11.9.0 from cdnjs, with token colours written inline in the page's CSS.

### DSA content rules

1. Free problems only. Replace LeetCode Premium problems with a free equivalent, and say so in the key insight if the free one differs (for example, closed versus half-open intervals).
2. Every problem must add something: the canonical form, a variation, an edge case, a combination, or a real step up in difficulty. Don't add near-duplicates.
3. "Why it's included" must not give away the solution; the approach belongs in the key insight.
4. Test before publishing. Run every template, and check every non-trivial key insight against a brute-force solution on random small inputs. When a check fails, find out whether the insight or the test is wrong before changing either.
5. Keep Python idiomatic and version-aware: note when something needs Python 3.10+ (`pairwise`, `bit_count`) or 3.14 (`heapq` max-heap functions), and call out Python-specific traps such as the recursion limit and unbounded integers.

## Creating a new track

1. Copy `aspnet-core/index.html` to `new-track/index.html`.
2. Change `<title>`, meta description, the hero `<h1>` and lede.
3. Change the accent tokens (`--accent`, `--accent-soft`, `--accent-ink`) in all three colour blocks: light `:root`, the dark media query, and `:root[data-theme="dark"]`.
4. Change the localStorage key to a new namespaced key.
5. Replace the topic data and the progress track.
6. On the landing page, turn the planned tile into a live `.track` tile (copy the ASP.NET Core tile, set `data-src`, `--acc` and `--soft` to the track's colour tokens, and update the chips and static counts). Rebalance spans: with two live tracks, give each `grid-column: span 3`.
7. Add a row to the README table.

Track colours are defined in the landing page: `--t-dotnet`, `--t-dsa`, `--t-react`, `--t-backend`, `--t-sys`, each with a `-soft` variant, in all three colour blocks. The landing page's count script reads each tile's `data-kind` (`level`, `difficulty` or `depth`) from a table, so a new track kind needs a table entry rather than a new branch.

## Content rules

1. One idea per answer, sized to be said aloud in one to two minutes.
2. Every question has a level and a source basis:
   - **Cited:** a page actually retrieved while writing, linked in the answer and in the topic's sources.
   - **Widely reported:** a recurring pattern with no single citable page.
   - Never attribute a question to a company or platform (Glassdoor, AmbitionBox, TCS...) without a source that says so. Forum and community claims are labelled as reported.
   - Each topic's sources section says which pages were opened in full and which were seen only as search excerpts.
3. Framework facts follow official documentation. Mark version-specific behaviour (".NET 10 changed...") and prefer current APIs (HybridCache, built-in OpenAPI, `IExceptionHandler`), naming the older one as context.
4. Traps are the common wrong answer, stated plainly, not trick questions.
5. Cross-reference other topics by name ("see the EF Core and SQL topic"), never by number.
6. Sentence case headings, no emoji in content, no all-caps labels.

## Checks before committing

```bash
# 1. HTML parses
python3 -c "import html.parser; [html.parser.HTMLParser().feed(open(f).read()) for f in ['index.html','aspnet-core/index.html','react-typescript/index.html','dsa/index.html','backend/index.html']]; print('html ok')"

# 1b. No duplicate DSA problem ids (prints nothing when clean)
grep -o 'data-id="[^"]*"' dsa/index.html | sort | uniq -d

# 1c. No duplicate backend stage or topic ids (prints nothing when clean)
grep -o 'id:"[^"]*"' backend/index.html | sort | uniq -d

# 2. Every inline script has valid syntax (needs Node)
for f in aspnet-core react-typescript backend; do
  python3 -c "import re,sys; s=open('$f/index.html').read(); [open(f'/tmp/chk{i}.js','w').write(b) for i,b in enumerate(re.findall(r'<script>(.*?)</script>', s, re.S))]"
  for j in /tmp/chk*.js; do node --check "$j" || exit 1; done
  rm -f /tmp/chk*.js
done && echo "script ok"

# 2b. Every backend topic has its required fields, and no stub or over-wide snippets
node -e '
const fs=require("fs"), s=fs.readFileSync("backend/index.html","utf8");
const m=s.match(/<script>([\s\S]*?)<\/script>/)[1];
const lit=m.slice(m.indexOf("const stages"), m.indexOf("const DEPTH"));
const stages=new Function(lit+"; return stages;")();
let n=0, bad=[];
for (const st of stages) for (const t of st.topics||[]) { n++;
  for (const f of ["id","name","depth","line","what","use","concepts","how"])
    if (!t[f]) bad.push(t.id+": missing "+f);
  if (!["understand","build"].includes(t.depth)) bad.push(t.id+": bad depth");
  if (!t.concepts || t.concepts.length < 3) bad.push(t.id+": needs 3+ concepts");
  for (const c of t.code||[]) {
    if (!c.code || c.code.trim().length < 20) bad.push(t.id+": stub snippet");
    if (c.code.split("\n").some(l => l.length > 92)) bad.push(t.id+": snippet line over 92 chars");
  }
}
console.log("topics:", n);
if (bad.length) { console.error(bad.join("\n")); process.exit(1); }
console.log("fields ok");'

# 3. No leftover artifact links.
#    docs/superpowers/ is excluded because the planning notes in there quote
#    this very command, so they match themselves.
grep -rn "claude.ai/artifact" --include=*.html --include=*.md \
  --exclude=CLAUDE.md --exclude-dir=superpowers . || echo "clean"

# 4. Preview (the landing page's live counts need http, not file://)
python3 -m http.server 8000
```

Then check by eye: every topic renders, the level filter and "Open all answers" work, the backend route's four filters work and its ticks and DSA ticks survive a reload, light and dark mode, and a phone-width window. A runtime error (for example an undefined source constant) leaves the page empty below the toolbar, so always open the page after editing.

## Publishing

After the first push, publishing is:

```bash
git add -A
git commit -m "Describe the change"
git push
```

GitHub Pages redeploys within a few minutes (Actions tab, "pages build and deployment"). Settings → Pages should say "Deploy from a branch: main, / (root)".

If the repo is renamed, update the Source links in `index.html`, `aspnet-core/index.html` and `README.md`. The landing page counts each track's questions or problems by fetching its page, so it shows live numbers over http and the static fallback numbers when opened from disk.
