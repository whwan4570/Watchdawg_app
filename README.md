# WatchDawg: Seattle Crime Analytics Dashboard

An interactive web dashboard for exploring Seattle crime incident data, built with
[Dash](https://dash.plotly.com/) and backed by a SQLite database. The dashboard
lets users filter incidents by date, time of day, neighborhood, and crime category,
and visualizes the results through KPI cards, trend charts, a drill-down bar chart,
and an interactive map.

**Course:** DATA 511 - Data Visualization

**Team WatchDawg:** Doyoung Jung, Wonjoon Hwang, Aneesh Singh, DH Lee, Jungmoon Ha, Jonathan Langley Grothe, Derek Tropf

---

## Table of Contents

1. [Running the App](#running-the-app)
2. [Screenshots](#screenshots)
3. [Project Overview](#project-overview)
4. [Tech Stack](#tech-stack)
5. [Project Structure](#project-structure)
6. [Data Source](#data-source)
7. [Data Cleaning and Preprocessing](#data-cleaning-and-preprocessing)
8. [Database Schema](#database-schema)
9. [Architecture Notes](#architecture-notes)
10. [Deployment](#deployment)
11. [Troubleshooting](#troubleshooting)
12. [License and Acknowledgments](#license-and-acknowledgments)

---

## Running the App

> **Note:** The previously hosted demo (`https://watchdawg-app.onrender.com/`) is
> **currently unavailable**. Please run the dashboard locally using the steps below.

Quick start (assumes Python 3.11+ is installed):

```bash
# 1. Move into the project folder
cd watchdawg_app

# 2. (Recommended) Create and activate a virtual environment
python3 -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run the app (downloads the database from Google Drive on first run)
python app.py
```

Then open **http://localhost:8050** in your browser. Press `Ctrl+C` in the terminal
to stop the server. To use a different port, set the `PORT` environment variable
(e.g. `export PORT=8051`). On first run the database is downloaded from Google Drive
automatically; you can also generate it locally from the source CSV with
`python convert_to_sqlite.py`.

---

## Screenshots

### Dashboard Overview

KPI cards, neighborhood crime trends, and the crime-type drill-down chart.

![Dashboard overview](img/dashboard.png)

### Interactive Map with Address Search and Radius

Search an address, draw a radius circle, and explore incident points by category
and time of day.

![Interactive map with address search and radius filter](img/map.png)

---

## Project Overview

Seattle residents, students, and visitors often want to understand the safety
profile of a neighborhood, but public crime data is large, raw, and hard to
interpret. WatchDawg turns the Seattle Police Department's incident records into an
interactive dashboard that answers practical questions such as:

- How does crime volume vary across neighborhoods and over time?
- What types of crime are most common in a given area?
- What does the incident distribution look like within walking distance of a
  specific address?
- How do patterns change by time of day?

The interface is designed to give users an immediate spatial overview and then let
them progressively narrow the data with a small set of intuitive filters.

---

## Tech Stack

| Layer          | Technology                                              |
| -------------- | ------------------------------------------------------- |
| Web framework  | Dash (Flask under the hood)                             |
| UI components  | dash-bootstrap-components, dash-mantine-components      |
| Visualization  | Plotly                                                  |
| Data handling  | pandas, numpy                                           |
| Storage        | SQLite                                                  |
| Geocoding      | OpenStreetMap Nominatim API                             |
| Deployment     | Render (Gunicorn)                                       |
| File transfer  | gdown / requests (Google Drive download)               |

---

## Project Structure

```
watchdawg_app/
├── app.py                  # Main Dash application (layout + callbacks)
├── convert_to_sqlite.py    # Converts crime_data_gold.csv -> crime_data_gold.db
├── Data Cleaning.ipynb     # Data cleaning and preprocessing notebook
├── requirements.txt        # Python dependencies
├── render.yaml             # Render deployment configuration
├── assets/                 # Logo images and custom CSS (auto-served by Dash)
└── README.md
```

> The SQLite database (`crime_data_gold.db`) is **not** committed to the repository.
> It is downloaded from Google Drive at runtime, or you can generate it locally from
> the source CSV using `convert_to_sqlite.py`.

---

## Data Source

The dashboard uses Seattle Police Department crime incident data from the
[Seattle Open Data Portal](https://data.seattle.gov/Public-Safety/SPD-Crime-Data-2008-Present/tazs-3rd5/about_data),
retrieved on December 4, 2025.

All incidents follow the **National Incident Based Reporting System (NIBRS)**, which
records detailed context for each crime, including location coordinates, crime
classification, and temporal data. Neighborhood boundaries are assigned using the
official City of Seattle Neighborhood Map.

---

## Data Cleaning and Preprocessing

Raw SPD data is cleaned in `Data Cleaning.ipynb` before being loaded into SQLite.
The process includes:

1. **Missing value standardization** - replace placeholders (`-`, `REDACTED`, `-1.0`) with `NaN`.
2. **Geographic validation** - keep only coordinates within Seattle bounds (Latitude 47.0-48.1, Longitude -123.5 to -121.0).
3. **Neighborhood cleaning** - drop `UNKNOWN`, `FK ERROR`, `OOJ`, and missing neighborhoods.
4. **Crime category standardization** - remove `ANY` and `NOT_A_CRIME`; keep PERSON, PROPERTY, SOCIETY.
5. **Offense sub-category cleaning** - drop `UNKNOWN`, `999`, and empty values.
6. **Location requirements** - require coordinates, block address, reporting area, and beat.
7. **Sector cleaning** - remove `None` and `99`.
8. **Shooting type handling** - fill missing shooting type with `"No Shooting"`.
9. **Temporal processing** - derive year, month, day, time, and hour fields.
10. **Type optimization** - convert columns to appropriate numeric and string types.
11. **Column selection and renaming** - rename to dashboard-friendly names (e.g. `Block Address` → `location`, `Neighborhood` → `area`).

The result is a clean dataset with valid coordinates, complete location data, valid
classifications, and well-formed timestamps.

---

## Database Schema

The `crimes` table contains the following columns:

| Column                   | Type    | Description                             |
| ------------------------ | ------- | --------------------------------------- |
| `date`                   | DATE    | Date of the incident                    |
| `time`                   | STRING  | Time of the incident                    |
| `hour`                   | INTEGER | Hour of day (0-23)                      |
| `datetime`              | STRING  | Combined date and time                  |
| `offense`                | STRING  | Offense category                        |
| `offense_sub_category`   | STRING  | Offense sub-category                    |
| `crime_against_category` | STRING  | PERSON, PROPERTY, or SOCIETY            |
| `location`               | STRING  | Block address                           |
| `area`                   | STRING  | Neighborhood name                       |
| `precinct`               | STRING  | Police precinct                         |
| `sector`                 | STRING  | Police sector                           |
| `hazardness`             | FLOAT   | Opinionated hazard score                |
| `latitude`               | FLOAT   | Latitude coordinate                     |
| `longitude`              | FLOAT   | Longitude coordinate                    |

Indexes are created on `date`, `hour`, `area`, `crime_against_category`, and
`(latitude, longitude)` to speed up queries.

---

## Architecture Notes

The app is designed to run within Render's free tier (512 MB memory limit):

- **On-demand loading.** Data is queried from SQLite as needed rather than held
  entirely in memory.
- **Date-range queries in SQL.** The selected date range (and a coordinate
  not-null check) is pushed down to the SQL `WHERE` clause, so only rows within the
  chosen range are read from disk.
- **In-memory filtering for the rest.** Additional filters - hour window, crime
  category, neighborhood, and the address radius - are applied in pandas after the
  date-range query returns.
- **Efficient dtypes.** String columns are converted to `category` and numeric
  columns to compact types (`int8`, `float32`) after loading.

> Note: to keep memory usage predictable, the app does not cache query results, so
> changing a filter re-runs the date-range query for each affected view.

---

## Deployment

The dashboard is deployed on [Render](https://render.com) using the configuration in
`render.yaml`:

- **Runtime:** Python 3.11
- **Server:** Gunicorn with 1 worker and 2 threads
- **Database:** downloaded from Google Drive on first run

### Environment Variables

- `DB_GDRIVE_URL` - Google Drive URL for the SQLite database. A default is defined in
  `app.py`; set this variable to override it.
- `DASH_DEBUG` - set to `0` for production (debug is on by default when running locally).
- `PORT` - the port to bind to (set automatically by Render).

---

## Troubleshooting

**"No data loaded" / database not found**
- Confirm `crime_data_gold.db` exists next to `app.py`, or that the app can reach the
  Google Drive URL.
- Check the application logs for download errors; a Google Drive sharing
  misconfiguration can cause an HTML page to be downloaded instead of the database.

**Memory issues**
- Narrow the date range and apply filters to reduce the amount of data loaded.

**Port already in use**
- Set a different port: `export PORT=8051 && python app.py`.

---

## License and Acknowledgments

This project was developed for educational purposes as part of **DATA 511**.

- Data: [Seattle Police Department Crime Data](https://data.seattle.gov/Public-Safety/SPD-Crime-Data-2008-Present/tazs-3rd5/about_data)
- Geocoding: [OpenStreetMap Nominatim](https://nominatim.openstreetmap.org/)
