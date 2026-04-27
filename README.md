# Spotify Global Chart & Audio Feature Analysis

End-to-end analysis of Spotify's **Global Weekly Top 50** (2021-01-07 → 2026-02-19), with audio features programmatically extracted via `librosa` (over YouTube audio) and artist/label enrichment from kworb and Groover. Final project for _Data Science in the Wild_.

**Final deliverable:** [`notebooks/06_final_analysis.ipynb`](notebooks/06_final_analysis.ipynb) — full Phase 4 write-up (introduction, data description, preregistration, analysis, conclusions, limitations).

---

## Research Questions

Our central research question is: **What contributes to a song's chart performance on Spotify?** We focus on analyzing this question from a few perspectives: artist collaborations, label power, and audio feature synergy.

The following are the specific data analysis questions:

1. **Collaboration & longevity** — Do songs with featured artists stay on the Global Top 50 longer than solo tracks, after controlling for first-week popularity?
2. **Label power & longevity** — Do tracks released under one of the "Big-3" major labels (Universal, Sony, Warner) stay on the chart longer than independent releases, after controlling for first-week popularity _and_ audio characteristics?
3. **Audio synergy & virality** — Does the **interaction** of energy and spectral brightness predict whether a song peaks inside the Top 15 (a "viral" tier), beyond either feature alone?

### Preregistered hypotheses (Phase 3 → tested in Phase 4)

- **H1 — Collaborations & longevity.** Songs with artist collaborations stay on the chart longer than solo tracks, controlling for first-week popularity. _(Result: fail to reject H0.)_
- **H2 — Label power & longevity.** Songs released under Big-3 major labels stay on the chart longer than independent releases, controlling for first-week popularity _and_ audio characteristics. _(Result: partially reject H0 — Big-3 subsidiary releases last ~1.7 more weeks; Big-3 parent attributions are not significant.)_
- **H3 — Audio synergy & virality.** The interaction of energy and spectral brightness predicts whether a song peaks inside the Top 15. _(Result: fail to reject H0; previous-week rank dominates.)_

Full interpretations are in `notebooks/06_final_analysis.ipynb`.

---

## Pipeline Overview

The project is split into seven notebooks. They are numbered in run order:

| Notebook                                           | Purpose                                                                                                                                                                                                                                           |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`notebooks/01_scrape_weekly_charts.ipynb`**      | Scrapes [Spotify Charts](https://charts.spotify.com) (regional · global · weekly) and downloads each week's CSV automatically using Selenium.                                                                                                     |
| **`notebooks/02_extract_audio_features.ipynb`**    | Concatenates weekly CSVs into one dataset, fetches each unique track's audio via `yt-dlp` (YouTube), and extracts nine `librosa` audio features.                                                                                                  |
| **`notebooks/03_enrich_artist_features.ipynb`**    | Adds `artist_count`, `collaboration_type`, `monthly_listeners` (kworb), and `popularity_score` (Groover) to every chart row.                                                                                                                      |
| **`notebooks/04_exploratory_data_analysis.ipynb`** | Phase 2 EDA: distributions, correlations, and the three RQ-framed exploratory analyses (Brand vs. Signal / Longevity Paradox / Popularity Premium).                                                                                               |
| **`notebooks/05_hypothesis_testing.ipynb`**        | Phase 3 preregistration: defines and runs H1 (OLS), H2 (OLS), H3 (logit).                                                                                                                                                                         |
| **`notebooks/06_final_analysis.ipynb`**            | **Phase 4 final deliverable** — full write-up reusing the canonical processed CSV; introduction, abridged data description, preregistration restatement, analysis with interpretations, supplemental EDA, conclusions, limitations, bibliography. |
| **`notebooks/07_additional_analysis.ipynb`**       | Follow-up analyses motivated by H1–H3's null / partial results — e.g. the 2024 audio-feature spike, year-to-year drift in audio "rules" of charting.                                                                                              |

**Run order:** 01 → 02 → 03 produces the canonical processed CSV (`data/processed/processed_chart_tracks_enriched_features.csv`); 04 → 05 → 06 → 07 are the analysis notebooks and all read directly from that CSV.

---

## Raw Data

- **Source.** [Spotify Charts](https://charts.spotify.com/home) — Regional Global Weekly, **2021-01-07 through 2026-02-19**. One row per (track × chart week).
- **Chart fields per weekly CSV:** `rank`, `uri`, `artist_names`, `track_name`, `source`, `peak_rank`, `previous_rank`, `weeks_on_chart`, `streams`, `date`.
- **Audio enrichment.** For each unique track, audio is fetched via `yt-dlp` (YouTube search by track + artist) and acoustic features are computed with `librosa`, then merged back onto the chart dataset by `uri`.
- **Artist enrichment.** Monthly listeners scraped from [kworb.net](https://kworb.net/spotify/listeners.html); popularity score from [Groover's public Spotify popularity endpoint](https://groover.co/en/lp/free-tools/spotify-popularity-score/).
- **Label enrichment.** Each unique `source` string was hand-mapped into a 3-class scheme (`raw_unique_source_labeled.csv`): `0 = Big-3 parent`, `1 = Big-3 subsidiary`, `2 = independent`.

> **Note.** Spotify's `audio_analysis` API is no longer available; we use YouTube + `librosa` as the replacement pipeline for all acoustic features below.

---

## Extracted Audio Features

| Feature              | Description                                   |
| -------------------- | --------------------------------------------- |
| `tempo`              | Estimated BPM                                 |
| `energy`             | Mean RMS energy (loudness proxy)              |
| `zero_crossing_rate` | Rate of sign changes (noisiness proxy)        |
| `spectral_centroid`  | Brightness / perceived pitch centre           |
| `spectral_rolloff`   | Frequency below which 85% of energy lies      |
| `mfcc_1`, `mfcc_2`   | First two Mel-frequency cepstral coefficients |
| `chroma_mean`        | Mean chroma energy (harmonic content)         |
| `chroma_std`         | Std dev of chroma (harmonic variation)        |

## Artist & Label Enrichment

| Column               | Description                                                     | Source                                                                                            |
| -------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `artist_count`       | Number of credited artists on the track                         | derived from `artist_names`                                                                       |
| `collaboration_type` | `0` = solo, `1` = collab                                        | derived from `artist_count`                                                                       |
| `monthly_listeners`  | Current monthly listeners for the primary (first-listed) artist | [kworb.net](https://kworb.net/spotify/listeners.html)                                             |
| `popularity_score`   | Spotify popularity score for the primary artist (0–100)         | [Groover Spotify Popularity Score](https://groover.co/en/lp/free-tools/spotify-popularity-score/) |
| `major_label`        | `0 = Big-3 parent`, `1 = Big-3 subsidiary`, `2 = independent`   | hand-mapped from `source`                                                                         |

---

## Setup

### Data pipeline (notebooks 01–03)

######test'
aeqlwjelkqwe
qwkejqlwkje

```bash
pip install selenium webdriver-manager pandas librosa numpy yt_dlp lxml
```

- **FFmpeg** is required for `yt_dlp` (audio extraction).
- **`01_scrape_weekly_charts.ipynb`** — first run: save Spotify login cookies (manual login in the browser once); later runs: scrape weekly charts (results batched by month, e.g. `spotify_weekly_data_2021-01.csv`).
- **`02_extract_audio_features.ipynb`** — set the chart-CSV directory variable to wherever you saved the weekly CSVs (recommended: `data/raw/`).
- **`03_enrich_artist_features.ipynb`** — the Groover scraper (cell 7) is documentation-only; the canonical `data/raw/raw_artist_popularity_scores.csv` is committed to the repo, and the merge step further down loads it back in.

### Analysis (notebooks 04–07)

```bash
pip install pandas numpy matplotlib seaborn statsmodels scikit-learn
```

All four analysis notebooks read from a single committed CSV — `data/processed/processed_chart_tracks_enriched_features.csv` — so you don't need to re-run the data pipeline to reproduce the analyses.

---

## Outputs

- **`01` (chart scraper):** Monthly CSVs (e.g. `spotify_weekly_data_YYYY-MM.csv`) and a cookie file for authenticated sessions.
- **`02` (audio features):**
  - `data/processed/processed_chart_tracks_audio_features.csv` — full chart dataset with audio features merged in.
- **`03` (artist enrichment):**
  - `data/processed/processed_chart_tracks_enriched_features.csv` — **canonical analysis-ready CSV** consumed by notebooks 04–07.
- **`06` (final analysis):** Phase 4 deliverable — fully executed Jupyter notebook with all interpretations and conclusions.

---

## Repository

**GitHub:** [https://github.com/Irene-Wu-1002/Spotify-Trend-Analyzer](https://github.com/Irene-Wu-1002/Spotify-Trend-Analyzer)

## Folder Structure

```text
Spotify-Trend-Analyzer/
├── notebooks/
│   ├── 01_scrape_weekly_charts.ipynb
│   ├── 02_extract_audio_features.ipynb
│   ├── 03_enrich_artist_features.ipynb
│   ├── 04_exploratory_data_analysis.ipynb
│   ├── 05_hypothesis_testing.ipynb
│   ├── 06_final_analysis.ipynb           # Phase 4 deliverable
│   └── 07_additional_analysis.ipynb      # Supplemental trend analyses
├── data/
│   ├── raw/
│   │   ├── raw_artist_popularity_scores.csv     # Groover popularity scores
│   │   ├── raw_spotify_top50_eda_input.csv      # snapshot used by 04
│   │   ├── raw_unique_source.csv                # unique label strings
│   │   └── raw_unique_source_labeled.csv        # hand-coded major_label classes
│   └── processed/
│       ├── processed_chart_tracks_audio_features.csv     # output of 02
│       └── processed_chart_tracks_enriched_features.csv  # output of 03 — analysis input
├── docs/
│   └── phase2_data_description.pdf       # full Phase 2 data description
├── reports/
├── src/
└── README.md
```
