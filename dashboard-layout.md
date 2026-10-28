# Local LLM Benchmarks: dashboard layout

All tests run in oMLX on a MacBook Pro M3 Pro (36 GB). † marks a quick run (30 questions instead of the full set).

## Header
- **Title**: Local LLM benchmarks.
- **Takeaway line**: the model with the highest average among the models with the most full benchmark runs, its average %, and how many benchmarks that average covers.

## 1. Scores by model (table)
- **Row**: one model.
- **Columns**: MMLU-Pro, ARC-C, HumanEval, TruthfulQA, BBQ (any new benchmark is added after these), then Average.
- **Cell**: % correct from that model's latest full run of that benchmark. If only a quick run exists, the cell shows its score with †. Darker = higher. — = not run.
- **Average**: mean of the model's full-run scores only (quick runs excluded).

## 2. Speed vs accuracy (scatter)
- **Point**: one model. x = generation speed (TG tok/s) at 4k context; y = average of MMLU-Pro, ARC-C, HumanEval and TruthfulQA.
- **Filled point**: all four scores are full runs. **Hollow point with †**: some are quick runs.
- **Line under the chart**: models with a speed run but not all four core scores, with their TG tok/s at 4k.

## 3. Ranking (bar chart)
- **Selector**: pick one benchmark or Average.
- **Bar**: one model's score on it, longest first, colored by model family. Faded bar with † = quick run.

## 4. Speed (table)
- **Row**: one model's latest speed run.
- **Cells**: tokens per second. TG = generation (writing the reply), PP = prompt processing (reading the input), each at 1k and 4k context; Run = when it ran.

## 5. Time and answer length (table)
- **Row**: one model, from its exported runs.
- **Core minutes**: total time for the four core benchmarks (full runs only).
- **MMLU-Pro / HumanEval tok/answer**: average tokens the model wrote per answer.
- **Answers cut off**: answers stopped at the length limit, across all its exported runs.
- Model name in italics with † = some of these numbers come from quick runs.

## 6. By category (table)
- **Selector**: BBQ or MMLU-Pro.
- **Row**: one BBQ bias category or one MMLU-Pro subject.
- **Column**: one model (its latest full run; † = quick run).
- **Cell**: accuracy in that category. Darker = better; outlined cell = that model's weakest category.

## 7. All runs (table)
- **Row**: one benchmark run, newest first, including runs replaced in the cards above.
- **Cells**: date, model, benchmark, score % († = quick run), correct, total questions, minutes, tokens per answer, thinking on/off, notes (cut-off answers, scoring issues, unknown times).
