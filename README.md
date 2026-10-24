# SUOU Video Recorder v4.4.6.8

**Windows 11 · Google Chrome / Microsoft Edge · Cloudflare Pages · AutoEpisode companion extension**

SUOU Video Recorder is a browser-based tab recorder designed to work with the **SUOU Desktop AutoEpisode v4.4.6.8** extension. It supports recording a sequence of episodes either as separate files or as one continuous WebM file, with captured tab audio and Audio Guard checks.

> **Release status:** v4.4.6.8 contains fixes intended to stop recording and finalize the file at the configured last episode. The final-episode stop and long-session save behavior still require verification in your own Windows browser setup. Do a two-episode test before a long recording.

## What's new in v4.4.6.8

- Adds a final-episode stop delivery/acknowledgment path between AutoEpisode and Recorder.
- Attempts to retry delivery when the final stop notification is not acknowledged (up to three attempts in the extension).
- Adds diagnostic stages for last-episode detection, stop-message delivery, receipt, and final-file completion.
- Retains the **single-file season recording** option introduced in v4.4.6.7.
- Retains per-episode recording mode and Audio Guard functionality.

A successful stop-message acknowledgment is **not** proof that a playable file was saved. Verify `FINAL_SAVE_COMPLETED` and inspect the output file itself.

## Packages

| Component | Archive | Purpose |
|---|---|---|
| Recorder website | `SUOU-Video-Recorder-v4.4.6.8-Final-Save-Fix-Pages.zip` | Cloudflare Pages website (`index.html`) |
| AutoEpisode extension | `SUOU-Desktop-AutoEpisode-v4.4.6.8-Final-Stop-Fix.zip` | Chrome/Edge Manifest V3 extension; source-tab selection, episode monitoring, Recorder communication |

**Use matching v4.4.6.8 versions of both components.** Do not leave an older AutoEpisode version enabled alongside the newer one.

## Requirements

- Windows 11 with recent desktop **Chrome or Edge**.
- A Cloudflare Pages deployment for the Recorder, such as `https://recoder.sudoz.workers.dev/`.
- The unpacked AutoEpisode extension installed locally in the same browser.
- A supported video-page source and permission to record the content.
- Adequate free disk space for long recordings.

The Recorder is a webpage; the AutoEpisode component is a desktop-browser extension. The extension package is **not** deployed to Cloudflare Pages.

## 1. Install the AutoEpisode extension (Chrome / Edge)

1. Extract `SUOU-Desktop-AutoEpisode-v4.4.6.8-Final-Stop-Fix.zip` into a permanent folder on your PC.
2. Open `chrome://extensions/` (Chrome) or `edge://extensions/` (Edge).
3. Turn on **Developer mode**.
4. Disable or remove earlier SUOU AutoEpisode extensions to prevent competing controls.
5. Click **Load unpacked** and select the folder containing `manifest.json` (inside `SUOU-Desktop-AutoEpisode-v4.4.6.8`).
6. Confirm that the extension appears and shows **v4.4.6.8**.
7. Keep the extracted folder in place; Chrome/Edge loads the extension from it.

The included manifest permits the configured Recorder origin `https://recoder.sudoz.workers.dev/*` and supported source-site patterns. If your Recorder uses a different custom domain, its origin may need to be added to the extension's `manifest.json` content-script matches and host permissions.

## 2. Deploy the Recorder to Cloudflare Pages

1. Open the Cloudflare dashboard and select the existing Pages project for SUOU Recorder.
2. Deploy the **contents** of `SUOU-Video-Recorder-v4.4.6.8-Final-Save-Fix-Pages.zip` as a new static-site deployment. `index.html` must be at the site's root.
3. Open the deployed Recorder URL.
4. Hard-refresh using **Ctrl + Shift + R** and verify that the UI identifies itself as **v4.4.6.8**.
5. Refresh the source-video tab as well so the updated extension content script loads.

If using the `recoder.sudoz.workers.dev` custom domain, make sure it is connected to the Pages project you updated. A successful deployment to a different Pages project will not update that custom domain.

## 3. Set up the video source and recording

1. Open the actual video **playback page** in a Chrome/Edge tab (not just the site homepage or the Recorder tab).
2. While that playback tab is active, open the SUOU extension popup and choose **Set current tab as source** (or its equivalent Chinese-language button).
3. Open the on-page SUOU control panel. Set the **current episode** and the **last episode** explicitly; **do not leave the last episode at `0`**. A scanned list of episode buttons does not prove that every listed episode is available.
4. Open the Recorder in another tab. Choose either **single-file recording** or **per-episode recording**.
5. Start recording. In the browser's screen-sharing picker, choose the **video playback tab** and enable **Share tab audio**.
6. Confirm the Recorder and extension both report an active session, then let the video play.

### Recording mode comparison

| Mode | Behavior | Best for |
|---|---|---|
| **Single-file recording** | Continues recording across episode changes; finalizes one file when the last episode ends | One continuous season video |
| **Per-episode recording** | Rotates/saves the recording as episodes change | A separate video file for each episode |

**Single-file mode:** Enable the checkbox for recording from EP1 through the final episode as one file. For lengthy sessions, also enable **direct streaming to a Windows file**. The browser asks you to choose a filename at the start; the file is finalized only when recording finishes. This uses the browser's file-save capability (`showSaveFilePicker`), which may not be available in all environments.

If direct disk streaming is disabled, the Recorder can accumulate data for a download at the end, which may consume substantial memory and fail on very large recordings. Do not close or reload the Recorder tab during recording. Episode-loading gaps will remain in the continuous video.

## 4. Verify audio (Audio Guard)

- Share the **browser tab**, rather than defaulting to full-screen capture.
- Ensure **Share tab audio** is enabled, the site player is unmuted, and the browser tab is not muted.
- Use the short trial recording/playback feature before a long session.
- Watch the live audio-level indicator; the presence of an audio track alone does not guarantee audible output.
- Play back the saved file to verify that audio and video are both present and synchronized.

## 5. Final-episode stop and save

The intended flow is:

1. The extension recognizes that the configured last episode has ended.
2. It sends a final-session stop notification to the Recorder.
3. The Recorder stops the active `MediaRecorder` session and completes the output process.
4. For direct-to-disk mode, the file writer must close successfully; for download mode, the browser must actually save the file.
5. The result is verified by opening the file from Windows File Explorer.

If the final episode repeats or the file does not finalize, **stop recording manually to protect the captured footage**, then export both diagnostics and check the output on disk. Do not leave a failed long recording unattended.

## 6. Diagnostic files

When troubleshooting, export the relevant diagnostics from the Recorder and extension (where the buttons are present):

- `SUOU-Recorder-ARM-Diagnostic.json` — Recorder-side session and message events.
- `SUOU-Source-Binding-Diagnostic.json` — extension source binding, session status, and ARM trace.
- Video-page episode diagnostics, if available — detected episode buttons, video state, and page-level events.

Useful stages include:

| Stage | Meaning |
|---|---|
| `ARM_SENT` / `BRIDGE_RECEIVED` | Recorder startup message sent / observed by bridge |
| `ARM_RECEIVED` | Extension background worker received startup |
| `PLAYER_START_SENT` / `PLAYER_ARMED` | Playback-page monitor start requested / acknowledged |
| `LAST_EPISODE_DETECTED` | Extension identified the configured last episode end |
| `END_SESSION_DELIVERY` | Final-stop notification delivery attempt/result |
| `END_SESSION_RECEIVED` | Recorder received the final-stop event |
| `FINAL_SAVE_COMPLETED` | Recorder reported completion of its final-save path |

The **same request/session identifiers** should be used to correlate messages where provided. Logs may be incomplete if a tab or browser session was closed. Even `FINAL_SAVE_COMPLETED` is not a substitute for checking that the file exists and plays correctly.

## 7. Troubleshooting

| Symptom | What to check |
|---|---|
| Recorder says source tab is not selected | Open the actual playback tab and bind it using the extension popup; do not bind the Recorder tab |
| `active=false` / `playerActive=false` | Ensure recording was started, both components are v4.4.6.8, and the source tab has been refreshed; export both diagnostic JSON files |
| EP1 keeps repeating | Confirm that the next episode truly loads and that the extension can detect the selected episode; check page diagnostics |
| Last episode loops rather than stopping | Confirm a nonzero, correct last-episode setting; inspect final-stop delivery and receipt logs; manually stop to preserve data |
| Video saved without sound | Re-select the source browser tab with **Share tab audio** enabled; test a short recording first |
| Recorder page still shows an older version | Confirm the correct Pages project/custom domain was deployed, then hard-refresh |
| A file-save dialog appears | The browser controls file selection and download confirmations; disable “Ask where to save each file” for normal downloads if appropriate |
| Single-file recording consumes too much memory | Enable direct-to-disk streaming in a supported desktop Chrome/Edge build |
| 404 favicon, CSS, or site image warnings | These are usually unrelated to source binding or episode transition; prioritize explicit SUOU errors and logs |

## 8. Recommended validation checklist

- [ ] Both Recorder and extension show **v4.4.6.8**.
- [ ] The actual playback tab is selected as source.
- [ ] The last episode is a specific valid number, not `0`.
- [ ] Audio Guard indicates signal and a short test file has audible sound.
- [ ] Episode 1 changes to episode 2 automatically.
- [ ] When episode 2 is configured as the test's last episode, recording **stops instead of looping**.
- [ ] The output file exists, has a plausible size, and plays from beginning to end with sound.
- [ ] Only after this test should a longer multi-episode recording be attempted.

## Known limitations and safety notes

- Browser and source-site updates can change episode selectors, iframe behavior, audio capture, or download permissions.
- Browser recording may be restricted for protected media. This software does not bypass copy protection or site access controls.
- Continuous recording captures any buffering, loading, or transition screens between episodes.
- Large recordings require sufficient disk space, stable browser operation, and a supported file-write path.
- Recording and playback must comply with applicable law, copyright, and website terms.

---

**SUOU Video Recorder v4.4.6.8** · English documentation · Recorder + Desktop AutoEpisode extension
