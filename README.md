# ReviewLab

**Which comments should a moderator review first?**

Catarina Leandro · Independent AI product work sample · November 2026

I gave a test queue two review slots. Model C flagged the only comment I had labelled a violation in a group of 16, but ranked it fifth. An affectionate remark about a dog and a message reassuring someone about their appearance took the first two slots. That result changed how I evaluated the product. I now count **which violations reach a person within the available review budget**, alongside precision and recall at the alert cutoff.

[Read the public case study](https://reviewlab-content-moderation.vercel.app/work-sample.html) · [Try the review queue](https://reviewlab-content-moderation.vercel.app/#queue) · [Compare the models](https://reviewlab-content-moderation.vercel.app/#models)

## Product stage and decision

**Phase 1 is in offline validation, before an operational pilot.** I defined the review problem, built a working text prototype with AI coding assistance, and acted as the project’s reviewer. I labelled 260 distinct archived TikTok comments across five batches, discussed ambiguous cases against a versioned rubric and tested changes to review order. Professional reviewer research and operational impact measurement are future work.

Model A powers the public demo. Models B and C run offline. My decision is to keep C and the one-slot rule experimental. The rule can move one already-flagged comment about possible physical harm to the front of the queue. Context helped on a small validation set, but the rule’s apparent benefit on a known mistake did not repeat on the next two batches. Phase 2 would add visual evidence for cases where text leaves a material question unanswered.

## What I tested

| Experiment | Observation | Product decision |
|---|---|---|
| **Establish a baseline.** Give keyword rules and a word-and-character text model the same 992 slots on 9,916 Civil Comments test examples. | Of 797 toxicity-labelled comments, the rules surfaced 253 and the text model surfaced 528. Recall at that budget rose from 31.7% to 66.2%. | Use the text model as the demo baseline. Evaluate transfer separately because TikTok policy labels answer a different question. |
| **Test whether context helps.** The first 100 TikTok comments all received a no-violation label, yet A flagged five. In the next 40, I used 26 for training, ten for validation and set aside four unresolved decisions. | With cutoffs chosen to catch both validation violations, B (comment only) flagged four of eight harmless comments. C (comment, caption and available parent reply) flagged one. | Take C into blind testing with its cutoff frozen at 0.3933 rounded. Two validation positives gave little evidence about other kinds of harm. |
| **Test a fresh queue.** I labelled 40 new comments before seeing C’s predictions. | C raised six alerts. One was the sole labelled violation and five were false alarms. One additional comment needed context. The violation ranked fifth for two review slots in the wording-selected group. | Investigate ranking and context errors. One positive gives too little evidence to estimate detection reliably. |
| **Challenge the proposed fix.** Give one already-flagged comment priority when its wording describes possible physical harm to a person. Compare both orders at identical capacity. | The rule moved the known example from fifth to first in replay. On two subsequent 40-comment batches, both methods picked the same five comments per batch. In the last batch, a revised human label left one violation, ranked fourth for two slots. | Keep Model A in the demo and retain C’s original order as the offline comparison. The replay motivated a hypothesis. The fresh comparisons have yet to demonstrate an improvement. |

The TikTok batches used public research archives and one reviewer. Some labels were refined through discussion. The final batch’s revision is separate from its original blind submission. Wording-selected samples help investigate rare cases and have a different mix of comments from an everyday queue.

## Assumptions behind the design

| Choice | Why I made it | How I would check it |
|---|---|---|
| Start with English comments and a human decision | I could review English comments against an English-language rubric. A reviewer can use the caption and parent reply to distinguish criticism, quotation and targeted abuse, or request more evidence. | Observe reviewers resolving the same cases, record what evidence they open and measure time and agreement. |
| Compare simple rules, A, B and C | Rules are easy to audit. A provides a low-cost reference. B and C use the same small sentence model, training rows and settings, so their comparison tests the value of adding context. | Freeze the models and cutoffs. Check false alerts and missed violations across policy types, language varieties and missing context. |
| Use four reviewer decisions | “Needs context” and “needs policy review” describe different next actions. Forcing either into a yes/no training label would teach the model an unsupported answer. | Have independent reviewers label a fresh sample and resolve disagreements with a policy specialist. Count escalations in workload. |
| Measure capture at a fixed budget | An alert helps only if a reviewer can reach it. I used roughly 10% of a batch as an experimental capacity assumption. Actual capacity depends on review time and severity. | Ask Operations for staffing and handling-time distributions. Compare violations caught per reviewer-hour and time to review serious harm. |
| Lock examples and predictions before revealing test results | Keeping creators, conversations and similar comments together reduces leakage. A rule designed after inspecting a mistake needs fresh evidence. | Compare queues on the same unseen items. Count violations gained and displaced. Inspect harmless comments that move ahead of a violation. |

## How I would apply this at TikTok

I would start with **one English-language comment-review queue** and a narrow question for the Content Ecosystem team. Can a reviewer reach actionable harassment sooner while preserving coverage of other serious harm? Earlier review could benefit the person targeted. An unnecessary flag costs reviewer time, and an incorrect enforcement decision can limit a creator’s reach and take time to appeal. Each has a different measurement.

1. **Validate the workflow.** With Operations, shadow reviewers across shifts and map the path from report to decision. With Policy and Trust & Safety, agree on the scope, escalation reasons and existing urgent routes. The deliverable is a reviewer-tested rubric and a measured capacity baseline.
2. **Set the offline release criteria.** With Data Science, collect a randomly sampled queue plus a separate challenge set. Double-label a subset, adjudicate disagreements and size each severity slice before testing. My provisional goals are 30% fewer false alerts at a validation target of 90% recall, and no observed loss of serious violations within the same review budget. Agree on uncertainty margins before opening the test.
3. **Run in shadow mode.** With Engineering, score the same incoming cases alongside the existing queue. Measure processing latency, cost, missing context and reviewer disagreement. Investigate any newly displaced serious violation. Advance only when Policy and Operations accept the safety and workload results.
4. **Pilot with a small reviewer team.** Compare matched shifts at equal staffing. Track violations caught per reviewer-hour, time to serious cases, reversals after appeal and reviewer overrides. Pause the pilot and restore the previous queue if the agreed serious-harm or creator-impact limits are crossed. Audit a sample below the cutoff for missed violations. If the pilot meets its safety and workload criteria across shifts, expand gradually and keep monitoring outcomes by policy version.
5. **Expand one capability at a time.** For a second language, use local policy experts, native-speaking annotators and a separate market evaluation. For visual review, start with one category and licensed image-caption pairs. A ResNet50 baseline could identify an object such as a knife. A reviewer would still need the surrounding activity to distinguish cooking from a threat. Compare text alone with text plus visual evidence before adding it to the queue.

## Explore the prototype

- Adjust the review budget and inspect which cases enter the queue. [Review queue](https://reviewlab-content-moderation.vercel.app/#queue)
- Compare the rules and text models at the same capacity. [Experiment lab](https://reviewlab-content-moderation.vercel.app/#experiments)
- Inspect the policy boundaries, reviewer actions and examples. [Policy guide](https://reviewlab-content-moderation.vercel.app/#policy)
- Compare comment-only and contextual inputs on the first development split. [Model choices](https://reviewlab-content-moderation.vercel.app/#models)

Sources include [Civil Comments](https://www.tensorflow.org/datasets/catalog/civil_comments), [TikTok’s Safety and Civility guidelines](https://www.tiktok.com/safety/en/policies-and-engagement/safety-civility), [Crawlora’s TikTok archive](https://huggingface.co/datasets/crawlora-net/tiktok-product-video-comments) and [Shofo’s TikTok archive](https://huggingface.co/datasets/Shofo/shofo-tiktok-general-small). ReviewLab is an independent prototype using public sources.

## Run and reproduce

Serve the dashboard locally with `python3 -m http.server 8765 --bind 127.0.0.1 --directory dist`. Open [the local dashboard](http://127.0.0.1:8765/). Run `npm test` to check the benchmark, policy workflow and model-comparison calculations. The [technical guide](TECHNICAL.md) covers model settings, experiments, data provenance and reproduction commands.
