# ReviewLab — Content Moderation Review Prototype

A two-phase AI product project by Catarina Leandro. Phase 1 provides real text-model inference and a measured benchmark. Phase 2 is a defined visual-evidence extension, not an implemented feature.

## Product question

With a fixed number of human reviews, can we capture more toxic-labelled comments without unacceptable contextual false positives?

Primary users are content reviewers and moderation operations leads. The product ranks content for review; a person decides whether the content violates the prototype rubric. No content is removed and no account action is taken.

## Run the existing prototype

No build tools, API keys, inference service, or installation are needed to preview the exported model.

```sh
python3 -m http.server 8765 --bind 127.0.0.1 --directory dist
```

Open http://127.0.0.1:8765/. Keep the server running while using the local preview. Do not open index.html directly: the app loads local JSON assets through HTTP.

Seven workspaces:

1. **Review queue** — real benchmark comments, model selection, capacity controls, reference-label toggle, error filters, and human decisions.
2. **Experiment lab** — frozen comparison, uncertainty interval, category and contextual diagnostics, and offline/pilot decision.
3. **Import & score** — local CSV scoring or single-comment prediction. Imported comments are not sent to an inference service.
4. **Error review** — six real mistakes, feature evidence, historical v2 proposals (not current policy labels).
5. **Policy guide** — source-linked TikTok text rules, examples, decision boundaries and versioned review controls.
6. **TikTok results** — human-labelled calibration, frozen-cutoff flags, top-10 review workload and contextual false positives.
7. **Methodology** — product brief, annotation rubric, model/data card, limitations, and the two-phase roadmap.

## Measured result

The final sampled test contains 9,916 comments, including 797 comments with toxicity annotation fraction ≥ 0.5. At a review budget of 992 comments:

| Method | Toxic captured | Recall | Precision | False positives | Missed toxic |
|---|---:|---:|---:|---:|---:|
| Keyword rules | 253 | 31.7% | 25.5% | 739 | 544 |
| Word TF-IDF + logistic regression | 497 | 62.4% | 50.1% | 495 | 300 |
| Word + character TF-IDF + logistic regression | 528 | 66.2% | 53.2% | 464 | 269 |

The combined model was selected on validation recall at 10% capacity before test predictions were calculated. It captured 275 more toxic-labelled comments than keyword rules at the same review volume, a relative increase of 108.7%. The paired row-bootstrap 95% interval for recall improvement is +30.8 to +38.4 percentage points. This interval does not account for unavailable thread clusters.

**Product decision: continue offline validation; hold live routing.** The overall capture target passed, but the reference-negative identity-term slice had a 23.3% false-positive rate versus 11.2% for rules. This is a lexical diagnostic, not a demographic identity measurement or fairness certification. The sample contains only 22 threat positives, of which the selected queue captures 10; severe-toxicity performance cannot be assessed because there are no positives at the chosen label cutoff.

## Reproduce the benchmark

The measured environment used Python 3.14.6. Install dependencies in an isolated environment. Download requires network access and about 422 MB of public data.

```sh
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python ml/download.py
python ml/train.py
node ml/verify.cjs
```

The downloader stores data in `../data`, outside the website directory. Training honours `REVIEWLAB_DATA` as an optional replacement data directory. The included source record identifies the exact Hugging Face revision and SHA-256 file hashes. The downloader uses that pinned revision when this record exists.

Uniform sampling uses seed 42 for training, 43 for validation, and 44 for test. Samples were initially 60,000 / 8,000 / 10,000; empty and normalised duplicate removal left 59,690 / 7,944 / 9,916. No class balancing was applied. The chosen mirror does not provide thread/video IDs; full fuzzy near-duplicate and group leakage checks remain incomplete.

`ml/protocol.json` fixes feature settings, candidates, selection, capacities, and uncertainty procedure. `ml/rules.json` contains the frozen keyword baseline. Word and character TF-IDF branches each have at most 30,000 features. Each branch uses sublinear term frequency and its own L2 normalisation. Logistic regression uses C=2 with no class weighting.

Exports:

- `dist/data/model.json`: vocabularies, IDFs, coefficients, intercepts, and keyword rules for real browser inference.
- `dist/data/benchmark.json`: test comments, reference labels, and frozen model scores.
- `dist/data/report.json`: complete metrics, source provenance, sample audit, and limitations.
- `ml/model.joblib`: Python model bundle. Only load trusted serialized model files.
- `ml/verification.json`: model parity and metric checks.

## CSV import and reviewer feedback

Required column: `text`, `comment`, or `comment_text`. Optional: `label` (0/1), `toxicity` (0–1), and `caption` (display-only context). A nonempty toxicity fraction maps to label 1 at ≥0.5. Conflicting supplied labels are rejected. Limits: 5,000 rows, 5 MB, 20,000 characters per comment.

The queue evaluates imported data only when all rows have labels. It never treats predictions as truth. Use independently annotated labels. CSV comments stay in page memory and disappear on reload. Review decisions and notes persist only in local browser storage, keyed by comment IDs. Export them through the top-right button. No shared multi-user database is implemented. Reference labels never change when a reviewer records a decision.

## Phase 1 remaining work

1. Review the exported 50 false positives and 50 false negatives. This is a diagnostic sample selected by score, not an unbiased estimate of error prevalence.
2. Have two people independently annotate an initial 100-comment policy sample and resolve rubric disagreements.
3. Obtain a permitted TikTok sample across topics and videos. Preserve thread/video IDs and avoid sampling only offensive keywords.
4. Create a development split for adaptation and a separately held-out final TikTok test. Document any rubric/benchmark label mismatch.
5. Evaluate 500–1,000 independently labelled comments as an exploratory transfer study. Rare categories may require substantially more examples.
6. Run 3–5 reviewer usability sessions; measure task success, time, and agreement. None has yet been conducted.
7. Resolve material contextual false positives before a live pilot. For any future operational feedback loop, audit a random sample outside the high-score queue to reduce selection bias.

## Phase 2 plan and acceptance gate

1. Select one visual policy category with an appropriate licensed/permissioned labelled dataset.
2. Use genuine image-caption pairs. Treat this as a post-level decision, separate from the Phase 1 comment-level task.
3. Use pretrained ResNet50 as an image-classification/feature baseline. Its ImageNet scores are not harm probabilities, and it does not locate every object. A knife in a cooking image is an example of why tags alone do not establish a violation.
4. Record image evidence, extracted text if OCR is added, model versions, and associated captions. Keep them distinct from policy-risk estimates and human decisions.
5. Train or validate a policy-specific scoring layer using labelled paired data, selecting any signal-combination weights on validation only.
6. Compare text-only, image-only, and combined systems on the same held-out pairs at fixed review capacity. Measure recall, precision, contextual false positives, latency, cost, and missing-image behaviour.
7. Add visual evidence to the review workspace only if the combined system provides useful measured improvement. Begin with shadow-mode evaluation; full video understanding additionally requires temporal/audio work.

## Sources

- Civil Comments: https://huggingface.co/datasets/google/civil_comments
- Annotation and dataset documentation: https://www.tensorflow.org/datasets/catalog/civil_comments
- Threshold selection: https://scikit-learn.org/stable/modules/classification_threshold.html
- ResNet50: https://docs.pytorch.org/vision/main/models/generated/torchvision.models.resnet50.html

Civil Comments is released under CC0-1.0. The dataset contains offensive material. This prototype is independently developed, is not affiliated with TikTok, and does not claim TikTok performance.

## TikTok-aligned rubric v3

Guideline release: 25 August 2026. Effective: 24 September 2026. Verified directly on TikTok’s English (US) pages on 6 November 2026. This is a selected text-policy mapping, not a complete copy of TikTok policy or an official enforcement system.

The guide covers violence/harmful crime, hate, harassment, sexual harassment, privacy/doxing and self-harm promotion. It offers four actions: No violation in scope, Flag violation, Needs context, and Policy / specialist review. Lower-severity hostility is not automatically a removal violation; FYF restrictions, public-interest exceptions and adult public-figure context require a separate assessment. An urgent priority is a local record only and does not notify a safety service.

`dist/policy.js` is the shared source for the guide, reviewer reason codes, validation and policy metadata. `dist/policy-view.js` renders the dashboard and [printable guide](dist/rubric.html). The examples are synthetic teaching examples. Raw TikTok sample comments are not republished. TT056 and TT083 summaries retain the product owner’s No violation decisions after AI-assisted discussion, not independent blind annotations.

Each new decision records policy version, source URL, effective date, target context and priority. Flags and escalations require evidence notes, and reason/action mismatches are rejected. Editing a decision preserves earlier records; the CSV export distinguishes current and superseded rows. Old browser records are not silently relabelled.

The six-case error audit in `dist/data/error-review.json` remains a historical v2 analysis. Its original proposed decisions must be explicitly re-reviewed under v3 before use as policy evaluation labels. The learned weights, Civil Comments labels, benchmark metrics and original audit are unchanged. The model estimates general toxicity; this update does not train or validate any new category detector.

Official policy sources:
- https://www.tiktok.com/safety/en/policies-and-engagement/safety-civility
- https://www.tiktok.com/safety/en/policies-and-engagement/privacy-security
- https://www.tiktok.com/safety/en/policies-and-engagement/mental-behavioral-health
- https://www.tiktok.com/safety/en/policies-and-engagement/enforcement

`node ml/verify.cjs` checks model parity, metrics and policy decision/history invariants. Repeat a policy-source review when TikTok changes its guidelines; the version and verification date are deliberately explicit.

## GitHub and Vercel

The source repository is intended to remain private while the project is refined. Vercel uses `vercel.json`: the dependency-free verification command is the build step and `dist` is the static output directory. Training is not run during deployment. Runtime scoring uses the included learned weights in the visitor's browser. There are no API keys or external inference costs.

The private repository `catarinaleandro/reviewlab-content-moderation` is connected to the existing Vercel project. Pushes to `main` run the verification build and automatically update [the public dashboard](https://reviewlab-content-moderation.vercel.app). Other branches use preview deployments. The Vercel GitHub app is limited to this repository. Check the deployment result and commit in Vercel before considering a change released.

## TikTok human-label comparison (v1.3)

The first calibration batch has 100 final No violation decisions by Catarina under rubric v3, with zero unresolved cases. The model predicts Civil Comments toxicity, while these human labels apply a TikTok-aligned policy subset. This is a single-reviewer, shopping/lifestyle sample across 24 videos and 12 source topics. Five cases involved AI-assisted contextual discussion. It is not a representative or independent gold test.

The frozen Civil Comments validation cutoff (score >= 0.15794454885986295) flags five comments: TT097, TT002, TT036, TT066 and TT063. Against the human labels: FP=5, TN=95, TP=0, FN=0, precision=0/5 and false-positive rate=5/100. Recall is undefined because there are no positive labels. This is not evidence of perfect harm detection or a model winner.

At fixed top-10 capacity, each ranked approach selects ten non-violations. Every keyword-rule score is zero, so its top-10 ranking is only the deterministic tie-break. An any-rule-match diagnostic flags zero but uses a different review workload.

The TikTok results page distinguishes the two operating modes. Public cases are paraphrases and feature evidence; source comments, captions and identifiers remain in private review files. `dist/pilot-data.js` holds aggregate results and anonymous sample-level scores/labels. The original benchmark, model, policy data and AI audit are unchanged.

Recompute from the local review inputs (not bundled or published):

```sh
node ml/evaluate-pilot.cjs /path/to/scored.json /path/to/human-decisions.json /path/to/private-outputs
node ml/verify.cjs
```

The evaluator checks human-label completeness and policy consistency, recomputes all 100 scores, checks saved prediction parity, and records input hashes. `ml/verify-pilot.cjs` independently checks the released metrics, rankings, cutoff, missing-denominator behavior and absence of private source fields. No training or cutoff tuning happens during this comparison.

Independent second review, broader sampling and evaluation containing confirmed violations remain future steps at the product owner’s request.
