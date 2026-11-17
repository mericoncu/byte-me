## Dataset 1: Billboard Hot 100 (1958-2021)
- Source and URL: Kaggle, dhruvildave, "Billboard The Hot 100 Songs"
  https://www.kaggle.com/datasets/dhruvildave/billboard-the-hot-100-songs
  · License: <check Kaggle page and copy exactly> · Date accessed: <2026-10-14>
- Time range: 1958-08-04 to 2021-11-06 · Granularity: one row per song per weekly chart
  · Size: ~330,000 rows x 7 columns (confirm after download)
- Columns used:
  - date (date, chart week)
  - rank (int, 1-100, position that week)
  - song (str, title)
  - artist (str, credited artist string)
  - last-week (int, previous week's rank, NaN for new entries)
  - peak-rank (int, best rank reached up to that week)
  - weeks-on-board (int, weeks on chart up to that week)
- Known issues: artist strings mix "Featuring", "&", "x" (no stable artist ID);
  same title by different artists; re-entries after dropping off; last-week missing
  for new entries; right-censoring (songs still charting in Nov 2021);
  chart-rule changes over time (e.g. streaming added, recurrent rules) shift lifespans.
- Cleaning decisions and why:
  - Parse date to datetime; sort by date.
  - Build song_id = lowercase(strip(song)) + "||" + lowercase(strip(artist)) so each song is one unit.
  - Aggregate to one row per song: debut_date, last_date, total_weeks (count of chart rows,
    robust to re-entries), peak (min rank), max weeks-on-board for cross-check.
  - Drop songs debuting in the last 52 weeks from lifespan trends (censoring) and say so.
  - Keep original columns untouched in data/raw; write cleaned data to data/processed.
- How to regenerate: python scripts/download_data.py && python scripts/preprocess.py

## Dataset 2: Top Spotify Songs in 73 Countries
- Source and URL: Kaggle, asaniczka, "Top Spotify Songs in 73 Countries (Daily Updated)"
  https://www.kaggle.com/datasets/asaniczka/top-spotify-songs-in-73-countries-daily-updated
  · License: <check Kaggle page and copy exactly> · Date accessed: <2026-10-14>
- Time range: from ~2023-10 to the access date (dataset is updated daily; freeze the snapshot)
  · Granularity: one row per track per country per day (Top 50)
  · Size: <rows> x ~25 columns (confirm after download)
- Columns used:
  - spotify_id (str, track ID), name (str, title), artists (str)
  - country (str, ISO-2 code; empty = Global chart), snapshot_date (date)
  - daily_rank (int, 1-50), daily_movement, weekly_movement (int, rank change)
  - popularity (int 0-100, Spotify score at scrape time)
  - album_release_date (date, mixed formats)
  - danceability, energy, valence, acousticness, speechiness, tempo, loudness, duration_ms (audio features)
- Known issues: empty country = Global chart (not a country); some countries start later
  or have gaps; same song can have several spotify_ids (album vs single vs re-release);
  album_release_date formats vary (year / year-month / full date); popularity is a snapshot,
  not historical; non-Latin text (needs UTF-8); possible duplicate rows per snapshot.
- Cleaning decisions and why:
  - Separate Global rows from country rows; use Global only as a reference.
  - Drop exact duplicates on (spotify_id, country, snapshot_date).
  - Build track_key = lowercase(name) + "||" + lowercase(first artist) to merge duplicate IDs.
  - Standardise album_release_date to a datetime (missing month/day -> Jan 1) and flag it.
  - Freeze one download date and use only complete weeks, so results are reproducible.
- How to regenerate: python scripts/download_data.py && python scripts/preprocess.py
