# CLAUDE.md

Everything needed to understand, edit and publish this repository. Read it fully before changing anything.

## What this repo is

Interview preparation tracks plus a landing page, served as a static site on GitHub Pages. No build step, no framework, no package manager: every page is one self-contained HTML file.

| Path | What it is | localStorage key |
|---|---|---|
| `index.html` | Landing page: bento grid linking to all tracks | none (reads each track's source to count questions) |
| `aspnet-core/index.html` | ASP.NET Core track, 8 topics, about 180 questions | `aspnet:level:v1` (selected level filter) |
| `README.md` | Public description | |
| `.nojekyll` | Tells GitHub Pages to serve files as-is. Do not delete. | |

Planned tracks: `dsa/`, `react-typescript/`, `system-design/`. They appear on the landing page as dashed "Planned" tiles until they exist.

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
- `predict:true` cards put the `code` (a snippet, schema, or prompt) up front and hide `a`. Use them for output prediction, query writing, code review, and scenarios.
- Source-link constants (like `CWM`, `SQLQ`) are defined once near the top of the script. Define a constant before any topic uses it; an undefined one stops the whole page rendering.

### Adding a topic

1. Add the parts array, a `topicNRevise` list and `topicNSources` HTML before `const topics = [`.
2. Add the topic entry to `topics`.
3. Add a link to the progress track `<nav class="track">` and update the track note.
4. Update the landing page chips and the "N topics" text.

## Creating a new track

1. Copy `aspnet-core/index.html` to `new-track/index.html`.
2. Change `<title>`, meta description, the hero `<h1>` and lede.
3. Change the accent tokens (`--accent`, `--accent-soft`, `--accent-ink`) in all three colour blocks: light `:root`, the dark media query, and `:root[data-theme="dark"]`.
4. Change the localStorage key to a new namespaced key.
5. Replace the topic data and the progress track.
6. On the landing page, turn the planned tile into a live `.track` tile (copy the ASP.NET Core tile, set `data-src`, `--acc` and `--soft` to the track's colour tokens, and update the chips and static counts). Rebalance spans: with two live tracks, give each `grid-column: span 3`.
7. Add a row to the README table.

Track colours are defined in the landing page: `--t-dotnet`, `--t-dsa`, `--t-react`, `--t-sys`, each with a `-soft` variant, in all three colour blocks.

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
python3 -c "import html.parser; [html.parser.HTMLParser().feed(open(f).read()) for f in ['index.html','aspnet-core/index.html']]; print('html ok')"

# 2. The track's script has valid syntax (needs Node)
python3 -c "import re; s=open('aspnet-core/index.html').read(); open('/tmp/track.js','w').write(re.findall(r'<script>(.*?)</script>', s, re.S)[0])" && node --check /tmp/track.js && echo "script ok"

# 3. No leftover artifact links
grep -rn "claude.ai/artifact" --include=*.html --include=*.md --exclude=CLAUDE.md . || echo "clean"

# 4. Preview (the landing page's live counts need http, not file://)
python3 -m http.server 8000
```

Then check by eye: every topic renders, the level filter and "Open all answers" work, light and dark mode, and a phone-width window. A runtime error (for example an undefined source constant) leaves the page empty below the toolbar, so always open the page after editing.

## Publishing

After the first push, publishing is:

```bash
git add -A
git commit -m "Describe the change"
git push
```

GitHub Pages redeploys within a few minutes (Actions tab, "pages build and deployment"). Settings → Pages should say "Deploy from a branch: main, / (root)".

If the repo is renamed, update the Source links in `index.html`, `aspnet-core/index.html` and `README.md`.
