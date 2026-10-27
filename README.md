# Roundabout Lab

A buildless research showcase for human decisions and automated roundabout coordination under stale V2V messages. The complete app, including the deterministic simulation and calibration engine, is in **index.html**.

## Run

Download this repository as a ZIP, extract it, and open **index.html** in a modern desktop browser. No npm, build tools, or server are needed. Internet access loads Three.js 0.128.0 from jsDelivr, with an unpkg fallback; WebGL is required. GitHub's source-code viewer does not run HTML.

```sh
git clone https://github.com/amln19/fri-sim.git
cd fri-sim
open index.html
```

`open` is the macOS command; on Windows/Linux, open the file from your browser or file manager. Desktop Chrome is the verified browser. Responsive layouts and touch pedals are included; tab recording depends on browser support.

## Suggested professor demonstration

1. **Agent lab:** run the fixed-margin preview at 300 ms. Select a vehicle to inspect generation time, receipt time, reported speed, and current message age. Compare actual, received, and predicted-current position markers.
2. **Occupancy timeline:** inspect predicted entry/exit intervals, the uncertainty buffers in seconds, the proposed release sequence, and observed occupancy. Pause, step, change playback speed, or select a cockpit view.
3. **Human pilot:** use **Practice · observer**, then a restricted scored trial. W/Up accelerates; S/Down brakes; Esc pauses/resumes. Steering follows the route automatically. Scored trials reveal only received traffic state; practice also reveals ground truth.
4. **Compare policies:** click **Develop & calibrate margins**, then **Run matched comparison**. Default development uses 12 seeds at each primary delay; calibration uses 360 independent encounters; testing uses 30 held-out seeds × 2 delays × 4 policies = 240 runs.
5. Inspect waiting, completion, collision/near-conflict counts, uncertainty intervals, coverage by age, and a replay. Replay is recomputed from the frozen configuration/model and checked against its recorded result.
6. **Method & hardware:** inspect assumptions, metric definitions, exploration settings, and the software interface for a future physical-car adapter.

Calibration and test results are generated locally when you run them. There are no prefilled human participants or fabricated study findings. The initial fixed-margin preview uses 0.75 s and is explicitly unqualified until development completes.

## What is implemented

- Four fixed routes around a single-lane, four-arm roundabout; one conservative shared conflict resource.
- Full-scale SI units: 13 m lane center radius; 4.4 × 1.85 m footprints; 2.4 m/s² acceleration; 4.5 m/s² agent braking; nominal 9 m/s agent cruise.
- Deterministic 60 Hz physics, 10 Hz timestamped V2V packets, and 5 Hz coordination.
- Generation/delivery queue, sequence-based out-of-order rejection, message age, optional live packet loss and delay jitter.
- Seeded post-message motion disturbances, indexed by seed, vehicle, and simulation-time epoch, matched across policies.
- A common route-relative predictor using only delivered packets and issued/candidate release commands.
- Four policies: no added uncertainty margin (common headway retained), development-tuned fixed margin, pooled split-conformal margin, and age-conditioned split-conformal margins.
- Enumeration of 24 entry orders. For each order, greedily select the earliest feasible releases on a common 0.5 s grid over 45 s; minimize predicted total waiting among those candidates. This is not a global continuous-time optimum. Respect committed releases and hold when unsupported/infeasible.
- An identical optional local-range emergency brake for all policies, using clearance/closing motion within 1.2 m, with interventions counted. It is explicitly separate from normal V2V-only planning.
- Oriented-footprint collision detection, exact sampled body clearance, encounter-level near-conflicts, per-car/total/maximum waiting, completion and timeout outcomes.
- Observer/practice versus restricted participant views; primary and extended 20-trial human protocols, paired scenarios, rotated condition order, retained exclusions, CSV/trace exports, and browser-local persistence.
- Native tab recording with MP4 preferred and WebM fallback, result-card tail capture, clip preview, and downloads. No microphone capture.

## Development, calibration, and test separation

Reserved seed ranges are disjoint:

- Development: starts at **10000**. Fixed q grid: 0, 0.25, 0.5, 0.75, 1, 1.5, 2, 3, 4 seconds. Choose the smallest tested value with zero development collisions, near-conflict rate ≤ 10%, and full completion. If no value qualifies, retain that failure visibly.
- Calibration: starts at **20000**, up to 3000 independent encounters. One target and snapshot per encounter, under recorded release commands; snapshots span held and moving pre-entry states. Joint endpoint score = max absolute entry/exit prediction error.
- Testing: starts at **30000** or a user-selected higher seed. Same seeds and external perturbation streams for all policies. Primary test delays are 0 and 300 ms; no packet loss/jitter.

Split-conformal rank is ceil((n + 1)(1 − alpha)); unsupported ranks imply an infinite margin. Fixed age bins are 0–200, 200–500, and 500–1200 ms. At least 30 calibration encounters are required per supported group; nonempty sparse adjacent groups merge before evaluation. Empty, uncalibrated groups remain unsupported and withhold new releases. Choose extended calibration to support 600 ms live exploration.

Dynamics, alpha, emergency behavior, and near-conflict threshold are included in the calibration signature. Changing them requires new calibration for new runs. An existing comparison and replay retain their frozen configuration and model.

### Interpretation limits

This is a research prototype, not a validated safety controller. Conformal marginal coverage depends on exchangeability; the closed-loop controller changes state/command distributions. The app reports measured joint endpoint coverage and does not claim a formal collision-avoidance guarantee.

The conservative shared-circle resource can allow the zero added margin to qualify on development data. In that case age-conditioned margins may add waiting without improving observed safety. The app explicitly reports this result rather than manufacturing a benefit. A successful software demonstration does not by itself establish the paper's proposed performance improvement.

## Metric definitions

- **Approach waiting:** time from the upstream waiting-zone boundary, s = 0, to first conflict entry, including slow travel. Initial states are already inside the zone at t = 0. Cars not entering before termination have censored observed waits.
- **Stopped yield wait:** the human-pilot incentive metric, separate from approach waiting. Counts speed ≤ 1 km/h with the front bumper within 3 m before or 1 m after the yield line.
- **Collision:** overlapping oriented vehicle footprints.
- **Near-conflict:** positive sampled footprint clearance below the frozen threshold. An encounter can have a near-conflict and subsequently collide; both flags are retained.
- **Minimum distance:** minimum footprint-to-footprint clearance over sampled times and vehicle pairs; contact gives zero.
- **Completion:** all four automated cars reach their route ends, or the human completes the northbound route. Agent timeout is 60 s; human timeout is 12 s.
- **Waiting comparison:** only seeds completed by all four policies at each delay; counts are shown. Failed runs remain in event/completion statistics.
- **Event uncertainty:** Wilson 95% intervals by delay/policy. Waiting differences use 1000 bootstrap resamples of whole seed clusters, keeping the two delays together.
- **Coverage audit:** one target selected by seed modulo four, probed 0.8 s after its committed release. Both realized entry/exit errors must fall within q. Missing endpoints/probes are reported. This closed-loop audit is not a formal repeated-decision safety certificate.

## Human trials, data, and original pilot

The primary protocol uses 10 matched scenarios × two delays (0/300 ms). Extended mode uses five scenarios × four delays (0/100/300/600 ms). Matched scenario blocks are shuffled, with condition order rotating by block and participant numeric suffix. Use a pseudonymous participant ID; assignment and cohort balance remain the experimenter's responsibility.

Every finished attempt is retained. Save includes it in the scored sequence; exclude records a reason and repeats the same encounter. Practice runs, interruptions, timeouts, and collisions remain distinguishable. The browser saves pending results across reloads and records an active attempt as interrupted when normal page-unload handling runs. Browser crashes cannot guarantee a final unload save.

Pilot CSVs include the original eight metrics plus participant/session/seed identifiers, outcome, approach waiting, censoring, inclusion/exclusion, timestamps, and interruption counts. Companion JSON includes message, input, and trajectory logs.

The original **blind-merge-pilot-v1** localStorage entry is read only. If present on the same browser origin, **Export original pilot data** downloads its JSON and CSV without altering it. New data uses **roundabout-research-v4**. Moving between file copies, hosted origins, or browsers can change local-storage availability. Export before moving or clearing storage.

Full research audit JSON includes the frozen model, calibration probes, development grid, configuration, held-out result rows, and analysis definitions. Encounter JSON contains full trajectory/message/command/decision traces. No results upload automatically to GitHub or a research server.

## Recording

Click **Record tab**, choose this simulator tab in the browser sharing picker, then start a trial/encounter. Recording stops three seconds after a result appears, or when you click Stop recording. Open **Clips** to preview/download each recording. Video includes the interface and has no microphone audio. Clips are memory-only until downloaded; download before reloading/closing.

## Physical-car interface

`window.RoundaboutLab` exposes the deterministic engine, a received-message coordinator factory, configuration, current delivered packets, issued commands, and audit exports.

```js
const coordinator = RoundaboutLab.createCoordinator(config, policy, calibratedModel);
coordinator.receive({vehicle, sequence, generated, s, v, route}, receiptClockSeconds);
const {commands, plan} = coordinator.tick(monotonicClockSeconds);
```

Packets use vehicle IDs 0–3 and simulation-relative seconds/meters. `receive` validates fields and rejects older sequences; `tick` plans at no more than 5 Hz. The factory accepts messages and produces release decisions without access to simulator ground truth. Release commands include the triggering packet, message age, and occupancy interval.

**There is no physical transport or hardware actuation in this app.** A 1/10-scale deployment needs route/localization mapping, clock synchronization, a transport adapter, validated units/dynamics, and separate calibration/testing. Do not apply full-scale simulation commands directly to physical vehicles.

## Developer verification

The shipped app has no runtime build dependencies. Optional deterministic engine checks use Node's standard library only:

```sh
node tests/verify.cjs
```

Browser checks cover live playback, calibration/comparison, deterministic replay, human result persistence, responsive layouts, exports, and native recording. Test-generated data is separate from participant data.
