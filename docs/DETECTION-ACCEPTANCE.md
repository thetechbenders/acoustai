# v0.1 extruder-skip detection acceptance contract

**Status:** proposed protocol for review, not experimental evidence or an approved numerical performance threshold.

**Tracking:** [Issue #1](https://github.com/thetechbenders/acoustai/issues/1). This M0 definition is a prerequisite for [normal-print collection (#3)](https://github.com/thetechbenders/acoustai/issues/3) and [verified-skip collection (#4)](https://github.com/thetechbenders/acoustai/issues/4).

## 1. Target and decision

The sole v0.1 positive target is a **verified extruder-feed failure episode** on a desktop FDM printer: loss of commanded filament advancement attributable to either (a) extruder motor missed steps or (b) drive-gear slip against filament. These mechanisms remain separate annotation attributes (**motor skip / gear slip / indeterminate**) and must have per-mechanism recall reported when sample counts permit. A click alone **does not** establish a positive. Under-extrusion without verified motor skip or drive-gear slip is **uncertain/excluded**, not an automatic negative. The experiment asks whether microphone data can distinguish independently verified skip events from normal print activity. The system may issue a local warning; it must never pause or control the printer, infer a root cause, or claim a safety function.

An acoustic detector's output is a **prediction**, never the ground-truth label. If no independent evidence of a skip is available, mark the interval *uncertain* rather than forcing a positive label.

## 2. Independent ground truth and annotation

Before collecting positives, identify the reference printer, operating procedure, and a safe means to witness faults. Controlled faults, if used, require an attended, non-destructive procedure; no unattended jams, heat/power excursions, or deliberate hazardous printer operation.

- **Confirmed positive:** a **non-acoustic physical criterion established before collection** demonstrates missed motor steps or gear-to-filament slip. Candidate evidence includes a visibly marked motor shaft/extruder gear or filament-motion reference filmed in sufficiently detailed silent/slow-motion video, or a validated independent feed-motion sensor; verify the method actually resolves the chosen printer's mechanism during the pilot. For a motor skip, a reproducible rotor discontinuity or backward motion relative to the commanded movement may qualify; for gear slip, measured drive motion without corresponding filament advancement may qualify. Ordinary video of an unmarked extruder, an audible click, or feed deficit alone cannot reliably identify the mechanism. Record timestamp/uncertainty, criterion, source and synchronization. Candidate positives must be sought using the **non-acoustic** channel, including potentially silent faults, not only sounds that attract attention.
- **Confirmed negative:** independently documented normal operation over a defined interval, with no positive indication from the predeclared non-acoustic criterion **and** a commanded-versus-observed filament-feed consistency check (e.g., marked filament length versus net commanded extrusion, accounting for retractions and measurement resolution). Define a meaningful tolerance before labeling. A failure to resolve the feed check makes that interval uncertain, rather than proof that no skip occurred. Neither silence nor absence of a detector alert establishes a negative.
- **Uncertain/excluded:** disputed events, under-extrusion without independently verified drive slip or missed steps, insufficiently resolved motion/feed evidence, missing ground truth, unreliable synchronization, microphone saturation, or unclassifiable overlaps. Retain metadata and reason; do not treat excluded time as scored negative exposure.
- **Annotation record:** session ID; printer and configuration ID; start/end timestamps on a stated clock; onset time or onset uncertainty interval; label; **mechanism** (motor skip / gear slip / indeterminate); evidence source and confidence; annotator; naturally occurring or induced and the inducement method; conditions; exclusion/adjudication reason, if any. Annotators must examine non-acoustic ground-truth material **with audio muted** and without detector outputs. Events first noticed by ear remain uncertain unless subsequently confirmed by the predefined physical criterion. Freeze all labels, exclusions and adjudications **before** viewing any final held-out detector outputs; retain the frozen annotation version and exclusion counts. A second blind review of ambiguous positives is preferred when feasible.

Store synchronization uncertainty explicitly, using a shared audio-visible synchronization marker at the start and end of a session where feasible to bound drift; the marker does not establish a fault. A claimed detection latency smaller than the ground-truth timing uncertainty must not be reported as meaningful precision.

For scoring, one **event is an episode**, which may contain multiple closely spaced mechanical skips. Merge independently confirmed constituent events separated by less than a predefined gap **G**; measure onset at the first confirmed constituent event. Choose and freeze G using pilot/development evidence before final scoring. Log constituent skips separately when resolvable. Fix the event-to-alert matching tolerance and allow at most one matched alert per episode.

## 3. Negative-class coverage

The normal-print baseline must deliberately include *at least*: idle/background, steady extrusion/infill, travel moves, retractions, fan-speed transitions, ordinary non-fault clicks, **at least two print speeds**, and when safe and applicable **non-extruder axis homing/stall-like sounds** and **filament loading/unloading**. Identify whether startup/homing sequences count toward scored exposure; unsafe or unavailable conditions must be documented, not deliberately induced. Record environmental interference and speed changes as conditions. Negative classes should include activity that superficially resembles skipping, not just quiet recordings.

Do not use intentionally induced skips as negative examples. When an independently observed skip occurs during a nominally normal recording, relabel the affected interval and assess whether the remaining session is eligible.

## 4. Collection and provenance

Capture short consented sessions on a single documented reference printer first; expand coverage only after the pilot protocol is repeatable. For each session, record:

- Session ID, date/time and clock reference; operator and consent status; recording provenance and content hash.
- Printer model/firmware, extruder configuration/gearing, filament/material, relevant G-code/print phase and print speed; stepper driver mode, motor current, microstepping, and pressure/linear-advance setting when accessible.
- Microphone model, location/orientation/distance, mounting, enclosure state, sample rate, bit depth, channel count, gain, clipping/dropout observations.
- Fans and other acoustic conditions, recording/ground-truth synchronization method, annotation version, commanded-versus-measured filament-feed check/tolerance, **fault inducement method or natural occurrence**, operator attendance, and changes from the prior session.

Fault inducement must be attended, reversible and demonstrably within ordinary safe printer/material operating limits. Do not authorize specific motor-current or temperature manipulation solely through this protocol: document any proposed method and its bounds for review before use. Report naturally occurring and induced positives separately, as induction may change their sound.

Raw audio and incidental speech remain outside Git. Define restricted local storage, access, retention/deletion terms and consent before capturing; publish only sanitized metadata and aggregate metrics where permitted. Do not claim anonymous collection if background speech or machine/operator identifiers may be recorded.

## 5. Leakage-resistant evaluation

Assign entire **recording sessions** to **development/training**, **development validation**, or a **sealed final holdout** before windowing, augmentation, spectrogram generation, or feature extraction. All clips, labels and transforms derived from a session stay in one partition. Record partition membership and the split seed/procedure.

Reserve complete, independently annotated sessions for a **sealed final comparison**; choose the fraction after inspecting pilot session counts rather than creating a tiny or duplicated holdout. Use separate development-validation sessions for M2 tuning and error analysis and subsequent M3 model design. **Do not inspect final-holdout detector outputs, per-event failures, summary metrics, or other outcome-derived information during M2/M3 development.** Freeze both detectors, all alert rules, labels, adjudications and scoring settings first, then evaluate **both once** on that same sealed holdout. If a final result prompts redesign, collect a fresh independent holdout for any later comparative claim. Group repeated contiguous runs and shared conditions to limit near-duplicate leakage; disclose printer-specific limits. Never count overlapping windows as independent holdout examples.

## 6. Metrics and counting rules

Use one documented evaluation configuration for both conventional DSP and TinyML: **same sealed final-holdout sessions**, microphone signal, sample rate, available audio context, ground truth, episode definition/gap G, event-matching tolerance, and common alert debounce/refractory logic (or justified differences). Give both approaches comparable tuning effort using **development data only**. Record computational resources and end-to-end alert latency on the actual target when available.

- **Episode recall:** matched confirmed positive episodes / all evaluable confirmed positive episodes. Each episode matches at most one alert; report by mechanism as well as pooled when counts permit.
- **Alert precision:** alerts uniquely matched to confirmed positive episodes / all alerts in evaluable intervals. Duplicate alerts on one episode are unmatched unless removed by a predeclared refractory rule.
- **False alerts per hour:** unmatched alerts during confirmed negative exposure / confirmed negative hours. Report numerators and actual duration **separately for active printing and idle**, alongside an overall rate. Define whether homing, print start, loading and unloading are in the denominator before scoring. Do not dilute the main active-print rate with extended quiet idle recordings.
- **Detection latency:** time from independently marked **episode** onset to the first correctly matched alert. Report a distribution and synchronization uncertainty, not only one average. Include missed events in recall, not as arbitrary numeric latency.
- **Operating point:** false-alerts-per-hour target, per-detector threshold, episode gap G, event-match tolerance, debounce/refractory duration, and minimum scored negative exposure are **TBD until pilot recordings exist**. Specify, justify, and freeze them on development data *before* scoring held-out sessions.

Report raw counts and binomial confidence intervals for recall and precision, denominators and exposure for false-alert rates, microphone/session conditions, frozen exclusions, failure cases, and whether metrics reflect host playback, embedded inference, or actual hardware. Interpret sparse denominators explicitly. Never conflate window-level accuracy with event detection.

## 7. Baseline comparison and go/no-go

M2 first develops a conventional signal-processing comparator (e.g., energy/transient or spectral features) without learned weights, using development and validation sessions only. M3 develops TinyML using the **same development/validation partition and matched audio information**. After both approaches and alert policies are frozen, compare them once on the **same sealed final-holdout sessions**. Report recall versus false-alerts-per-hour curves as well as recall at a **common, predeclared false-alert-rate operating point**, paired episode outcomes, latency and resource costs. Do not cherry-pick independently favorable thresholds. M2 development-validation findings are not final-holdout results.

**Stop/revise the target** if independently verified positives cannot be collected safely, timestamped reliably, or distinguished from normal transients with reproducible evidence. Improve collection/annotation rather than training on questionable labels. If learned inference fails to demonstrate a meaningful and defensible benefit over the non-ML comparator under matched conditions, report that limitation and do not represent AI as required.

No exact precision, recall, false-alarm, or latency targets are asserted here. A measurable acceptance threshold must be selected from pilot feasibility data, reviewed, and fixed before final evaluation. This protocol defines what must be measured, not a claim that AcoustAI already works.

## 8. Exit checklist for Issue #1

- [ ] Confirm the operational positive definition and independent evidence method on the chosen printer.
- [ ] Agree on negative classes and uncertain/excluded annotation rules.
- [ ] Approve session metadata, consent/privacy, and synchronization process before capture.
- [ ] Review episode gap G, event matching, denominators, three-way session separation, sealed holdout and metrics.
- [ ] Approve pilot-derived operating-point selection procedure, acceptable attended fault-inducement methods and stop gates.

**Scope boundary:** this document does not authorize recording, hardware modification, model training, firmware development, or M1/M2 work. Completing the protocol is distinct from collecting or validating evidence.
