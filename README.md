# WatchDawg Crime Analytics Dashboard

An interactive dashboard for visualizing Seattle crime data using Dash and SQLite. This dashboard provides real-time incident visualization and statistical insights.

## DATA Cleaning
Data can available to download at https://data.seattle.gov/Public-Safety/SPD-Crime-Data-2008-Present/tazs-3rd5/about_data. 
The raw crime data from the Seattle Police Department undergoes comprehensive cleaning and preprocessing before being loaded into the SQLite database. The cleaning process is documented in `Data Cleaning.ipynb` and includes the following steps:

### 1. Missing Value Standardization

- Replace placeholder values (`-`, `REDACTED`, `-1.0`) with `NaN` across all columns
- Standardize missing data representation for consistent handling

### 2. Geographic Data Validation

- Convert `Latitude` and `Longitude` to numeric types
- Validate coordinate ranges:
  - Latitude: 47.0° to 48.1° (Seattle area bounds)
  - Longitude: -123.5° to -121.0° (Seattle area bounds)
- Remove records with invalid or out-of-range coordinates

### 3. Neighborhood Data Cleaning

- Remove invalid neighborhood values:
  - `UNKNOWN`
  - `FK ERROR` (foreign key errors)
  - `OOJ` (out of jurisdiction)
- Drop records with missing neighborhood information

### 4. Crime Category Standardization

- Remove invalid `NIBRS Crime Against Category` values:
  - `ANY`
  - `NOT_A_CRIME`
- Ensure all records have valid crime categories (PERSON, PROPERTY, or SOCIETY)

### 5. Offense Sub-Category Cleaning

- Remove invalid sub-category values:
  - `UNKNOWN`
  - `999` (invalid code)
  - Empty strings
- Drop records with missing offense sub-category

### 6. Location Data Requirements

- Remove records missing `Latitude` or `Longitude` (required for map visualization)
- Remove records missing `Block Address`
- Remove records missing `Reporting Area`
- Remove records missing `Beat` information

### 7. Sector Data Cleaning

- Remove invalid sector values:
  - `None`
  - `99` (invalid code)
- Ensure all records have valid sector assignments

### 8. Shooting Type Handling

- Fill missing `Shooting Type Group` values with `"No Shooting"` to preserve all records

### 9. Temporal Data Processing

- Extract date components from `Offense Date`:
  - `Offense Year`
  - `Offense Month`
  - `Offense Day`
  - `Offense Time`
- Convert to appropriate datetime formats for analysis

### 10. Data Type Optimization

- Convert numeric columns to appropriate types (int, float)
- Optimize string columns for memory efficiency
- Ensure consistent data types across the dataset

### 11. Column Selection and Renaming

- Select relevant columns for dashboard visualization
- Rename columns to standardized names:
  - `Offense Category` → `offense`
  - `Offense Sub Category` → `offense_sub_category`
  - `NIBRS Crime Against Category` → `crime_against_category`
  - `Block Address` → `location`
  - `Neighborhood` → `area`
  - And others as needed

### Result

After cleaning, the dataset contains only records with:
- Valid geographic coordinates within Seattle boundaries
- Complete location information (address, neighborhood, precinct, sector)
- Valid crime classifications
- Properly formatted temporal data

This ensures high data quality and reliability for all dashboard visualizations and analyses.

## Features

- 📅 **Date Range Filter**: Select custom date ranges to analyze crime trends
- 📊 **KPI Cards**: Total incidents, crimes against property, person, and society
- 📈 **Trend Analysis**: Crime trends by neighborhood over time
- 📊 **Crime Type Distribution**: Interactive bar chart with drill-down capability
- 🗺️ **Interactive Map**: Point-based visualization with address search and radius filtering
- ⏰ **Time Filter**: Filter crimes by hour range
- 🏘️ **Neighborhood Filter**: Filter by specific neighborhoods
- 🎯 **Crime Category Filter**: Filter by crime type (Person, Property, Society)
- 📋 **Detailed Data Table**: View top 500 incidents with sorting and filtering

## CRITICAL EVALUATION (SUCCESS, STRUGGLES ETC)

Our interactive Seattle Crime Visualization was designed with a user-centered approach inspired by Cooper et al.’s About Face. From the beginning, we recognized that users evaluate safety differently depending on their goals, locations, and lived experiences. This insight led us to prioritize intuitive navigation, simple interaction, and a visualization that never makes users feel like they made a “wrong” choice. We focused on showing crime locations using spatial position—the most powerful visual encoding according to Bertin—to give users an instant, low-cognitive-load snapshot of citywide crime patterns. Additional details such as crime type and time of day were intentionally encoded with hue, size, and filters to enrich exploration without overwhelming users.

We refined our filters to three key dimensions—neighborhood, crime type, and hour of day—balancing usability with the limits of our dataset. Neighborhoods were chosen because they align with how people naturally think about location; crime types were consolidated from 25+ categories into a smaller, more interpretable set; and time-of-day filtering allows users to personalize risk based on daily routines. To support coordinated exploration, we incorporated summary plots highlighting crime distributions before users dive into the map. Ultimately, our process was heavily informed by our desire to provide the user as much agency as we could. This visualization process that is rich in context yet simple to navigate, minimizing clutter while maximizing meaningful insight.

## Quick Start

### Option 1: Use the Deployed Version (Recommended)

The dashboard is deployed and available at:

**🌐 [https://watchdawg-app.onrender.com/](https://watchdawg-app.onrender.com/)**

Simply open the link in your browser - no setup required! The database will be automatically downloaded from Google Drive on first use.

### Option 2: Run Locally

#### Prerequisites

- Python 3.11 or higher
- `crime_data_gold.db` SQLite database file (or it will be downloaded automatically) or you can convert to db from csv with convert_to_sqlite.py.

#### Setup Steps

1. **Clone or download this repository**

2. **Install Dependencies**

```bash
pip install -r requirements.txt
```

Required packages:
- `dash` - Web framework
- `dash-bootstrap-components` - UI components
- `dash-mantine-components` - Date picker component
- `pandas` - Data manipulation
- `numpy` - Numerical operations
- `plotly` - Interactive charts
- `gunicorn` - Production server (for deployment)
- `gdown` - Google Drive download
- `requests` - HTTP requests

3. **Prepare the Database**

The app will automatically download the database from Google Drive if it doesn't exist locally. However, if you have the `crime_data_gold.db` file:

- Place `crime_data_gold.db` in the same directory as `app.py`
- The app will use the local file if it exists

4. **Run the Application**

```bash
python app.py
```

The app will:
- Check for `crime_data_gold.db` in the current directory
- If not found, automatically download it from Google Drive
- Start the server on `http://localhost:8050`

5. **Open in Browser**

Navigate to `http://localhost:8050` in your web browser.

## Data Source

The dashboard uses a SQLite database (`crime_data_gold.db`) containing Seattle crime incident data from the [Seattle Police Department](https://data.seattle.gov/Public-Safety/SPD-Crime-Data-2008-Present/tazs-3rd5/about_data), retrieved on December 4, 2025.

All incidents are reported according to the **National Incident Based Reporting System (NIBRS)**, which captures detailed contextual information for each crime, including:
- Precise location coordinates
- Crime classification
- Temporal data
- Neighborhood boundaries (using official City of Seattle Neighborhood Map)

## Database Schema

The `crimes` table contains the following columns:

- `date` - Date of the crime (DATE)
- `time` - Time of the crime (STRING)
- `hour` - Hour of the day (0-23) (INTEGER)
- `datetime` - Combined date and time (STRING)
- `offense` - Type of crime (STRING)
- `offense_sub_category` - Sub-category of crime (STRING)
- `crime_against_category` - Category: PERSON, PROPERTY, or SOCIETY (STRING)
- `location` - Block address (STRING)
- `area` - Neighborhood name (STRING)
- `precinct` - Police precinct (STRING)
- `sector` - Police sector (STRING)
- `hazardness` - Hazard score (FLOAT)
- `latitude` - Latitude coordinate (FLOAT)
- `longitude` - Longitude coordinate (FLOAT)

## Usage Guide

### Date Range Selection

1. Click the **📅 Date Filter** button in the top-left sidebar
2. Select a date range using the date picker
3. The dashboard will automatically update all visualizations

### Map Interaction

1. **View Crime Points**: Each dot represents a crime incident, colored by category:
   - 🔴 Red: Crimes against Person
   - 🟡 Yellow: Crimes against Property
   - 🔵 Blue: Crimes against Society

2. **Search by Address**:
   - Enter a Seattle address in the search box
   - Click "Search" to center the map
   - Adjust the radius slider to filter crimes within that distance

3. **Filter Options**:
   - **Time Filter**: Use the vertical slider to select hour ranges
   - **Neighborhood Filter**: Select one or more neighborhoods
   - **Crime Type Filter**: Check/uncheck crime categories

### Charts

- **Trend Line Chart**: Shows top/bottom 10 neighborhoods by crime count or hazard score
  - Toggle between "Count" and "Hazard Score" metrics
  - Switch between "Highest" and "Lowest" neighborhoods

- **Crime Type Bar Chart**: 
  - Click on a category bar to drill down into subcategories
  - Click "← Back" to return to category overview

### Data Table

- View the top 500 incidents matching your filters
- Sort and filter columns directly in the table
- Use pagination to navigate through results

## Memory Optimization

This dashboard is optimized for memory efficiency:

- **On-Demand Loading**: Data is loaded only when needed, not all at once
- **Date Range Queries**: Only queries data within the selected date range
- **Efficient Data Types**: Uses optimized pandas data types (category, int8, float32)
- **No Caching**: Prevents memory spikes by avoiding full dataset caching

This allows the app to run efficiently on Render's free tier (512MB memory limit).

## Deployment

The dashboard is deployed on [Render](https://render.com) using:

- **Runtime**: Python 3.11
- **Server**: Gunicorn with 1 worker, 2 threads
- **Database**: Automatically downloaded from Google Drive on first deployment

### Environment Variables

- `DB_GDRIVE_URL`: Google Drive URL for the SQLite database
- `DASH_DEBUG`: Set to "0" for production
- `PORT`: Automatically set by Render

## Troubleshooting

### Database Not Found

If you see "No data loaded" errors:

1. Check that `crime_data_gold.db` exists in the project directory
2. Verify Google Drive URL is accessible (check `DB_GDRIVE_URL` in `render.yaml`)
3. Check application logs for download errors

### Memory Issues

If you encounter memory errors:

1. Reduce the date range selection
2. Use filters to narrow down results
3. The app automatically limits data loading for memory efficiency

### Port Already in Use

If port 8050 is already in use:

```bash
# Set a different port
export PORT=8051
python app.py
```

## Team

**WatchDawg**: Doyoung Jung, Wonjoon Hwang, Aneesh Singh, DH Lee, Jungmoon Ha, Jonathan Langley Grothe, Derek Tropf

## License

This project is for educational purposes as part of DATA 511.

## Links

- **Live Dashboard**: [https://watchdawg-app.onrender.com/](https://watchdawg-app.onrender.com/)
- **Data Source**: [Seattle Police Department Crime Data](https://data.seattle.gov/Public-Safety/SPD-Crime-Data-2008-Present/tazs-3rd5/about_data)
