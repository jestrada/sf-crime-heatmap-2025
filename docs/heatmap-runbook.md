# Crime Heatmap Runbook

Step-by-step process for building a city crime heatmap from raw incident data.

## Prerequisites
- DuckDB (for CSV exploration)
- Python 3 (for data processing + HTML generation)
- D3.js v7 (loaded via CDN in output HTML)

## Input
A CSV of crime incidents with at minimum:
- **Date/time** column (to derive week-of-year)
- **Location grouping** column (neighborhood, police beat, district, etc.)
- **Crime category** column (to filter homicides)
- **Lat/lon** or location coordinates (for geographic ordering)

## Steps

### 1. Explore the data
```bash
duckdb << EOF
SELECT COUNT(*) as rows, COUNT(DISTINCT neighborhood) as areas,
  MIN(date) as min_date, MAX(date) as max_date
FROM 'input.csv';

-- Check crime categories
SELECT crime_category, COUNT(*) FROM 'input.csv' GROUP BY 1 ORDER BY 2 DESC LIMIT 20;

-- Check for homicides specifically
SELECT crime_category, description, COUNT(*)
FROM 'input.csv'
WHERE crime_category ILIKE '%homicide%' OR description ILIKE '%murder%'
GROUP BY 1, 2 ORDER BY 3 DESC;
EOF
```

### 2. Map raw areas to neighborhoods (if needed)
If data uses police beats or other codes instead of neighborhood names:

1. Compute centroid lat/lon for each area from the data
2. Reverse geocode centroids via Nominatim: `https://nominatim.openstreetmap.org/reverse?lat={lat}&lon={lon}&format=json&zoom=16`
3. Rate limit: 1 req/sec, use `User-Agent` header
4. Fix any "Unknown" results by trying `zoom=14` or manual assignment
5. Save mapping as `beat_to_neighborhood.csv` (beat, neighborhood, lat, lon)

### 3. Aggregate to weekly counts
For each neighborhood × week (1-52):
- **Crime count**: total incidents in that neighborhood during that week
- **Homicide count**: filtered to actual murders only (exclude attempted, unexplained deaths, etc.)

### 4. TSP-optimize neighborhood ordering
Neighborhoods ordered by geographic proximity so adjacent rows on the heatmap are near each other on the map.

1. Compute centroid lat/lon per neighborhood
2. Start from highest-crime neighborhood
3. Run nearest-neighbor chain
4. Improve with 2-opt optimization (+ random restarts)
5. Save ordering to `neighborhood_order.json` or `beat_order.json`

**Why not just sort by latitude?** Pure lat/lon sort creates nonsensical adjacencies (e.g., Inner Richmond next to Tenderloin in SF). TSP preserves actual geographic proximity.

### 5. Build self-contained HTML
The output HTML embeds all data as JavaScript arrays — no external file loading (avoids CORS issues with `file://`).

Key parameters:
- **Color scale**: `d3.scaleLinear().domain([0, maxValue]).range(["#f7fbff", "#d62728"])` (light blue → red)
- **Zero cells**: `#f7f7f7` (light gray, distinct from the color scale)
- **Homicide markers**: Lucide skull icon via `<symbol>` + `<use>` elements
- **Layout**: Fullscreen, no scrollbars — cells sized to fill viewport
- **Margin left**: ~135px for SF (shorter names), ~180px for Oakland (longer names like "Old Oakland Historic District")
- **Month labels**: Approximate week positions for Jan-Dec on x-axis
- **Tooltip**: Neighborhood name, week number, incident count, homicide count if any

### 6. Verify & publish
```bash
# Local preview
open crime_heatmap.html
# or
python3 -m http.server 8000

# Push to GitHub Pages
git add -A
git commit -m "Add [city] crime heatmap"
git push origin main
```

GitHub Pages URL pattern: `https://{user}.github.io/{repo}/{subdir}/crime_heatmap.html`

## File Structure
```
{city}-crime-heatmap/
├── crime_heatmap.html          # Self-contained heatmap (committed)
├── beat_to_neighborhood.csv    # Beat→neighborhood mapping (if applicable)
├── neighborhood_order.json     # TSP-optimized ordering
├── source_data.csv             # Raw data (gitignored, too large)
└── intermediate_*.csv          # Processed data (gitignored)
```

## Homicide Filtering by City

Different cities categorize homicides differently. Always inspect the data first.

### San Francisco
- Filter: `incident_category == 'Homicide'`
- Includes: Murder, Manslaughter, Vehicular Manslaughter
- Excludes: Attempted homicide (categorized under Assault)

### Oakland
- Filter: `crimetype == 'HOMICIDE' AND description CONTAINS 'MURDER'`
- Watch out: "SC UNEXPLAINED DEATH" (701 records!) is categorized as Homicide but is NOT murder
- Includes: MURDER, MURDER:FIRST DEGREE
- Excludes: Attempted murder, unexplained deaths, shooting incidents linked to homicide cases

## Color Scale Decision
User preference: light blue (#f7fbff) → red (#d62728). No yellow — YlOrRd was explicitly rejected. The blue-to-red scale provides better contrast and readability.
