# Speaker notes: Measure, don't guess

Slides and notes in English. Timing target about 35 min plus Q&A.


## 1. Measure, don't guess

[~1 min · total ≈ 35 min]
- Welcome. Short intro: software architects, IT-Visions Austria.
- This morning: software for local LLMs (Winkler), GPU sizing (TNG). My slot is the question in between: WHICH model goes on that GPU, and how much model do you really need?


## 2. “Which model should we use?”

[~1.5 min]
- A question I get almost every week from customers. Optional chat poll: how did you pick your current model?
- Click through the three typical answers. We've all given one of them.
- Remember answer #2, "the biggest one that fits". We'll come back to it at the very end.
- → Transition: "Why is this question so hard right now? Because the list of candidates keeps growing."


## 3. The list of candidates keeps growing

[~1.5 min]
- Just a few open-weight releases of this year (dates verified): Mistral Small 4 (16 Mar, 119B MoE, 6B active, Apache 2.0), Gemma 4 (2 Apr, E2B…31B, first Gemma under Apache 2.0), Kimi K3 (weights 27 Jul, 2.8T), Qwen 3.8 (3 Aug; 27B under Apache 2.0, first Max-tier model with downloadable weights under a custom licence).
- Each one: several sizes × quantizations (BF16, FP8, INT4, GGUF) × serving stacks (vLLM, SGLang, llama.cpp) × providers.
- So "which model is best" has no stable answer. What we need is a way to answer it again and again.
- → Transition: "Many of these are called 'open'. Before we compare them, what does 'open' actually mean?"


## 4. “Open” is not one thing

[~2 min]
- API only: you rent a black box (here: an Azure Foundry deployment).
- Open weights with own licence: e.g. Llama 4 Community Licence, >700M monthly active users need a separate licence, acceptable-use policy, "Built with Llama" attribution. Qwen 3.8 Max open weights: separate licence for model-as-a-service offerings above $50M revenue (reported, eWeek Aug 2026).
- Open weights with OSI licence: Gemma 4 (Apache 2.0 since April 2026, earlier Gemmas had custom terms), Mistral Small 4, Qwen 3.8 27B.
- Open source in the strict sense, OSI Open Source AI Definition 1.0 (Oct 2024): weights + code to train and run + enough information about the training data to rebuild it. Examples: Ai2's OLMo, Swiss AI's Apertus.
- For an on-prem decision: weights are the technical prerequisite, the licence is a selection criterion (gate), we'll see it again.
- → Transition: "So: many candidates, different kinds of 'open'. What will you get out of the next 30 minutes?"


## 5. What you'll get today

[~0.5 min]
- Explicitly: no universal ranking, no hardware recommendation. A decision process.
- One running example throughout, let me introduce it.


## 6. Our example: Novedu, an AI learning platform

[~1.5 min]
- Novedu: learning platform for an Austrian technical secondary school (HTL). Four kinds of activities: tutors, quizzes with open answers, writing coach, coding help. (PRODUCT.md, teacher guide)
- Focus today: the quiz GRADER. The teacher writes a question and grading rules in YAML (the rules stay on the server). The student answers in their own words. An LLM decides correct / partial / incorrect and writes feedback.
- The model is part of the YAML: `llm: provider / model / reasoning`. Switching = one line. (docs/ai-models.md)
- Why it's a good example for companies: it's a classification + text generation task with real stakes, like ticket triage or document checks.
- → Transition: "And Novedu has the same hosting options you have."


## 7. Where can the grader run?

[~1.5 min]
- Three providers behind ONE OpenAI-compatible interface in Novedu: SCCH (self-hosted vLLM server, Austria-only network), OpenRouter (hosted multi-vendor router), Azure Foundry (GPT-5.x deployments).
- The sample quizzes use `RedHatAI/gemma-4-31B-it-FP8-Dynamic` on SCCH. FP8 = quantized: part of the model choice.
- The middle column is where many companies start: rent a GPU, run the same vLLM image, measure before buying hardware.
- THIS is the question of the talk: bigger, smaller, or just right? (My question, not a Novedu roadmap item.)
- → Transition: "First instinct: look it up somewhere."


## 8. Why not just look it up?

[~0.3 min] Chapter 1 of the agenda.


## 9. Leaderboards show what providers choose to show

[~1.5 min]
- "The Leaderboard Illusion" (Cohere Labs, Stanford, Princeton, MIT, Ai2…, arXiv:2504.20879): Meta tested 27 private variants before Llama 4; private testing lets providers publish selectively. LMArena disputed parts, be fair.
- Also: proprietary labs got far more arena data (Google ~19 %, OpenAI ~20 %) than 83 open-weight models combined (~30 %).
- Model cards: licence, context length, languages → shortlist input.
- And our task, German answers from 16-year-olds about sorting algorithms, is on no leaderboard. Neither is your ticket system.
- → Transition: "And even a perfect leaderboard measures someone else's installation of the model."


## 10. Same model, different provider, different result

[~1.5 min]
- Artificial Analysis "Endpoint Accuracy Index" (4 Aug 2026): same open weights, many serverless providers, compared with their own reference deployment. Some endpoints 22 % vs. 37 % on a tool-calling benchmark (BFCL). GLM-5.2: restrictive output limits → half the reference on a reasoning benchmark.
- Causes: quantization, custom kernels, chat templates, tool-call parsing, token limits, and bugs.
- Same on-prem: your quantization, your vLLM version, your template are part of "the model".
- Jan Jikeli at 14:15 takes this further for coding agents.
- → Transition: "So we have to measure ourselves. Four questions."


## 11. So we measure ourselves, with four questions

[~0.5 min]
- Chapter 2 of the agenda. Each question starts with a moment from Novedu, then the principle for your company.


## 12. What is “good enough”?

[~0.2 min]


## 13. What if the grader says “correct” to a wrong answer?

[~1.5 min]
- The question (abridged) and the answer are real: Novedu's Bubble Sort sample quiz and the "incorrect" example answer from the teacher guide (translated from German). The answer describes Selection Sort.
- Click: what IF the grader said "correct"? (Imagined, we'll see what really happened later.)
- Novedu calls this the false-correct rate: wrong answers that the grader accepted. The teacher guide: "usually the number worth acting on".
- Not all errors are equal: too strict annoys students; too lenient teaches them something wrong.
- In your company: the support bot that confirms a refund you don't offer.
- → Transition: "So 'good enough' is not one accuracy number. It's a set of minimum requirements, decided up front."


## 14. Write the requirements down first example values

[~2 min]
- Values are illustrative, agree yours with the business and write them down BEFORE the run. Otherwise you rationalise your favourite model.
- Domain quality (false-correct, agreement), technical (latency under realistic load), operations (cost), legal (data residency, licence, remember the "open" slide; AI Act: AI that evaluates learning outcomes can be high-risk, obligations from 2 Dec 2027).
- Speed only means something under load: a whole class submits at the same time. Measure with GuideLLM or `vllm bench serve`.
- Cost: amortised GPU hours for self-hosted, token prices for APIs. Full TCO → Haar/Koch at 15:30.
- Rare-error gates need many test cases: 0 errors in 40 wrong answers only proves "≤ 7 %". Rule of thumb: about 3 / threshold → ~75 wrong answers to show ≤ 4 %.
- → Transition: "A requirement is only as good as the test cases behind it."


## 15. What do we test with?

[~0.2 min]


## 16. Test cases: example answers with the expected grade

[~1.5 min]
- Real Novedu format (teacher guide "Test how your quiz grades"), translated from German: the file sits next to the quiz, `target` points to it, each case = question + invented answer + expected grade.
- Click 1: `expect` can be a list, "when more than one mark is genuinely defensible". Ambiguity is data, not noise.
- Click 2: the Selection Sort answer from two slides ago, now a permanent test case.
- Privacy: the guide says never paste a real student's answer; surprises from class are added "rewritten in your own words".
- In your company: domain experts write or label cases; anonymise anything taken from logs.
- → Transition: "Three answers are a demo. Here's what turns test cases into an asset."


## 17. A good test set covers four kinds of cases

[~1.5 min]
- Novedu's guide: 3–4 answers per question you care about, one clearly right, one half-right, one confidently wrong. "The confidently wrong ones earn their keep."
- Start with ~50 real cases (anonymised logs, domain experts, past incidents; ideally two people label independently). 50 cases show big differences (~15 points). Grow from there.
- Version them, keep them private (otherwise they end up in training data).
- → Transition: "Test cases aren't always single answers. For a tutor, they're conversations."


## 18. Novedu's AI tutors: the teacher scripts a conversation

[~1.5 min]
- Novedu also has AI tutors. Real format (docs/cli-eval.md): the teacher scripts the whole conversation, ending with a student message. The model writes exactly ONE reply; it gets checked. Stateless and repeatable.
- Per-case rule (`grading_instructions`): here "must NOT contain a complete working loop".
- Optional deterministic check: did the tutor call its tool (e.g. random number generator)?
- Novedu principle: "Pedagogy over answers". Being maximally helpful IS the failure here, so the strongest model is not automatically the best tutor. Measure it.
- The red reply is what we test for, not a recorded output.
- In your company: the agent that should ask before it acts.
- → Transition: "We have test cases. Now: who decides whether an answer passed?"


## 19. How do we score the answers?

[~0.2 min]


## 20. The response was valid, yet the student saw a broken text

[~2 min]
- Real: Novedu issue #115. The text is one of the real truncated outputs kept in the unit test (lib/quiz-feedback-truncation.unit.test.ts).
- The model wrote inline code around a quoted value like `"lightblue"`. vLLM's guided JSON decoding treated that quote as the END of the string. Feedback cut off, JSON still valid → invisible to every check that only parses the response.
- Tell: an unclosed code span = odd number of backticks. On the 572-string corpus that surfaced it: all real truncations found, no false positives.
- Mitigation, not cure: one retry; if still cut off, the first verdict is kept and a telemetry event fires. Deliberately the SAME code path in the student flow and the eval, "an eval must measure what a student would actually have been shown".
- Callback: "you evaluate a system", here it is in our own house: the serving software is part of the model.
- In your company: test what the user sees, not just the field you parse.
- → Transition: "Once you know a failure, you turn it into the cheapest possible check."


## 21. Score with code first, with a model last

[~1 min]
- Push as much as possible to the top.
- Novedu: the GRADE is compared with the expected grade → this decides pass/fail of the test run. The FEEDBACK TEXT is checked by a second model ("judge") → reported, never blocks.
- Humans: measure how often judge and humans agree before trusting the judge. Novedu's judge prompt was validated against ~100 real example answers plus deliberately planted mistakes.
- Judges have biases (position, length, own model family) → pin one judge model, preferably from a different family than the candidates.
- → Transition: "Where does the judge get its rules from?"


## 22. The judge needs no new rules: it reuses the grader's instructions

[~2 min]
- Payoff of the "imagined" slide: in the teacher guide's example report, the grader marked our Selection Sort student "incorrect", correct grade. (The guide's real run on Gemma 4 31B FP8: 3/3 grades right, 0 of 2 wrong answers accepted, 1 feedback flagged.)
- But the course rule says: if not correct, STATE the correct answer. The feedback asked a Socratic question instead. Grade right, feedback broke a rule, only the judge saw it (criterion `ignores_instructions`). Translated from German.
- Key idea (lib/quiz-feedback-judge.ts): the judge gets the grader's own system prompt as its standard. No extra rubric to write.
- Design rules: the judge doesn't re-grade; when in doubt it doesn't flag; it must name a concrete issue, no bare "not ok".
- Use a strong judge: a small model judging itself flagged noise; a strong judge caught every planted violation without noise (docs/cli-eval.md; small probe set, qualitative).
- → Transition: "Now we can score. Last question: how do we decide?"


## 23. How do we decide?

[~0.2 min]


## 24. A candidate is more than a model name

[~1.5 min]
- Everything speaks the OpenAI-compatible API: vLLM, rented GPU, Azure, OpenRouter. So a candidate = endpoint + model name + settings.
- Quantization is not an ops detail: FP8 vs. BF16 is its own candidate. Studies: FP8 practically lossless, 4-bit mostly fine, reasoning models more sensitive below 4 bit, but only YOUR cases tell you.
- Rented GPU: run the same vLLM image before you buy hardware, the cheapest way to answer TNG's sizing question with your own data.
- → Transition: "Settings sound harmless. One of them surprised us."


## 25. Same setting, different behaviour

[~1 min]
- Real measurement in the Novedu repo (docs/ai-models.md), one prompt, house models.
- Qwen 3.8: real ladder (xhigh ≈ 2.4× low), but rejects OpenAI level names like "high" with HTTP 400.
- Gemma 4: thinks, but every level except "none" produced byte-identical reasoning. "high" buys nothing.
- gpt-5.4-mini: monotonic ladder. gpt-5.6-terra even rejects "minimal" → deployment-specific.
- And: "none" ≠ leaving the setting out (default for Qwen is the maximum).
- → Transition: "In Novedu, running all candidates is one command each."


## 26. What happens in a test run

[~1.5 min]
- This is the mechanics behind everything so far.
- 1 Load: the test file (YAML/CSV): input + expected result + optional per-case rules.
- 2 Ask: send the request to the candidate through the OpenAI-compatible API, using EXACTLY the prompt production uses. Novedu builds it from the teacher's YAML with the same code as the live app. If the test prompt differs from production, you test something else.
- 3 Check: cheap code checks first (valid format, text complete), then compare the grade with the expected grade, then the judge reviews the feedback text.
- 4 Repeat: language models aren't deterministic; 3 repeats show whether a grade is stable. They do NOT count as extra test cases.
- 5 Report: totals (agreement, wrong accepted, with uncertainty) plus every failure with question, answer and model output, so a human can look at it.
- Speed and cost: measured separately, see "Speed and cost need their own measurement".
- → Transition: "In Novedu, all five steps are one command."


## 27. One command runs all test cases

[~1 min]
- Real output from Novedu's teacher guide ("Test how your quiz grades"): 3 test cases, 1 repeat, the sample quiz on Gemma 4 31B FP8. "SCCH" is the name of the self-hosted server.
- It replays the REAL production grading path: same prompt (built from the teacher's YAML), same output format, same retry for cut-off feedback. Prints the number of model calls before it starts.
- The flagged feedback is the Selection Sort case from the previous slide.
- Exit code → can gate CI. But Novedu's CI can't reach the models (server only reachable from Austria, CI without secrets) → tests run where the models are. On-prem lesson: you need a test runner inside your network.
- → Transition: "And a different candidate?"


## 28. Testing another candidate is one more option

[~1 min]
- Click 1: the candidate, provider + model for the whole run; report header shows "quiz-llm → override-llm", so a comparison is never mistaken for the baseline.
- Click 2: the judge is pinned to ONE model for every candidate, otherwise each candidate judges itself. Novedu's docs recommend a strong judge (gpt-5.6-terra). Caveat: if the judge is also a candidate, watch for self-preference, ideally use a judge from another model family.
- Latency is not measured by this tool → separate load test (GuideLLM / vllm bench serve) against the same deployment.
- → Transition: "Run this for every candidate and you get…"


## 29. Open-source frameworks do the same

[~1.5 min]
- For companies without a custom eval tool: open-source frameworks do the same five steps. Example: promptfoo (MIT licence; acquired by OpenAI in March 2026, the open-source repo stays MIT).
- Click 1: the prompt, loaded from the same file production uses.
- Click 2: the test cases as CSV: one row per case, with the expected grade.
- Click 3: three candidates, three ways of hosting: own vLLM server, rented GPU running the same vLLM image, commercial API. All OpenAI-compatible, so only the base URL and model name differ.
- Click 4: the checks, from cheap to expensive: valid JSON, expected grade, then an LLM judge with a rubric.
- Run: `npx promptfoo eval --repeat 3`, view results side by side with `npx promptfoo view`. Pin the judge model via `defaultTest.options.provider`.
- This is a sketch: model names and URLs are placeholders, adapt to your deployment.
- Other options: Inspect AI (UK AI Security Institute, Python, strong for agents and multi-turn), DeepEval (pytest style), Microsoft.Extensions.AI.Evaluation for .NET teams.
- Keep the test cases tool-neutral (CSV/YAML/JSONL): frameworks come and go, your cases stay.
- → Transition: "Quality is one axis. Speed and cost need their own measurement."


## 30. Speed and cost need their own measurement

[~1.5 min]
- The quality test calls the model one case at a time; that says nothing about speed under load. A whole class submits at the same moment.
- Speed: load test against the SAME deployment (same quantization, same vLLM version) with a realistic token profile from your logs (e.g. long prompt, short answer). Report p95, not the average. Tools: GuideLLM (vLLM project), `vllm bench serve`. More on GPU sizing: TNG's talk this morning.
- Cost, self-hosted: hourly price of the GPU (bought: amortised; rented: list price) divided by the answers per hour you reached at the required speed. Cost, API: tokens per answer from the usage report × price.
- Novedu's eval CLI reports token usage per run, but no latency; the load test is a separate step.
- Full TCO (staff, operations, idle time): Haar/Koch at 15:30.
- → Transition: "Put everything into one table."


## 31. Every candidate, every requirement placeholder data

[~2 min]
- ⚠ PLACEHOLDER. Replace with tonight's run (see README): agreement + false-correct per candidate from `novedu-cli eval … --repeats 3 --json`; 95 % interval with n = number of test cases; cost per 1,000 answers; p95 latency from a separate load test.
- Reading it: one row per candidate. The bar is the 95 % range of agreement with the teacher, a candidate passes only if the WHOLE bar is right of the 88 % line, AND the other requirements hold. Red = requirement missed.
- Story with placeholder data: the small mixture-of-experts model is cheapest and fastest but accepts too many wrong answers; GPT-5.4 mini misses agreement and accepts too many wrong answers; GLM looks good but data would leave the EU; GPT-5.6 is best, but 12× the cost and too slow under load. FP8 vs. BF16: same quality, about half the cost.
- Cost per 1,000 answers: tokens × price for APIs; GPU hours ÷ answers per hour for self-hosted. p95 = 95 % of answers are faster than this, measured with 30 parallel users (separate load test).
- → Transition: "Wait, is Qwen at 95 really better than Gemma at 94?"


## 32. Is 95 % better than 94 %? placeholder data

[~1.5 min]
- Bars = 95 % confidence interval: 1.96 × √(p(1−p)/n). At ~90 % and n = 120 → about ±5 points. 100 cases at 80 %: ±8. 500 cases: ±3.5. (Miller, "Adding Error Bars to Evals", arXiv:2411.00640)
- "Can't tell apart" ≠ "no difference". Compare paired on the same cases (McNemar/bootstrap) to be more precise.
- Repeats measure how much the model wobbles, they don't shrink these bars. n = number of test cases.
- If you tried many configurations, confirm the winner with a fresh run.
- Rule: within noise, the cheaper, faster, more sovereign option wins.
- → Transition: "Which gives us our answer."


## 33. The answer: just enough model placeholder

[~1.5 min]
- ⚠ Placeholder conclusion, if tonight's data disagrees, change the pick, not the story.
- Decision rule: requirements first → among the survivors the cheapest / fastest / most sovereign → keep a margin.
- Why not Qwen? Within noise of Gemma, but more tokens (reasoning) → more cost and latency. Why not GLM? Data residency.
- The big API model is our CEILING: it shows how much better it could get at all. If even the best model misses a requirement, fix prompt or test data first.
- Option: routing, most traffic to the small self-hosted model, the hardest cases to a big one.
- Document it: requirements, eval report, all versions → decision record.
- → Transition: "And next month there's a new model. Then what?"


## 34. A process you can rerun

[~0.2 min] Chapter 3 of the agenda.


## 35. A new model is a new run, not a new project

[~1 min]
- While I prepared this talk: Gemma 4, Mistral Small 4, Kimi K3, Qwen 3.8. Next month: more.
- Once harness and test cases exist, a new candidate costs a few GPU hours and one review. The first setup is the investment.
- Novedu: eval files next to the activities; the CLI warns when its version doesn't match the server's; "what you tested is what you must publish".
- → Transition: "Four things for Monday."


## 36. Start on Monday

[~1 min]
- 50 cases show big differences (~15 points). Grow from there.
- Callback to the opening: "the biggest one that fits on our GPU", now we know better.
- If no model passes: that's where fine-tuning comes in → Christian Winkler's workshops (22/29 Oct). GPU vs. API cost → Haar/Koch at 15:30.


## 37. Measure, don't guess.

Last spoken words: "Models come and go. Your test cases stay. Measure, don't guess." Then the thank-you slide.


## 38. Thank you!

- Put the QR code (slides, example eval files) into the dashed square: e.g. replace the div with <img src="/qr.png" class="qr" />.
- Prepared answers:
  - "How many test cases?" → uncertainty slide; start with 50, grow to hundreds for fine differences.
  - "Which judge locally?" → the strongest model you can run, from another family than the candidates, checked against human labels.
  - "Agents?" → evaluate the whole system; Jikeli 14:15.
  - "Cost of evals?" → a judge doubles the calls; Novedu prints the number of calls first; smoke run without judge.


## 39. Appendix: tools

- Backup for Q&A. Keep test cases tool-neutral (YAML/JSONL), harnesses are replaceable, cases are not.
