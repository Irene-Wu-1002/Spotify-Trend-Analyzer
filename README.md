# Spotify Global Chart & Audio Feature Analysis

Analysis of Spotify’s Global Weekly Chart data with programmatically extracted audio features, for the *Data Science in the Wild* final project.

---

## Research Questions

1.   **What audio characteristics are most common among songs in Spotify’s Global Top 50 playlist?**

     We want to examine whether globally popular songs share similar musical patterns. By analyzing extracted audio features such as tempo, energy, spectral centroid, spectral rolloff, MFCC coefficients, and chroma features, we aim to identify dominant acoustic characteristics that define mainstream global hits.

2.   **Do the audio characteristics of Spotify’s Global Top 50 songs vary over time?**
    
     We want to investigate whether the dominant sound of globally popular music changes across different time periods. Using weekly Global Top 50 data, we analyze temporal variation in tempo, energy, and timbral descriptors to detect stylistic trends. This allows us to examine whether mainstream music remains stable or evolves dynamically over time.

3. **Do emotionally expressive audio features correlate with higher global popularity?**
    
    We want to explore whether certain emotionally associated features (e.g., higher energy, faster tempo, brighter spectral characteristics) are associated with higher chart rankings or greater streaming counts. By examining relationships between audio features and popularity metrics (rank, streams, weeks on chart), we aim to understand whether specific musical qualities systematically contribute to global success.
---

## Pipeline Overview

The project is split into two notebooks:

| Notebook | Purpose |
|----------|--------|
| **`notebooks/01_scrape_weekly_charts.ipynb`** | Scrapes [Spotify Charts](https://charts.spotify.com) (regional-global-weekly) and downloads each week’s CSV automatically. |
| **`notebooks/02_02_extract_audio_features.ipynb`** | Merges weekly CSV files into one dataset and extracts audio features using **librosa** (with audio sourced via **yt-dlp** from YouTube). |

**Run order:** Run the chart scraper first to obtain weekly CSVs; then run the audio feature notebook (point it at the folder containing those CSVs).

---

## Raw Data

- **Source:** [Spotify Charts](https://charts.spotify.com/home) — Regional Global Weekly (e.g. 2021-01-01 through 2026-02-19).
- **Chart fields (per weekly CSV):** rank, track URI, artist names, track name, source, peak rank, previous rank, weeks on chart, streams.
- **Enrichment:** For each track, audio is fetched via **yt-dlp** (YouTube search by track + artist). **librosa** is used to compute acoustic features, which are then merged back into the chart dataset using track URI.

**Note:** Spotify’s `audio_analysis` API is no longer available; we use YouTube + librosa as the replacement pipeline.

---

## Extracted Audio Features

| Feature | Description |
|--------|-------------|
| `tempo` | Estimated BPM |
| `energy` | Mean RMS energy (loudness proxy) |
| `zero_crossing_rate` | Rate of sign changes (noisiness proxy) |
| `spectral_centroid` | Brightness / perceived pitch centre |
| `spectral_rolloff` | Frequency below which 85% of energy lies |
| `mfcc_1`, `mfcc_2` | First two Mel-frequency cepstral coefficients |
| `chroma_mean` | Mean chroma energy (harmonic content) |
| `chroma_std` | Std dev of chroma (harmonic variation) |

---

## Setup

### Chart scraper (`notebooks/01_scrape_weekly_charts.ipynb`)

```bash
pip install selenium webdriver-manager pandas
```

- **First run:** Use the notebook to save login cookies (manual Spotify login in the browser once).
- **Later runs:** Scrape weekly charts; the notebook batches results by month (e.g. `spotify_weekly_data_2021-01.csv`).

### Audio feature extraction (`notebooks/02_02_extract_audio_features.ipynb`)

```bash
pip install librosa numpy pandas yt_dlp
```

- **FFmpeg** is required for `yt_dlp` (audio extraction).
- Set `CHART_CSV_DIR` (or equivalent path variable) in the notebook to the directory containing your weekly chart CSVs (recommended: `data/raw/`).

---

## Outputs

- **Chart scraper:** Monthly CSV files (e.g. `spotify_weekly_data_YYYY-MM.csv`) and a cookie file for authenticated sessions.
- **Audio notebook:**  
  - `data/processed/unique_songs_audio_features.csv` — one row per unique track with extracted features.  
  - `data/processed/processed_chart_tracks_audio_features.csv` — full chart dataset with audio features merged in.

---

## Repository

**GitHub:** [https://github.com/Irene-Wu-1002/Spotify-Trend-Analyzer](https://github.com/Irene-Wu-1002/Spotify-Trend-Analyzer)

## Folder Structure

```text
Spotify-Trend-Analyzer/
├── notebooks/
│   ├── 04_exploratory_data_analysis.ipynb
│   ├── 03_enrich_artist_features.ipynb
│   ├── 02_extract_audio_features.ipynb
│   ├── 05_hypothesis_testing.ipynb
│   └── 01_scrape_weekly_charts.ipynb
├── data/
│   ├── raw/
│   │   ├── raw_artist_popularity_scores.csv
│   │   └── raw_spotify_top50_eda_input.csv
│   └── processed/
│       ├── processed_chart_tracks_audio_features.csv
│       └── processed_chart_tracks_enriched_features.csv
├── docs/
├── reports/
├── src/
└── README.md
```
