# Blind Merge — V2V Latency Pilot

A browser-based 3D roundabout simulator for a 20-trial human pilot investigating decisions under delayed vehicle-to-vehicle (V2V) position messages.

## Run the simulator

1. On this repository page, click **Code → Download ZIP**.
2. Extract the downloaded ZIP.
3. Open **index.html** in a modern desktop browser such as Chrome, Edge, Firefox, or Safari.
4. Click **Start Trial**.

No installation, npm, build tools, or local server are required. Internet access is needed to load Three.js from its CDN. The browser must support WebGL.

You can also clone the repository and open the file:

```sh
git clone https://github.com/amln19/fri-sim.git
cd fri-sim
open index.html
```

The `open` command above is for macOS. On Windows or Linux, open `index.html` from your file manager or browser.

**Viewing index.html on github.com shows its source code; it does not launch the simulator.** Download and open the file locally. GitHub Pages hosting is not currently configured.

## Project files

- **index.html** — the entire application: HTML interface, CSS, Three.js scene, driving physics, delayed radar, experiment management, recording, and CSV export.
- **README.md** — these instructions.

The application was intentionally built as a single self-contained HTML file. It generates its scene assets in code and loads only the Three.js library externally. There are no missing source folders or asset packages.

## Controls and experiment

- **W / Up Arrow:** accelerate.
- **S / Down Arrow:** brake.
- **Esc:** pause. Steering follows a fixed route automatically.
- Complete 20 trials: five each at 0, 100, 300, and 600 ms latency, in shuffled order. The latency is revealed after the trial.
- Each trial ends on a successful clearance, collision, or 12-second timeout.
- Score: +100 for passing, −500 for a collision, and −10 per second nearly stopped at the yield line. Timeouts receive no completion bonus.
- **Save & Next Trial** saves the result. **Discard & Redo Trial** repeats the same encounter without saving the discarded attempt.

The current pilot presents delayed positions without conformal prediction assistance. It is a preliminary human experiment before the broader research implementation.

## Results and export

Saved results remain in the current browser's local storage. Click **Export Data to CSV** to download:

```text
trial_id,trial_number,latency_ms,collision_occurred,min_distance_m,wait_time_s,total_time_s,score
```

Minimum distance is vehicle-body clearance, rather than the distance between vehicle centers. Yield wait measures time nearly stopped near the yield line. Paused time is excluded from trial time.

Data does not upload to GitHub or a research server. Each participant must export and submit their CSV separately. Export before clearing browser storage or moving between browsers or copies of the simulator. Historical sessions in the same browser are included in the export; current setup trial IDs begin with `p3-`.

## Record a presentation trial

In a supported desktop browser:

1. Click **Record Trial** and select the simulator tab in the browser's sharing picker.
2. Click **Start Trial** and drive.
3. Recording stops automatically three seconds after the result appears.
4. Preview and download the recording. MP4 is preferred when supported; otherwise the simulator uses WebM.

The recording includes the cockpit, radar, timer, and result card. It does not capture microphone audio. Videos remain in memory until downloaded; download them before reloading or closing the page. Recording availability depends on the browser's screen-capture support.
