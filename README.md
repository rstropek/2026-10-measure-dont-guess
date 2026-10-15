# Measure, don't guess , Slidev deck

Talk: "Messen statt raten: Das passende LLM systematisch auswählen"
LLMs im Unternehmen (Mastering AI / heise iX), online, 15 Oct 2026, 11:15–12:00

```bash
npm install
npm run dev      # presenter view with speaker notes: /presenter
npm run export   # PDF (needs playwright-chromium)
```

- `slides.md` · slides + speaker notes (`<!-- … -->` at the end of each slide)
- `style.css` · design (one ink, one accent, one warning colour)
- `public/results.svg` · per-candidate requirements table (**placeholder data**, generated)
- `speaker-notes.md` · all notes, printable
- `archive/slides-v1-de.md` · first German draft
- `public/pareto.svg` · unused (old draft), can be deleted

## Before the talk , replace the placeholders

Slides "How much model do we need?" and "Just enough model" use invented numbers (yellow badge).

1. Run every candidate with the SAME eval files and the SAME pinned judge:
   ```bash
   novedu-cli eval "evals/*.eval.yaml" --repeats 3 --json --out <row>.json --report <row>.md \
     --judge-llm-provider "Azure Foundry" --judge-llm-model gpt-5.6-terra \
     [--llm-provider <p> --llm-model <m> [--llm-reasoning <level>]]
   ```
2. Per row compute: agreement and false-correct as per-call rates (mean over repeats),
   95 % CI with n = number of cases, cost per 1,000 answers (tokens × price, or GPU-h ÷ throughput).
3. p95 latency is NOT measured by the CLI → separate load test (GuideLLM / `vllm bench serve`).
4. Check every candidate actually exists on its provider (e.g. Gemma 4 26B-A4B on SCCH).
5. Update `public/results.svg` (generator values) and the "Just enough model" cards. If the data
   disagrees with the placeholder story: change the pick, not the story. Remove the yellow badges.

- [ ] Link / QR code on the final slide.
