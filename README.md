# Spotify Global Chart & Audio Feature Analysis

Analysis of Spotify’s Global Weekly Chart data with programmatically extracted audio features, for the *Data Science in the Wild* final project.

---

## Research Questions

1. **What audio characteristics are most common among songs in Spotify’s Global Top 50?**  
   We examine whether globally popular songs share similar musical patterns by analyzing tempo, energy, spectral centroid, spectral rolloff, MFCCs, and chroma features to identify dominant acoustic traits of mainstream hits.

2. **Do the audio characteristics of Global Top 50 songs vary over time?**  
   We investigate whether the dominant sound of popular music changes across time periods by analyzing temporal variation in tempo, energy, and timbral descriptors in weekly chart data.

3. **Do emotionally expressive audio features correlate with higher global popularity?**  
   We explore whether features such as energy, tempo, and spectral brightness correlate with chart rank, streams, and weeks on chart to understand if specific musical qualities contribute to global success.

---

## Pipeline Overview

The project is split into two notebooks:

| Notebook | Purpose |
|----------|--------|
| **`spotify_chart_scraper.ipynb`** | Scrapes [Spotify Charts](https://charts.spotify.com) (regional-global-weekly) and downloads each week’s CSV automatically. |
| **`audio_feature_extraction.ipynb`** | Merges weekly CSV files into one dataset and extracts audio features using **librosa** (with audio sourced via **yt-dlp** from YouTube). |

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

### Chart scraper (`spotify_chart_scraper.ipynb`)

```bash
pip install selenium webdriver-manager pandas
```

- **First run:** Use the notebook to save login cookies (manual Spotify login in the browser once).
- **Later runs:** Scrape weekly charts; the notebook batches results by month (e.g. `spotify_weekly_data_2021-01.csv`).

### Audio feature extraction (`audio_feature_extraction.ipynb`)

```bash
pip install librosa numpy pandas yt_dlp
```

- **FFmpeg** is required for `yt_dlp` (audio extraction).
- Set `CHART_CSV_DIR` (or equivalent path variable) in the notebook to the directory containing your weekly chart CSVs.

---

## Outputs

- **Chart scraper:** Monthly CSV files (e.g. `spotify_weekly_data_YYYY-MM.csv`) and a cookie file for authenticated sessions.
- **Audio notebook:**  
  - `unique_songs_audio_features.csv` — one row per unique track with extracted features.  
  - `final_songs_with_audio_features.csv` — full chart dataset with audio features merged in.

---

## Repository

**GitHub:** [https://github.com/Irene-Wu-1002/Spotify-Trend-Analyzer](https://github.com/Irene-Wu-1002/Spotify-Trend-Analyzer)
