# Backend engineering route — design

Status: approved design, implementation pending.
Date: 2026-11-12.
Branch: `claude/dazzling-mccarthy-0px7mf`.

## 1. What this is

A fourth track for PrepOps (`harsh07may/interview-prep`): a practical, progressive
**learning route** for backend engineering, as opposed to the three existing tracks,
which are interview question banks or a problem sheet.

It teaches what a backend engineer needs to build, debug, operate and scale real
services. It is not an interview question list, and it does not duplicate the DSA
track (algorithms and Big-O are cross-referenced there, never restated here).

Lives at `backend/index.html`, one self-contained HTML file, no build step, served
by GitHub Pages like every other page in this repo.

## 2. Decisions taken

| Decision | Choice |
|---|---|
| Path | `backend/index.html` |
| Track name | Backend engineering. Kicker: "Server-side craft" |
| Colour | New landing tokens `--t-backend` / `--t-backend-soft`: `#0F766E` / `#D7F0ED` light, `#5ED3C4` / `#10332F` dark. Teal, clear of .NET purple, DSA azure, React blue, sys orange, and the level greens |
| localStorage | `backend:done:v1` (ticked topic ids), `backend:filter:v1` (toolbar filter) |
| Code policy | Language-neutral pseudocode by default. Real code only where it is genuinely concrete: **Node as primary**, **C# as secondary**, plus real SQL, YAML, Dockerfile, nginx and shell. No framework is assumed by the route |
| Shape | 15 stages, ~101 topics, plus a capstone spec |
| Axes | Each topic carries `depth` (`understand` or `build`) and `core` (boolean). Toolbar filters: All / Core only / Understand / Build |
| System design track | The planned `sys` tile stays. Stage 15 cross-references it by name rather than absorbing it |

## 3. Page design

Reuses the repo's visual language (hero, phase-dot track, sticky toolbar, 760px
column, highlight.js, back-to-top, the same three colour blocks) but introduces a
**new renderer**: a data-driven topic card, plus DSA-style progress ticks brought
into the data-driven engine for the first time.

### Hero

Six phase dots, not fifteen, so the track stays legible:

Foundations (stages 1-2) · Build the service (3-6) · Secure and fast (7-8) ·
Async and integrate (9-11) · Operate (12-14) · Scale (15 + capstone)

The fifteen stages appear in the jump nav and as body sections.

### Toolbar

```
[ All topics ][ Core only ][ Understand ][ Build ]    12 of 101 done   Open all topics
```

Filter persists to `backend:filter:v1`. Hiding a topic hides it from its stage's
visible count, and a stage with nothing visible hides itself, matching the
existing `applyFilter` behaviour in `aspnet-core` and `react-typescript`.

### Topic card

Collapsed by default, like every other track in this repo: you commit to an answer
before opening. The closed state shows enough to read the page as a curriculum map.

```
+-----------------------------------------------------------+
| [x]  Idempotency and safe retries        [Core] [Build] v |
|      One line: what it is                                 |
+-----------------------------------------------------------+
| What & why       prose, 2-4 sentences                     |
| +-- In practice -------------------------------------+    |
| |   concrete scenario, accent-tinted box             |    |
| +----------------------------------------------------+    |
| Core concepts    term -> one-line definition list         |
| How to build it  prose + snippet(s) + config/SQL          |
| Trade-offs       when to choose each approach             |
| ! Pitfall        the mistake people actually make         |
| > Exercise       nested disclosure, closed                |
| References                                                |
+-----------------------------------------------------------+
```

The tick checkbox sits outside `<summary>` so ticking does not toggle the card.

### Data shape

```js
{ id:"idempotency",                    // stable anchor + progress key: never rename
  name:"Idempotency and safe retries",
  depth:"build",                       // "understand" | "build"
  core:true,                           // part of the ~40-topic spine
  line:"One line shown on the closed card",
  what:`<p>What it is and why it matters.</p>`,
  use:`<p>A concrete scenario.</p>`,
  concepts:[ ["Term","One-line explanation"], ... ],
  how:`<p>Implementation guidance.</p>`,
  code:[ {label:"Node", lang:"js", code:`...`},
         {label:"Postgres", lang:"sql", code:`...`} ],   // optional, array
  tradeoff:`<p>When to choose each.</p>`,                // optional
  pitfall:"The mistake people actually make.",           // optional
  exercise:`<p>A small task.</p>`,                       // optional
  refs:[ ["Label","https://..."] ] }                     // optional
```

Stage shape:

```js
{ num:1, id:"foundations", phase:1, name:"Runtime foundations",
  intro:"One line.", topics:[ ... ],
  after:[ ["You can now...","..."] ],   // reuses .revise styling
  sources:`<p>...</p>` }
```

### Rendering rules

- `code` entries are inserted with `textContent`, so snippet source is written raw.
  Code inside `what`, `how`, `tradeoff` is HTML and must escape `<`, `>`, `&`.
- `langFor()` maps `js`, `cs`, `sql`, `yaml`, `bash`, `json`, `dockerfile`, `nginx`.
  Values highlight.js 11.9.0's common bundle does not carry (`http`, pseudocode)
  render unhighlighted, deliberately.
- Stage progress line: "4 of 7 done". Overall count in the toolbar.
- Per-stage "After this stage you can" block, then a collapsed sources `details`.

## 4. Route inventory

101 topics. Ids are the progress keys and anchors: **never rename a published id.**

### Stage 1 - Runtime foundations (8)
1. The path of a request, end to end (absorbs DNS and TCP)
2. HTTP semantics: methods, status codes, headers, negotiation
3. Server concurrency models and what one request costs
4. Linux for backend engineers: processes, signals, file descriptors, limits
5. Configuration and secrets
6. Layering and dependency boundaries
7. Serialization and data contracts (UTC, decimals, null vs absent, field evolution)
8. Statelessness and where state actually leaks

### Stage 2 - Networking and the edge (4)
1. TLS, certificates and where encryption ends
2. Connections: keep-alive, pooling and HTTP/2 multiplexing
3. Reverse proxies and load balancers at the edge
4. Real-time: WebSockets, SSE and long polling

### Stage 3 - Designing APIs (9)
1. Resource modelling and URL design
2. Validation at the boundary
3. Error model design
4. Collection endpoints: pagination, filtering, sorting
5. Versioning and backward compatibility
6. Idempotency and safe retries
7. Rate limiting and quotas
8. Choosing an API style: REST, gRPC, GraphQL
9. OpenAPI that stays true

### Stage 4 - Data modelling and the database (10)
1. SQL you need fluently
2. Relational modelling and deliberate denormalization
3. Keys, constraints and integrity in the database
4. Indexes
5. Reading query plans
6. Transactions and isolation anomalies
7. Locking, deadlocks and lost updates
8. Connection pooling and pool exhaustion
9. Zero-downtime schema migrations
10. ORMs and the N+1 problem

### Stage 5 - The data layer in production (4)
1. Choosing a data store
2. Batching, bulk writes and server-side cursors
3. Soft deletes, audit trails and time-based data
4. Read replicas and replication lag

### Stage 6 - Testing and quality (6)
1. What to test at which level
2. Integration tests against real dependencies
3. Test doubles, and what not to mock
4. Flaky tests and determinism
5. Contract testing between services
6. Load, stress and soak testing

### Stage 7 - Auth, authorization and security (9)
1. Sessions vs tokens, and cookie flags
2. Passwords and credential handling
3. Tokens in practice: JWT validation, expiry, refresh, revocation
4. OAuth 2.1 and OpenID Connect
5. Authorization models: RBAC, ABAC, ownership, tenant isolation
6. Service-to-service auth
7. Untrusted input: injection, SSRF, traversal, deserialization
8. CORS, security headers and browser-facing concerns
9. Data protection: encryption, key rotation, PII, and what never goes in a log

### Stage 8 - Caching and performance (9)
1. Cache layers and the CDN
2. Strategies: cache-aside, write-through, write-behind
3. Keys, TTLs and invalidation
4. Stampedes and hot keys
5. HTTP caching and ETags
6. Redis beyond caching
7. Finding the bottleneck: profiling and percentiles
8. Memory, GC and leaks
9. Why p99 goes bad before CPU does

### Stage 9 - Concurrency and async work (6)
1. Async I/O in your code: what actually yields, and how pools starve
2. Background jobs: moving work out of the request
3. Scheduled work, overlapping runs and singletons
4. Shared state: races, atomic updates and compare-and-swap
5. Distributed locks and leases
6. Concurrency limits and backpressure

### Stage 10 - Messaging and event-driven architecture (8)
1. Queue vs pub/sub vs log, and delivery semantics
2. At-least-once and consumer idempotency
3. Ordering, partitioning and consumer groups
4. Retries, dead-letter queues and poison messages
5. The outbox pattern and dual writes (CDC folded in)
6. Event schemas and versioning
7. Sagas and compensation
8. Read models and projections (CQRS scoped; event sourcing as a warning)

### Stage 11 - Storage and external integrations (6)
1. File uploads
2. Object storage and pre-signed URLs
3. Streaming large payloads, imports and exports
4. Consuming third-party APIs
5. Receiving webhooks
6. Sending webhooks

### Stage 12 - Reliability and fault tolerance (5)
1. When a dependency fails: timeouts, retries, jitter and budgets
2. Circuit breakers, bulkheads and degraded modes
3. Readiness, liveness and graceful shutdown
4. Error handling that aids diagnosis
5. Backups, restore drills and rollback

### Stage 13 - Observability and debugging (5)
1. Structured logs and correlation IDs
2. Metrics: golden signals, RED/USE, histograms over averages
3. Distributed tracing and context propagation
4. Alerting, SLOs, runbooks and postmortems
5. Debugging production systematically

### Stage 14 - Deployment and production engineering (7)
1. Containerising a service properly
2. Pipeline: build once, promote the artifact
3. Release strategies and feature flags
4. The Kubernetes objects you actually touch
5. Resource limits, autoscaling and OOMKills
6. Infrastructure as code
7. Cloud primitives and least-privilege IAM

### Stage 15 - Distributed systems you'll actually meet (5)
1. Monolith, services and the distributed-monolith trap
2. Horizontal scaling, sharding and partition keys
3. Consistency models and what eventual consistency costs you
4. Replication, leader election and quorum as things you operate
5. Clocks, ordering and why timestamps lie

### Capstone
An order service. Eight milestones, each with: what to build, the topics it
exercises, and an acceptance check the learner can run. The per-topic exercises in
stages 3-14 are written so they compose into it.

## 5. Excluded on purpose

Recorded so later sessions do not re-add them.

| Excluded | Why |
|---|---|
| DSA, Big-O, data structure internals | The repo has a 285-problem DSA track. Cross-reference, never restate |
| Raft and Paxos internals | Kept as "consensus and quorum as things you operate". You tune quorum; you do not write consensus |
| Multi-region architecture as a topic | A trade-off paragraph in consistency models. Most backend engineers never build it |
| Event sourcing as a recommendation | Sidebar warning inside read models, with the cases where it is right |
| Standalone GraphQL and gRPC topics | One substantial "choosing an API style" topic, with dataloader/N+1 and deadlines/streaming treated concretely inside it |
| OWASP Top 10 as a topic | A list whose items live in five security topics. Linked as reading |
| Standalone Prometheus, Grafana, log-level topics | Named as the concrete toolchain inside the metrics and logging topics |
| Normal-form taxonomy (1NF/2NF/3NF drill) | Taught as its one practical rule inside relational modelling |
| Resource management as its own topic | Was a grab-bag: fds to Linux, pools to the database stage, sizing to backpressure |
| Profiling in observability | Duplicated the performance stage; lives there only |

## 6. Accuracy policy

Follows `CLAUDE.md` rule 2. Nothing is labelled as a retrieved source unless it was
actually opened.

- A short list of high-risk factual claims gets verified against primary sources and
  cited: OAuth 2.1's PKCE requirement, Postgres' default isolation level, the
  OpenTelemetry propagation header, specific HTTP status semantics, Redis command
  behaviour. Each gets a real link.
- Everything else is labelled as a reference not retrieved while writing.
- Node snippets are run before publishing. SQL is checked against a real engine if
  one is available in the container, and marked illustrative if not. Pseudocode is
  labelled as pseudocode so it is never mistaken for runnable code.
- No claim is attributed to a company or platform.

## 7. Repository integration

| File | Change |
|---|---|
| `backend/index.html` | New |
| `index.html` | Add `--t-backend` / `--t-backend-soft` to all three colour blocks. Add a live `.track.backend` tile with `data-src="backend/"`. Rebalance spans: notes move to `span 3` so four tracks, the planned tile and three notes tile cleanly. Generalise the count script to read `data-keys` off the tile instead of hard-coding two keysets. Update `<title>` and the meta description. Update the footer's version line to mention Node |
| `README.md` | Table row, a "Using the tracks" entry, structure block, accuracy note |
| `CLAUDE.md` | Table row with the new localStorage keys, a "How the backend route works" section covering the data shape and `langFor()`, `--t-backend` in the track colours line, `backend/index.html` added to the HTML-parse and script-check commands |

Generalising the landing count script is required integration, not unrelated
refactoring: the existing script cannot count a two-bucket track.

## 8. Build order and status

One stage per commit, so the page is never half-rendered and a later session can
resume from this list.

- [ ] Shell: head, colour blocks, CSS, hero, toolbar, renderer, progress, `langFor()`
- [ ] Stage 1 - Runtime foundations
- [ ] Stage 2 - Networking and the edge
- [ ] Stage 3 - Designing APIs
- [ ] Stage 4 - Data modelling and the database
- [ ] Stage 5 - The data layer in production
- [ ] Stage 6 - Testing and quality
- [ ] Stage 7 - Auth, authorization and security
- [ ] Stage 8 - Caching and performance
- [ ] Stage 9 - Concurrency and async work
- [ ] Stage 10 - Messaging and event-driven architecture
- [ ] Stage 11 - Storage and external integrations
- [ ] Stage 12 - Reliability and fault tolerance
- [ ] Stage 13 - Observability and debugging
- [ ] Stage 14 - Deployment and production engineering
- [ ] Stage 15 - Distributed systems you'll actually meet
- [ ] Capstone
- [ ] Landing page integration
- [ ] README and CLAUDE.md
- [ ] Final review pass: duplication, sequencing, accuracy, link check

## 9. Verification

Per `CLAUDE.md`, before each commit:

```bash
python3 -c "import html.parser; [html.parser.HTMLParser().feed(open(f).read()) for f in ['index.html','aspnet-core/index.html','react-typescript/index.html','dsa/index.html','backend/index.html']]; print('html ok')"
python3 -c "import re,sys; s=open('backend/index.html').read(); [open(f'/tmp/b{i}.js','w').write(b) for i,b in enumerate(re.findall(r'<script>(.*?)</script>', s, re.S))]" && for f in /tmp/b*.js; do node --check "$f" || exit 1; done && echo "script ok"
grep -o 'id:"[^"]*"' backend/index.html | sort | uniq -d   # stage and topic ids; prints nothing when clean
grep -rn "claude.ai/artifact" --include=*.html --include=*.md --exclude=CLAUDE.md . || echo "clean"
```

Then by eye: every stage renders, all four filters work, ticks survive a reload,
light and dark mode, phone width. A runtime error leaves the page empty below the
toolbar, so the page is opened after every stage is added.
