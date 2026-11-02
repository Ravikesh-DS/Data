# CSAT/NPS voice scoring - summary

Score version: voice_v0.1 | weights {'text': 0.6, 'csat': 0.2, 'nps': 0.2} | cap 0.3 | text source: baseline TF-IDF+Ridge

## Findings

- **A1** NPS on 12.01% / CSAT on 45.05% of interactions; both-together ratio 1.002 vs 1.0 if independent -> surveys issued independently
- **A2** NPS mix: promoters 25.7%, passives 31.3%, detractors 42.9% -> NPS index -17.2
- **A3** NPS response rate flat across all 9 dimensions (max spread 0.49 pp, max V 0.0033) -> MCAR on observables; survey presence is uninformative
- **A4** 10.87% of customers have any NPS in 12 months; 90.4% of them only once -> NPS is a snapshot, not a monthly per-customer series
- **A5** Of active customer-months: 31.52% have an interaction, 14.81% a CSAT, 4.05% an NPS
- **A6** Text sentiment explains 66% of NPS variance; Spearman(CSAT,NPS)=0.68 -> NPS is interaction-driven; CSAT and NPS overlap (cap combined survey weight)
- **A7** ~39% of NPS variance is between customers (stable component); the rest is event-driven
- **A8** Within positive-labelled texts NPS ranges 7.79-8.48 by issue clause -> GT text label misses issue severity; NPS adds that signal
- **A9** Inactive in next 3m: detractors 33.18% vs promoters 23.97% (lift 1.38x); gap persists within the same text label -> NPS adds predictive signal beyond text
- **A10** Avg NPS ranges 5.9-8.42 across loyalty tiers -> never adjust a customer's score by tier; compare against own-tier benchmarks in reporting
- **A11** 7,351 NPS responses on PENDING interactions - confirm when the survey is sent (no survey timestamp in data)
- **B** Only 360 distinct texts -> templated text; text metrics on random splits are optimistic (see V1 template hold-out)
- **F1** Columns not found (features depending on them are left NULL and skipped): engagements.converted
- **F1** 1 configured mapping value(s) not found in the data: event=sms_opt_out -> review F0 mappings
- **G1** NPS vs transactions: strongest feature n_tx (rho +0.062, negligible); 14/14 transaction features negligible (|rho| < 0.1)
- **G2** NPS vs engagements: strongest feature avg_session_sec (rho +0.100, negligible); 16/16 engagement features negligible
- **G3** NPS detractor vs promoter lift - no TX 1.15x, no ENG 1.21x, no INT 1.13x, fully inactive 1.38x -> read each dataset's outcome separately, not only the combined one
- **G5** NPS combined: detractors lower TX? yes - small effect (d=+0.14); lower ENG? yes - small effect (d=+0.17); delta AUC (fully inactive) beyond text+behaviour -0.0011
- **H1** CSAT vs transactions: strongest feature n_tx (rho +0.061, negligible); 14/14 transaction features negligible (|rho| < 0.1)
- **H2** CSAT vs engagements: strongest feature avg_session_sec (rho +0.099, negligible); 16/16 engagement features negligible
- **H3** CSAT dissatisfied vs satisfied lift - no TX 1.15x, no ENG 1.21x, no INT 1.12x, fully inactive 1.37x -> read each dataset's outcome separately, not only the combined one
- **H5** CSAT combined: dissatisfieds lower TX? yes - small effect (d=+0.14); lower ENG? yes - small effect (d=+0.16); delta AUC (fully inactive) beyond text+behaviour -0.0002
- **I1** Active months without an interaction: NPS 0.0% / CSAT 0.0% -> both surveys only exist where there is an interaction; TX/ENG-only months must be scored on behaviour
- **I2** Feature-importance ranking agreement NPS vs CSAT (Spearman of |rho|): 0.99; mean |rho| by dataset (both-observed) - engagements: NPS 0.044 vs CSAT 0.047; interactions: NPS 0.239 vs CSAT 0.242; transactions: NPS 0.031 vs CSAT 0.032
- **I4** Fully-inactive-next AUC (both-observed, test): behaviour+text 0.7017, +CSAT -0.0001, +NPS -0.0003, NPS given CSAT -0.0003, CSAT given NPS -0.0001
- **I6** Largest relative NPS-vs-CSAT gap (SD units vs own mean): text_label_pred=neutral (+0.03); rank agreement - channel 0.54, interaction_type 0.83, resolution_status 0.80, text_label_pred 1.00
- **I7** 5.4% of both-observed interactions show strong CSAT/NPS disagreement - CSAT measures the interaction, NPS the relationship; keep both components (capped) rather than substituting one for the other

## Validation

| check | metric | value | threshold | status |
|---|---|---|---|---|
| V0 | customers in both splits | 0.0 | == 0 | PASS |
| V1 | MAE text vs GT score (customer hold-out) | 0.164 |  | INFO |
| V1 | Spearman text vs GT score | 0.8611 | >= 0.7 | PASS |
| V1 | Accuracy label | 1.0 |  | INFO |
| V1 | Macro-F1 label (customer hold-out) | 1.0 | >= 0.8 | PASS |
| V1 | Weighted-F1 label | 1.0 |  | INFO |
| V1 | Macro-F1 label (unseen-template hold-out) | 1.0 | >= 0.7 | PASS |
| V1 | Spearman (unseen-template hold-out) | 0.9691 |  | INFO |
| V2 | Spearman text vs NPS | 0.7235 | >= 0.5 | PASS |
| V2 | AUC detractor (lower text = detractor) | 0.8943 | >= 0.75 | PASS |
| V2 | AUC promoter | 0.8216 |  | INFO |
| V2 | Spearman text vs CSAT | 0.7318 |  | INFO |
| V2 | 3-class agreement text label vs NPS category | 0.6115 |  | INFO |
| V3 | AUC future inactivity - text only | 0.5665 |  | INFO |
| V3 | AUC future inactivity - text + NPS | 0.5692 |  | INFO |
| V3 | Delta AUC from adding NPS | 0.0027 | >= 0.0 | PASS |
| V4 | AUC inactivity: default - text-only (test) | 0.0042 | >= 0.0 | PASS |
| V4 | Spearman with next NPS: default (test) | 0.4009 |  | INFO |
| V4 | Best grid weights (train) |  |  | INFO |
| V5 | Decile rho: score vs future inactivity | -0.8909 | <= -0.8 | PASS |
| V5 | Decile rho: leave-NPS-out score vs same-month NPS | 0.9879 | >= 0.8 | PASS |
| V6 | Error decreases low -> medium -> high | 0.0 | == 1 | FAIL |
| V6 | Spearman(confidence_score, -error) | 0.0046 |  | INFO |
| V7 | Max NPS response-rate spread across groups (pp) | 0.49 | <= 1.5 | PASS |
| V7 | |mean text| with NPS vs without | 0.0046 | <= 0.02 | PASS |
| V8 | Max |detractor AUC tier - overall| | 0.1086 | <= 0.05 | FAIL |
| V9 | Max PSI vs 2025-10 | 0.0012 | <= 0.1 | PASS |
| V9 | Lag-1 within-customer correlation (consecutive months) | 0.4578 |  | INFO |
| V10 | Health cross-check |  |  | INFO |