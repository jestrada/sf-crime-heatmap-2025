# San Francisco Crime Heatmap 2025

An interactive heatmap visualization of crime incidents across San Francisco neighborhoods in 2025, aggregated by week.

## 🎯 Overview

This project analyzes **89,954 crime incidents** from the San Francisco Police Department's 2025 data, visualizing:
- **41 neighborhoods** (rows)
- **52 weeks** (columns)
- **Weekly incident counts** with color intensity (light blue = few incidents, red = many)

## 📊 Key Findings

### Top Crime Types (2025)
1. **Larceny Theft** – 20.71% (vehicle theft, shoplifting, general theft)
2. **Assault** – 7.98% (battery, simple/aggravated assault)
3. **Drug Offense** – 7.4% (narcotics possession, paraphernalia)
4. **Malicious Mischief** – 6.54% (vandalism)
5. **Warrant** – 6.13% (warrant arrests)

### Crime Hotspots
- **Tenderloin** – Consistently highest incidents throughout the year
- **South of Market** – High property crime and drug offenses
- **Mission** – Elevated assault and larceny

### Temporal Patterns
- **Peak Day**: Wednesday (15.85% of weekly incidents)
- **Peak Hour**: Noon & 4-5pm (afternoon surge)
- **Lowest**: Sunday (12.54%), early morning 4-5am

### Resolution Status
- **67.75%** Open or Active
- **31.51%** Cite or Arrest Adult
- **0.42%** Unfounded
- **0.32%** Exceptional Adult

## 📁 Files

- **crime_heatmap.html** – Interactive fullscreen heatmap (self-contained, no dependencies needed)
- **2025_sf_crimes_detail.csv** – All 2025 incidents with neighborhood, category, coordinates, resolution
- **2025_crime_heatmap_long.csv** – Long-format: neighborhood × day with incident count
- **2025_crime_heatmap_wide.csv** – Wide-format: neighborhoods as rows, days as columns

## 🚀 Usage

### View the Heatmap
1. Download `crime_heatmap.html`
2. Open in any web browser (no server required)
3. Hover over cells to see exact incident counts

### Data Files
All CSV files include:
- Date/day-of-year
- Neighborhood (analysis neighborhood)
- Incident category & subcategory
- Incident description & code
- Police district
- Latitude/longitude coordinates
- Resolution status

## 🛠️ Technical Details

- **Visualization**: D3.js v7
- **Data Source**: San Francisco Police Department 2025 data
- **Time Period**: Jan 1 – Dec 31, 2025 (partial year, data as of Feb 28, 2026)
- **Neighborhoods**: 41 unique analysis neighborhoods
- **Cell Coloring**: Linear scale from light blue (0 incidents) to red (max incidents per week)

## 📈 Data Quality

- **89,954 incidents** with neighborhoods (5.1% missing location data overall)
- **95.3%** have valid latitude/longitude coordinates
- **100%** have police district assigned
- **78.7%** remain Open or Active

## 🎨 How to Read the Heatmap

- **Rows**: 41 SF neighborhoods (alphabetically ordered)
- **Columns**: 52 weeks of 2025
- **Cell Color**: Intensity indicates incident count for that week/neighborhood
  - Light blue = few incidents
  - Red = many incidents
  - Hover for exact numbers

## 📚 Data Sources

- San Francisco Police Department (SFPD) incident data
- Dataset: `wg3w-h783_version_8124.csv`
- Data loading date: June 13, 2025
- Data accuracy as of: February 28, 2026

## 🔗 Links

- [Interactive Heatmap](https://jestrada.github.io/sf-crime-heatmap-2025/crime_heatmap.html)

---

Created: March 1, 2026
