# AIOS Consultation Integration (Reference Only)

These three modules are the **AIOS-side consumers** that wire HYDRA's per-clone feeds into expert consultation prompts. They are NOT part of HYDRA core and are NOT installable from this repository — they reference the AIOS framework directly.

They are included here purely as illustrative reference, to show how a consumer system pulls HYDRA's distributed feeds into runtime prompts.

## Files

| File | Role |
|------|------|
| `self-consultation.js` | Entry point that builds expert consultation prompts. Calls `injectHydraResearch()` to enrich the system prompt with the latest 3 insights from the expert's HYDRA feed. |
| `consultation-engine.js` | Topic-based expert recommendation. Searches the 162-expert index (55 Mega Brain + 107 AIOS) by topic/agent/project. |
| `project-detector.js` | Resolves the current project context (tocks, serenity, bretda, low-ticket-10k, etc.) from cwd and recent git activity. |

## Integration shape

```
User prompt
  ↓
AIOS @agent invokes self-consultation.js
  ↓
project-detector → identifies project
  ↓
consultation-engine → picks expert(s) by topic
  ↓
self-consultation → reads HYDRA feed: D:/jarvis/mega brain/agents/minds/{expert}/hydra-research/recent.md
  ↓
LLM prompt = base persona + last-3 HYDRA insights + user question
  ↓
Expert answers with FRESH research (not frozen training knowledge)
```

## Why this matters

HYDRA's value proposition is the **anti-hallucination loop**: experts answer with research published yesterday, not training data from 2024. The consumption integration is what closes that loop. Without it, HYDRA is just an RSS aggregator with a fancy scoring layer.

## To adapt for your own framework

The HYDRA core (in `../../src/`) emits markdown feeds to `feeds/{clone-id}/recent.md`. Any consumer needs to:

1. **Locate the feed**: read `feeds/{expert-id}/recent.md` (path depends on your `OUTPUT_DIR` env).
2. **Parse top N entries**: each entry is a YAML+markdown block with `score`, `summary`, `source_url`, `published_at`.
3. **Inject into prompt**: prepend the top 3 to your expert's system prompt as fresh context.

That's the entire contract — HYDRA writes markdown, your consumer reads markdown.

See `../../docs/03-architecture/` for the formal ADRs on streaming pattern, vector search, and observability.
