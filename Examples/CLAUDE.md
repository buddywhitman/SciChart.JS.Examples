# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This directory (`Examples/`) is the source for the SciChart.js demo gallery app (demo.scichart.com) — a React/webpack site that hosts many small chart examples plus a few "Featured Apps". The current branch's active work lives in one featured app: a Vital Signs Monitor (ECG/heart-rate/GSR biofeedback dashboard) with an ESP32 hardware front-end and Firebase logging.

Sibling directories one level up (`../ExampleDependencies`, `../Tools`) belong to the wider `SciChart.JS.Examples` repo, not to this app.

## Commands

Run from `Examples/`:

- `npm run dev` — start the dev server (webpack-dev-server, client-only config) at https://localhost:8080
- `npm run dev:clean` — same, via `start-dev.ps1` (PowerShell, cleans state first)
- `npm run build` — full production build: vanilla examples compile → images → client bundle (output to `./build`)
- `npm start` — after `build`, serves client+server in prod mode at https://localhost:3000/demo
- `npm run buildServer` / `npm run startServer` — build/run the Express server config separately
- `npm run compileVanillaExamples` — `tsc --build tsconfig.vanilla.json` (compiles the plain-JS/vanilla example variants used for CodeSandbox/StackBlitz embeds)
- `npm run prettier` — format `src/**/*.{js,ts,tsx}`
- `npm run testUnit` — `mocha` over `testUnit/**/*.test.ts` (no `testUnit/` directory currently exists in this branch — the script is a no-op until tests are added)
- `npm run generateAll` / `generateSitemap` / `generateFooterLinks` — regenerate sitemap.xml and footer links from `examples-metadata.json`

There is no `npm run lint` script; `tslint.json` exists but tslint is deprecated/unused in practice — use `npm run prettier` for formatting.

## Architecture

### Demo gallery shell
- `src/index.tsx` — app entry
- `src/components/App.tsx`, `AppRouter/`, `AppDetailsRouters/` — top-level routing between the gallery and individual example detail pages
- `src/components/Examples/ExamplesRoot.tsx` + `ExampleStrings.ts` — registry/routing glue that maps example metadata to renderable example components
- `src/components/Gallery/`, `GalleryItems/`, `Search/`, `Navigation/`, `DrawerContent/`, `Breadcrumbs/` — gallery browsing UI
- `examples-metadata.json` (repo root) — the master list of example entries (title, path, tags, image) that drives the gallery, sitemap, and footer links; `before-generate.js`/`after-generate.js` + `generate-sitemap.ts`/`generate-footer-links.ts` consume it
- `src/server/` — Express server used in production mode (`webpack.server*.config.js`)
- Multiple webpack configs exist for different purposes: `webpack.client.config.js` (prod client), `webpack.client.dev.config.js` / `webpack.client.no_server.config.ts` (dev, client-only), `webpack.server*.config.js` (Express server), `webpack.image.config.js` (image pipeline), `webpack.deps.config.ts` (prebuilds SciChart example dependency bundles)

### Example structure
Each example under `src/components/Examples/...` (e.g. `FeaturedApps/MedicalCharts/VitalSignsMonitorDemo/`) is self-contained: an `index.tsx` (React wrapper for the gallery), a `drawExample.ts`/`.js` pair (the actual chart-building logic, kept in both TS and compiled/vanilla JS so it can be embedded standalone in CodeSandbox/StackBlitz), and an `exampleInfo.ts` describing metadata for the gallery entry. `compileVanillaExamples` is what produces the `.js` siblings from the `.ts` sources — run it after editing an example's TS logic if the vanilla/embedded version needs to stay in sync.

### Vital Signs Monitor (current focus)
`src/components/Examples/FeaturedApps/MedicalCharts/VitalSignsMonitorDemo/` is a full biofeedback dashboard, not a simple chart demo:
- `ecgProcessor.ts`, `signalProcessor.ts`, `gsrTrendAnalyzer.ts` — signal processing (ECG/GSR analysis, Pan-Tompkins-style peak detection)
- `firebaseLogger.ts`, `sessionRecorder.ts` — session data logging to Firebase Realtime Database
- `data/` — sample/recorded datasets (CSV + generated `.ts`/`.js`) used for offline playback/testing
- `firmware/Heart_Rate_Display_Arduino.ino` — companion Arduino sketch (separate from the top-level `esp32/` firmware, which is the primary ESP32 sensor firmware: `esp32/main.ino`, `mpu6050.ino`, `PanTompkins.h`)
- Config is loaded via `src/config/environment.ts`, which reads `REACT_APP_FIREBASE_*` vars from `.env` (see [README_ENVIRONMENT_SETUP.md](README_ENVIRONMENT_SETUP.md)) — restart the dev server after editing `.env`
- Firebase setup/debugging: [README_FIREBASE_FRONTEND.md](README_FIREBASE_FRONTEND.md), [FIREBASE_TROUBLESHOOTING.md](FIREBASE_TROUBLESHOOTING.md)
- Recorded vital-signs dataset format: [VITAL_SIGNS_DATA_DOCUMENTATION.md](VITAL_SIGNS_DATA_DOCUMENTATION.md)

### Unrelated sibling project
`vitals-monitoring/` at the repo root is a **separate standalone app** using LightningChart JS (not SciChart) — it has its own `package.json`, `webpack.config.js`, and is not wired into the gallery build. Don't assume changes to the SciChart demo app affect it, and vice versa.

## Licensing note
SciChart.js runs under a free Community License by default for non-commercial/evaluation use (see [readme.md](readme.md)); no license key setup is needed for local dev.
