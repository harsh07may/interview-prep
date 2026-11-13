# PrepOps

Interview preparation tracks with answers you can say out loud, the follow-ups that usually come next, and the traps that catch most candidates. Written for developers interviewing in India.

**Live site:** https://harsh07may.github.io/interview-prep/

| Track | Covers | Status |
|---|---|---|
| [ASP.NET Core](aspnet-core/) | Fundamentals, C# and OOP, EF Core and SQL, REST APIs, authentication and authorization, performance, caching and security, architecture and testing, and a guide to how interview rounds run | Live, about 180 questions |
| [DSA patterns](dsa/) | Prerequisites in Python, 25 problem-solving patterns from hashing to bitmask DP, and a mixed set with no pattern labels | Live, 285 problems |
| [React and TypeScript](react-typescript/) | JavaScript and the event loop, hooks, rendering, state, TypeScript, the browser, HTTP and security, machine coding, frontend system design, and a priority guide for SDE-1 and SDE-2 | Live, 154 questions |
| [Backend engineering](backend/) | A practical route rather than a question bank: runtime and networking foundations, API design, data modelling, the data layer in production, testing, security, caching and performance, concurrency, messaging, storage and integrations, reliability, observability, deployment, distributed systems, and a capstone service | Live, 101 topics across 15 stages |
| System design | Designing for scale | Planned |

## Using the tracks

**ASP.NET Core**
- Every question is tagged Fresher (0–2 years), Mid (2–5 years), or Senior (5+ years). Filter to your level and skim one level above.
- Answers start closed. Say your answer aloud, then open the card.
- "Predict the output" and "Write the query" cards hide the answer behind a second click.
- Each answer is labelled with its source basis: a linked source, or a widely reported pattern.

**React and TypeScript**
- Ten topics ordered the way frontend rounds unfold, from JavaScript and async to machine coding. Topic 10 has the priority table and the 20 must-not-get-wrong list.
- Same card format as ASP.NET Core: level tag, follow-up, trap, and a source label. Snippets marked "predict" show the code first; every output-prediction snippet was run in Node.
- Filter to Fresher (SDE-1), Mid (SDE-2) or Senior.

**DSA patterns**
- Work through it in order: each pattern builds on earlier ones, and every problem names what to solve first.
- Each pattern has recognition cues, a Python template, and problems grouped as canonical, variations, and combinations.
- Key insights stay hidden until you open them. The final mixed set hides the pattern too.
- All problems are free on LeetCode. Tick problems off as you solve them.

**Backend engineering**
- A learning route, not interview questions. Fifteen stages in order, each assuming the ones before it, then a capstone.
- Laid out like a documentation site: the sidebar is the navigation, one stage shows at a time, and nothing is collapsed. Search covers all 101 topics; press `/` to focus it.
- Every topic is the same shape: two sentences on why it matters, a code example, four one-line points, and one-line trade-off, pitfall and exercise notes. About 150 words plus code, so you can actually read 101 of them.
- Each topic is labelled **Core** (the 46-topic spine), and **Build** or **Understand** — what you should be able to implement yourself versus what you should know the trade-off for.
- Tick topics off as you go. The capstone composes the exercises into one order service in eight milestones, each with a check you can run.
- **Node and Express throughout**, plus real SQL, YAML, Dockerfile and nginx config where those are the actual artifact.

Your level filters and solved problems are saved in your browser's `localStorage` and never leave your machine.

## Structure

```
index.html              landing page (bento grid)
aspnet-core/index.html  ASP.NET Core track
react-typescript/index.html  React and TypeScript track
dsa/index.html          DSA patterns track
backend/index.html      backend engineering route
```

Each track is one self-contained HTML file with no build step. Fonts load from Google Fonts.

## Accuracy

Framework details target .NET 8 and .NET 10 and were written in September 2026. The DSA sheet's Python was tested on Python 3.12: templates were run, and key insights were checked against brute-force solutions on random inputs. React and TypeScript targets React 19 and TypeScript 5.x. Documentation sites were not reachable while it was written, so only a few React claims are backed by documentation excerpts; every source label says which. Company-specific attributions come from public interview write-ups and are labelled as reported, not verified.

The backend engineering route was written in November 2026 and redesigned later that month. Its SQL was run against PostgreSQL 16 and its JavaScript against Node 22, both inside the authoring container, and the outputs quoted in the snippets are the real ones. Where a snippet could not be run there, the Dockerfile and the Kubernetes manifests among them, the page says so. Each stage's sources section separates pages that were actually retrieved from references that were not.
