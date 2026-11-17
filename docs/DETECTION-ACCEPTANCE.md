# v0.1 extruder-skip detection acceptance contract

**Status:** proposed protocol for review, not experimental evidence or an approved numerical performance threshold.  
**Tracking:** [Issue #1](https://github.com/thetechbenders/acoustai/issues/1). This M0 definition is a prerequisite for [normal-print collection (#3)](https://github.com/thetechbenders/acoustai/issues/3) and [verified-skip collection (#4)](https://github.com/thetechbenders/acoustai/issues/4).

## 1. Target and decision

The sole v0.1 positive target is an **extruder drive skipping event** on a desktop FDM printer: observed loss of commanded filament advancement caused by the drive mechanism slipping or stepping unsuccessfully. An audible click alone **does not** establish this event. The experiment asks whether microphone data can distinguish independently verified skip events from normal print activity. The system may issue a local warning; it must never pause or control the printer, infer a root cause, or claim a safety function.

An acoustic detector's output is a **prediction**, never the ground-truth label. If no independent evidence of a skip is available, mark the interval *uncertain* rather than forcing a positive label.

## 2. Independent ground truth and annotation

Before collecting positives, identify the reference printer, operating procedure, and a safe means to witness faults. Controlled faults, if used, require an attended, non-destructive procedure; no unattended jams, heat/power excursions, or deliberate hazardous printer operation.

- **Confirmed positive:** contemporaneous independent observation (e.g., video of the extruder/filament or another independently validated sensor) supports an actual missed/slipped feed event, with a timestamp or bounded event interval. Record the observer/evidence type and how it was synchronized to audio. An acoustic click or model/DSP prediction alone is insufficient.
- **Confirmed negative:** independently documented normal operation, including expected mechanical activity, without observed skip events during the labeled interval. Mere absence of a detector alert does not establish normal operation.
- **Uncertain/excluded:** disputed events, missing ground truth, unreliable synchronization, microphone saturation, or unclassifiable overlapping events. Retain metadata and reason, but exclude from scored positive/negative evaluation until adjudicated.
- **Annotation record:** session ID; printer and configuration ID; start/end timestamps on a stated clock; onset time or onset uncertainty interval; label and confidence/evidence provenance; annotator; conditions; exclusion reason (if any). Annotate event boundaries without consulting detector scores. A second independent review of ambiguous positive events is preferred where feasible.

Store synchronization uncertainty explicitly. A claimed detection latency smaller than the ground-truth timing uncertainty must not be reported as meaningful precision.

## 3. Negative-class coverage

The normal-print baseline must deliberately include *at least*: idle/background, steady extrusion/infill, travel moves, retractions, fan-speed transitions, and ordinary non-fault clicks. Record print-speed changes and environmental interference as conditions. Negative classes should include activity that superficially resembles skipping, not just quiet recordings.

Do not use intentionally induced skips as negative examples. When an independently observed skip occurs during a nominally normal recording, relabel the affected interval and assess whether the remaining session is eligible.

## 4. Collection and provenance

Capture short consented sessions on a single documented reference printer first; expand coverage only after the pilot protocol is repeatable. For each session, record:

- Session ID, date/time and clock reference; operator and consent status; recording provenance and content hash.
- Printer model/firmware, extruder configuration, filament/material, relevant G-code/print phase and print speed.
- Microphone model, location/orientation/distance, mounting, enclosure state, sample rate, bit depth, channel count, gain, clipping/dropout observations.
- Fans and other acoustic conditions, recording/ground-truth synchronization method, annotation version, and changes from the prior session.

Raw audio and incidental speech remain outside Git. Define restricted local storage, access, retention/deletion terms and consent before capturing; publish only sanitized metadata and aggregate metrics where permitted. Do not claim anonymous collection if background speech or machine/operator identifiers may be recorded.

## 5. Leakage-resistant evaluation

Assign entire **recording sessions** to development or held-out evaluation **before** windowing, augmentation, spectrogram generation, or feature extraction. All clips, labels and transforms derived from a session stay in one partition. Record partition membership and the split seed/procedure.

For the initial experiment, reserve some complete, independently annotated sessions for final assessment; choose the exact fraction after inspecting pilot session counts, not by creating tiny or duplicated holdouts. Where possible, group repeated contiguous runs and shared recording conditions to prevent near-duplicate leakage; disclose the extent to which the evaluation is printer-specific. Never report overlapping windows from the same session as independent holdout examples. Avoid tuning thresholds or feature parameters using held-out outcomes.

## 6. Metrics and counting rules

Use one documented evaluation configuration for both the conventional DSP baseline and any TinyML model: same held-out sessions, ground truth, event matching tolerance, alert debounce/refractory logic (or explicitly controlled differences), and resource/latency measurement method.

- **Event recall:** matched confirmed skip events / all evaluable confirmed skip events. An event can match at most one predicted alert.
- **Event precision:** matched true alerts / all evaluated alerts in evaluable intervals. Count duplicate alerts for the same event as unmatched unless a prespecified refractory rule suppresses them.
- **False alerts per hour:** unmatched alerts in confirmed negative exposure / total confirmed negative recording hours. Report the actual negative exposure duration and event counts; do not extrapolate from a few quiet minutes.
- **Detection latency:** time from independently marked skip onset to the first correctly matched alert. Report a distribution and synchronization uncertainty, not only one average. Include missed events in recall, not as arbitrary numeric latency.
- **Operating point:** detection threshold, event-match tolerance, debounce/refractory duration, and minimum negative exposure are **TBD until pilot recordings exist**. Specify, justify, and freeze them on development data *before* scoring held-out sessions.

Report denominators, confidence/uncertainty where feasible, microphone and session conditions, exclusion counts, failure cases, and whether metrics reflect host playback, embedded inference, or actual hardware. Never conflate window-level accuracy with event detection.

## 7. Baseline comparison and go/no-go

M2 first implements and evaluates a conventional signal-processing comparator (e.g., energy/transient or spectral features) without learned weights. M3 TinyML must be compared **on exactly the same held-out sessions**, with comparable alerting logic and disclosed resource requirements, before claiming incremental value.

**Stop/revise the target** if independently verified positives cannot be collected safely, timestamped reliably, or distinguished from normal transients with reproducible evidence. Improve collection/annotation rather than training on questionable labels. If learned inference fails to demonstrate a meaningful and defensible benefit over the non-ML comparator under matched conditions, report that limitation and do not represent AI as required.

No exact precision, recall, false-alarm, or latency targets are asserted here. A measurable acceptance threshold must be selected from pilot feasibility data, reviewed, and fixed before final evaluation. This protocol defines what must be measured, not a claim that AcoustAI already works.

## 8. Exit checklist for Issue #1

- [ ] Confirm the operational positive definition and independent evidence method on the chosen printer.
- [ ] Agree on negative classes and uncertain/excluded annotation rules.
- [ ] Approve session metadata, consent/privacy, and synchronization process before capture.
- [ ] Review event matching, denominators, session-level separation and metrics.
- [ ] Approve pilot-derived operating-point selection procedure and stop gates.

**Scope boundary:** this document does not authorize recording, hardware modification, model training, firmware development, or M1/M2 work. Completing the protocol is distinct from collecting or validating evidence.
