# PrepOps

Interview preparation tracks with answers you can say out loud, the follow-ups that usually come next, and the traps that catch most candidates. Written for developers interviewing in India.

**Live site:** https://harsh07may.github.io/interview-prep/

| Track | Covers | Status |
|---|---|---|
| [ASP.NET Core](aspnet-core/) | Fundamentals, C# and OOP, EF Core and SQL, REST APIs, authentication and authorization, performance, caching and security, architecture and testing, and a guide to how interview rounds run | Live, about 180 questions |
| [DSA patterns](dsa/) | Prerequisites in Python, 25 problem-solving patterns from hashing to bitmask DP, and a mixed set with no pattern labels | Live, 285 problems |
| React and TypeScript | Hooks, rendering, state, and the type system | Planned |
| System design | Designing for scale | Planned |

## Using the tracks

**ASP.NET Core**
- Every question is tagged Fresher (0–2 years), Mid (2–5 years), or Senior (5+ years). Filter to your level and skim one level above.
- Answers start closed. Say your answer aloud, then open the card.
- "Predict the output" and "Write the query" cards hide the answer behind a second click.
- Each answer is labelled with its source basis: a linked source, or a widely reported pattern.

**DSA patterns**
- Work through it in order: each pattern builds on earlier ones, and every problem names what to solve first.
- Each pattern has recognition cues, a Python template, and problems grouped as canonical, variations, and combinations.
- Key insights stay hidden until you open them. The final mixed set hides the pattern too.
- All problems are free on LeetCode. Tick problems off as you solve them.

Your level filter and solved problems are saved in your browser's `localStorage` and never leave your machine.

## Structure

```
index.html              landing page (bento grid)
aspnet-core/index.html  ASP.NET Core track
dsa/index.html          DSA patterns track
```

Each track is one self-contained HTML file with no build step. Fonts load from Google Fonts.

## Accuracy

Framework details target .NET 8 and .NET 10 and were written in September 2026. The DSA sheet's Python was tested on Python 3.12: templates were run, and key insights were checked against brute-force solutions on random inputs. Company-specific attributions come from public interview write-ups and are labelled as reported, not verified.
