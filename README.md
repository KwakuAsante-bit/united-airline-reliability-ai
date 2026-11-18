# Can We Trust AI to Write Executive Reports?

*AI-assisted airline executive reporting: a United Airlines case study with January 2025 BTS data*

Alex Asante and Faye Williams
INFO 4340, Fall 2026, University of Denver

## Overview

**Business question:** can generative AI reliably turn verified airline performance data into executive reports that
give leadership useful insight without adding unsupported claims?

We built a workflow in which Python calculates every figure, Claude drafts a one-page monthly executive report for
United Airlines from those verified figures, and a human reviews the report before it reaches leadership. The airline
analysis is the case study that supplies the evidence: airline on-time rankings compare carriers that fly very
different airports and routes, so we used one month of U.S. DOT flight data to test whether United's headline ranking
tells the full story. We then measured how often the AI's claims are unsupported, and set governance rules for when a
human must review a report before it is released.

## Key findings

- **Rankings depend on the method.** United was 5th of 14 airlines overall (82.37% on time) but 3rd on shared routes,
  +3.53 percentage points above its route competitors; 13 of 14 airlines changed rank under the shared-route comparison.
  Neither of United's two weakest priority routes was significant after multiple-testing correction (0 of 2).
- **The AI is mostly accurate, but adds causes.** 8 of 194 claims were unsupported: **4.1% (95% CI 2.1% to 7.9%)**.
  7 of the 8 were causal claims the data cannot support; 1 was a wrong number.
- **Human agreement was low before discussion:** 86.6% raw agreement, **Cohen's κ = 0.13 (95% CI −0.04 to 0.32)**;
  all 26 disagreements were resolved together.
- **The automated checker alone is not enough:** precision 0.38 (0.15 to 0.64), recall 0.75 (0.50 to 1.00),
  F1 0.50 (0.25 to 0.73). Combining checker flags with low cross-run consistency caught **8 of 8** errors while sending
  25.8% of claims to human review.
- **Cost:** about **$0.013 per report** (21,936 input and 5,769 output tokens for 12 reports). Differences between
  prompt variants (largest 7.3 points) are below the minimum detectable difference (11.3 points).

## Data

- **Source:** U.S. Bureau of Transportation Statistics, *Reporting Carrier On-Time Performance (1987–present)*,
  January 2025: <https://www.transtats.bts.gov/DL_SelectFields.aspx?gnoyr_VQ=FGJ&QO_fu146_anzr=b0-gvzr>
- **Size:** 539,747 scheduled domestic flights, 14 reporting airlines, 329 origin airports.
- **Unit of observation:** one scheduled flight. The AI evaluation's unit is one claim (a sentence or table row) in an AI report.
- **Access:** the original BTS download (27 MB zip) is included in this repo, and the notebook reads it directly.
  The 243 MB extracted CSV is not committed (too large for GitHub). To download it yourself, choose January 2025 on the
  page above with "Prezipped File" checked and save the zip into the repo's top folder.

## Workflow

1. **Cleaning:** select the needed columns, build route IDs, count cancellations and diversions separately, and use
   completed flights as the on-time denominator; assertions check the row count, the month and duplicates.
2. **Analysis:** airline on-time rates with confidence intervals, departure-airport exposure, the flight-volume
   correlation, shared-route rankings, and route-level two-proportion tests with multiple-testing correction.
3. **AI step:** Claude Sonnet 4.6 (`claude-sonnet-4-6`), default sampling settings, `max_tokens` = 1400. The prompt supplies
   only verified figures and asks for a fixed one-page report: executive summary, scorecard with "—" for unavailable
   history, and two monitored routes. It forbids causal claims and recommendations.
4. **Evaluation:** 4 prompt variants × 3 runs = 12 reports, split into 194 claims. Both partners labeled every claim
   independently (κ), then reached consensus; we measured the error rate (Wilson CI), checker precision/recall/F1
   (report-level bootstrap), cost, and the minimum detectable difference between variants.
5. **Recommendation:** launch the workflow as a supervised pilot, using the AI draft only under three back-tested
   governance rules (claim review, a report release gate and prompt escalation). The governance memo also proposes a
   quarterly review against a target of at least 98% supported claims before human correction; the initial sample was
   95.9%, and this target has not yet been tested quarterly.

## Repo contents

- `final_project.ipynb`: the full analysis, AI step and evaluation; runs top to bottom.
- `final_project.html`: the executed notebook.
- `requirements.txt`: Python packages.
- `On_Time_Reporting_Carrier_On_Time_Performance_1987_present_2025_1.zip`: the BTS data.
- `ai_outputs/`: saved Claude outputs (`baseline_report.json`, `eval_reports.json`, with token counts and model).
- `evaluation/`: the claims with checker flags, both partners' label workbooks, the consensus labels, verified values,
  the report texts, and the governance rules (the AI's original draft and our revised version).
- `executive_report_source_register.csv`: where each figure in the executive report comes from.

## How to run it

1. Clone this repo and install the packages (tested with Python 3.13):
   `pip install -r requirements.txt`
2. Open `final_project.ipynb` and run all cells.
   **By default the notebook reads the saved AI outputs in `ai_outputs/` and does not call the API**, so no key is needed.
3. (Only to regenerate AI outputs) set an environment variable named `ANTHROPIC_API_KEY` (or a Colab secret with that
   name), and delete or change the saved files in `ai_outputs/`. Never put the key in the notebook.

## Limitations

- One month of data (January 2025): no trends, and winter weather may affect airlines unevenly.
- Airport exposure uses departure airports only, and it is an association (r = +0.44, p ≈ 0.11, n = 14), not a cause.
- The evaluation is small: 194 claims, and 7 of the 8 errors come from one sentence template.
- The governance thresholds were chosen and tested on the same 12 reports; they need re-testing on new months.
- The Wilson CIs and the MDD treat claims as independent, so they are approximate.
- Results apply to Claude Sonnet 4.6 and these prompts; a model or prompt change needs a new evaluation.

## AI use

We used Claude Code to help build the Python notebook and Claude Sonnet 4.6 to generate the reports evaluated in this
project. ChatGPT assisted with governance and presentation development, while Codex assisted with project reviews,
speaker notes, and drafting and revising the written summary; our individual AI use logs document each partner's
decisions and verification.
