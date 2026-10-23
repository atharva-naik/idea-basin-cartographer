# Idea Basin Cartographer

**arXiv → functional specs → t-STE → concrete research ideas**

Type a topic in plain English. The app scrapes arXiv, distills every abstract into a
`GIVEN / WHEN / THEN / MEASURED BY` functional spec (optionally judged by an LLM),
computes low-rank functional similarity (LSA), samples *"i is closer to j than k"*
triplets, fits a **t-distributed Stochastic Triplet Embedding**, clusters the result into
**idea basins**, detects **coverage gaps**, and drafts **concrete research proposals**
for the empty regions.

Zero dependencies. No build step. No server. One HTML file.

```
            ┌────────────┐   ┌──────────────┐   ┌─────────────┐   ┌──────────────┐
  query ──► │ arXiv Atom │──►│ functional   │──►│ LSA cosine +│──►│   t-STE      │
            │   scrape   │   │ spec extract │   │  triplets   │   │  2-D map     │
            └────────────┘   └──────────────┘   └─────────────┘   └──────┬───────┘
                                                                        ▼
                              ┌────────────────┐   ┌──────────────┐   ┌──────────────┐
                              │ concrete idea  │◄──│  coverage    │◄──│  k-means     │
                              │   proposals    │   │  gap finder  │   │   basins     │
                              └────────────────┘   └──────────────┘   └──────────────┘
```

## Run locally

```bash
# either just open it
open index.html

# or serve it (needed for clipboard + some fetch behaviour)
python3 -m http.server 8000
# → http://localhost:8000
```

## Deploy to GitHub Pages

```bash
gh repo create idea-basin-cartographer --public --source=. --push
# then: Settings → Pages → Build and deployment → Source: "Deploy from a branch"
#        Branch: main, folder: / (root) → Save
```

Manual equivalent:

```bash
git init && git add index.html README.md LICENSE
git commit -m "Idea Basin Cartographer: arXiv → specs → t-STE → gaps"
git branch -M main
git remote add origin https://github.com/atharva-naik/idea-basin-cartographer.git
git push -u origin main
```

Live at `https://atharva-naik.github.io/idea-basin-cartographer/`.

## URL modes

| URL | Effect |
|---|---|
| `?tour=1` | **Choreographed one-take demo**: types the query, scrapes arXiv, plays the cinematic embedding, scrolls to the concrete ideas. Built for screen recording. |
| `?tour=1&speed=0.7` | Slows the cinematic timeline (default `1.0`). Range `0.2`–`3`. |
| `?fullscreen=1` | Fullscreen mode: hides all chrome, plot fills the viewport. (`?cinema=1` still works as an alias.) |
| `?still=1` | Skip the cinematic, jump straight to the final map. |
| `?tour=1&speed=0.8` | ≈ 28 s take — the sweet spot for a tweet video. |

### Recording recipe (20–60 s tweet video)

1. Open the hosted URL (or `index.html` locally). Add `?speed=0.8` for a slower, more
   legible take (range `0.2`–`3`).
2. **Advanced → LLM judge → paste your key** first (memory-only). This is what upgrades
   the headlines to NeurIPS-level titles mid-take. Skip it for a keyless template take.
3. Start screen recording (Cmd+Shift+5 / OBS / ScreenStudio).
4. Click **🎬 Tour (record me)** — the page self-directs the whole take: **query typed →
   arXiv scraped → ideas settle into basins → contour topology → gaps flagged → real
   paper titles generated → scroll to the concrete proposals**.
5. Stop recording after the first proposal card glows (~25–35 s).

Convert a QuickTime `.mov` (or anything else) to MP4:

```bash
ffmpeg -i screen-recording.mov -c:v libx264 -pix_fmt yuv420p -crf 20 -movflags +faststart demo.mp4
```

`?tour=1` on the URL still auto-runs the tour with no clicks (keyless → template titles).

## BYOK LLM judge

Open **Advanced → LLM judge** and paste a key:

- **Anthropic** — paste an `sk-ant-…` key and the endpoint/model switch automatically to the
  native Claude Messages API (`https://api.anthropic.com/v1/messages`, default `claude-opus-5-5`;
  type `claude-haiku-5-5` in the model box for a cheaper run). Calls go straight from the
  browser to Anthropic.
- **OpenAI or any OpenAI-compatible endpoint** (OpenRouter, Groq, Together, vLLM…) — paste the
  key, and set the endpoint + model if it isn't OpenAI.

The key lives in the tab's **memory only** — never in the HTML, never in `localStorage`,
cleared on reload, and only ever sent to the endpoint you configured. Without a key the local
NLP extractor runs (free, offline, same schema).

**Titles.** Proposal headlines are drafted locally as “Mechanism: Application”. With a
key loaded, the app automatically generates **NeurIPS-level paper titles** for every
idea (≈$0.001 total on GPT-5.6 Luna) — or click **✦ Generate paper titles with LLM**.
Titles are cached, so parameter tweaks don't re-spend credits.

**Refine.** "✦ Refine with LLM" on any proposal uses the same key to rewrite the draft
into a sharper, named-papers-only research idea (with its own title).

## Stress test

**⚡ Stress test · 100 papers** loads 100 real arXiv titles (23 with full abstracts +
77 title-only, fetched from `export.arxiv.org`) to verify pipeline speed and basin
quality at scale. Stage-by-stage timings are logged.

## Parameters (all under Advanced)

| Knob | Default | Meaning |
|---|---|---|
| NN neighbours | 10 | near-set size when sampling triplets |
| triplets / anchor | 8 | constraints per paper |
| LSA rank | 6 | latent idea families for functional similarity |
| iterations | 500 | t-STE optimization steps |
| restarts | 4 | best-of-N local minima |
| basins (k) | 6 | k-means idea families |
| seed | 42 | full reproducibility |

**Underexplored** flags are evidence-based: thin sample share vs. a uniform `1/k` null
**and** scattered functional spacing. Singletons are never flagged. Hit **Calibrate on
arXiv** to replace sample statistics with real per-family field sizes from the arXiv
index (`opensearch:totalResults`, one request per basin, ≥3 s apart).

## Why t-STE (and not t-SNE)

The input is *ordinal* — "this idea is more like that one than the other" — which is
exactly the triplet format t-STE consumes directly. t-SNE would need a numeric
similarity and would distort density; t-STE preserves the ranking constraints that
define functional similarity. 2-D was benchmarked against 3-D/4-D: 2-D converged to
lower loss and higher triplet satisfaction across a full learning-rate sweep.

## arXiv attribution

Thank you to arXiv for use of its open access interoperability.

## References

- van der Maaten & Weinberger, *Stochastic Triplet Embedding* (2012) — https://lvdmaaten.github.io/ste/
- Ziakas et al., *Fork-and-Flush: Escaping Idea Basins in Autoresearch Agents* — https://arxiv.org/abs/2610.07447

## License

MIT