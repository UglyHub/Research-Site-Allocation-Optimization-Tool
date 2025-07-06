# Research Site Allocation Optimization Tool

## Overview and Project Goals

This project presents a sophisticated, data-driven methodology for identifying optimal locations for the establishment of new research sites within England and Wales. The core objective is to strategically select sites that maximize impact by focusing on areas characterized by high indices of multiple deprivation (IMD) while also considering the logistical advantages of proximity to existing healthcare facilities and established research infrastructure.

By integrating and analyzing comprehensive datasets related to demographics, healthcare facilities, and existing research projects, this tool provides a quantitative framework for decision-making in research site allocation. The output includes a prioritized list of potential locations and an interactive geographical visualization to facilitate deeper understanding and strategic planning.

The analysis is conducted using Python, leveraging powerful libraries such as `pandas` for data manipulation, `numpy` for efficient numerical operations (particularly for geospatial calculations), and `folium` for generating rich, interactive map visualizations.

## Data Sources and Preparation

The following datasets are critical to this analysis:

1. **Index of Multiple Deprivation (IMD) 2019**: Sourced from official statistics, this dataset provides a measure of relative deprivation for small areas (specifically, Lower Layer Super Output Areas - LSOAs) across England. The key indicators utilized are the **IMD Decile** (ranking LSOAs from 1 to 10, with 1 being the most deprived) and the **Total Population (TotPop)**. This data is crucial for prioritizing areas with the greatest need and potential for impactful research.
2. **NHS Hospital and Other Data**: This dataset contains detailed information on healthcare facilities, including hospitals. For this analysis, the **Organisation Name**, **Postcode**, **Latitude**, and **Longitude** are extracted to represent the geographical distribution of healthcare access points. Accurate geospatial coordinates are essential for calculating proximity.
3. **Lower Layer Super Output Areas (LSOA) Boundaries December 2021**: This dataset provides the geographical boundaries and, importantly for this project, the centroid coordinates (mean **LAT**itude and **LONG**itude) for each LSOA in England and Wales. Merging this with the IMD data allows for spatially referenced deprivation analysis. The **LSOA21CD** is used as the primary key for joining with the IMD data (assuming compatibility with `lsoa11cd`).
4. **NIHR Infrastructure Supported Projects**: This dataset details research projects supported by the National Institute for Health and Care Research (NIHR). Relevant information includes the **Centre** name, **Centre Postcode**, **Latitude**, **Longitude**, and **Research Theme**. This data helps identify areas with existing research capacity and expertise, which can be advantageous for new site development.

**Data Loading and Initial Preparation**: The first step involves loading these datasets from their respective CSV files into pandas DataFrames. Robust error handling is implemented to manage potential `FileNotFoundError` exceptions. Subsequent preparation steps involve selecting and renaming columns for consistency and ease of use. Crucially, geographical coordinates (Latitude and Longitude) and the population column are converted to numeric types, with missing or invalid entries handled appropriately (e.g., filling missing population values with the median and dropping rows with missing coordinates).

## Methodology: A Step-by-Step Approach

The analytical process is structured into the following key stages:

1. **Data Integration**: The IMD data is merged with the LSOA geographical data using the LSOA code as the join key. This creates a comprehensive DataFrame (`lsoa_combined_df`) that includes deprivation metrics, population, and geographical coordinates for each LSOA.
2. **Geospatial Distance Calculation**: The [Haversine formula](https://en.wikipedia.org/wiki/Haversine_formula) is employed to calculate the shortest distance between two points on the surface of a sphere (a good approximation for the Earth). A dedicated Python function, `haversine_distance`, is defined for this purpose, taking pairs of latitude and longitude coordinates as input and returning the distance in kilometers.
3. **Optimized Proximity Analysis**: To efficiently calculate the distance from every LSOA to every hospital and every NIHR project, a vectorized approach using NumPy is implemented. LSOA, hospital, and NIHR project coordinates are converted into NumPy arrays. By reshaping the LSOA coordinate arrays, NumPy's broadcasting capabilities are leveraged to compute all pairwise distances simultaneously without the need for explicit Python loops. The minimum distance from each LSOA to any hospital (`Min_Dist_To_Hospital_km`) and any NIHR project (`Min_Dist_To_NIHR_Project_km`) is then determined. A proximity threshold (`NEARBY_RADIUS_KM`, set to 10 km) is defined to focus on facilities within a reasonable commuting distance.
4. **Composite Suitability Scoring**: A composite score (`Total_Research_Site_Score`) is calculated for each LSOA to provide a single metric for ranking potential research sites. This score is a weighted sum of three components:
    * **IMD Score (Weighted)**: Calculated as `(11 - IMD_Decile) * TotPop`. This awards higher points to LSOAs with lower IMD deciles (more deprived) and larger populations, effectively prioritizing areas with a greater number of people experiencing deprivation.
    * **Hospital Proximity Score**: LSOAs within the `NEARBY_RADIUS_KM` of a hospital receive points based on their distance (closer is better, with a maximum of 10 points at zero distance). LSOAs outside this radius receive 0 points. This component favors locations with good access to healthcare facilities, which can be beneficial for research recruitment and collaboration.
    * **NIHR Proximity Score**: Similar to the hospital score, LSOAs within the `NEARBY_RADIUS_KM` of an existing NIHR project receive points based on their distance (closer is better, with a maximum of 10 points at zero distance). This component recognizes the potential for synergy and collaboration with established research centers.
    The current weights for these components are set to 50% for the weighted IMD score and 25% each for hospital and NIHR proximity scores. These weights can be adjusted to reflect different strategic priorities.
5. **Ranking and Reporting**: The LSOAs are sorted in descending order based on their `Total_Research_Site_Score`. The top 10 ranked LSOAs are then displayed in a clear, professional format (markdown table), including their name, IMD Decile, population, minimum distances to hospitals and NIHR projects, and their composite score.
6. **Interactive Map Visualization**: A dynamic and interactive map is generated using the `folium` library. This map provides a rich visual representation of the analysis results:
    * LSOAs are represented by `CircleMarker` objects, with their color intensity corresponding to their `Total_Research_Site_Score` (a darker color indicates a higher suitability score). A color scale is included on the map for easy interpretation.
    * Hospitals are marked with distinct blue hospital icons. Clicking on a hospital marker reveals its name and postcode in a popup.
    * Existing NIHR Projects are marked with distinct green flask icons. Clicking on an NIHR project marker displays the centre name, research theme, and postcode.
    A layer control is added to the map, allowing users to selectively show or hide the LSOA, Hospital, and NIHR Project layers. The map is saved as an HTML file (`optimized_research_site_allocation_map.html`) for easy sharing and external viewing and is also displayed inline within a Jupyter/Colab notebook environment for immediate interactive exploration.

## Code Execution

To execute the analysis, ensure you have the necessary Python libraries installed. You can install them using pip:

```bash
pip install pandas numpy folium geopandas
```

1. Place the required CSV files (IMD 2019, NHS Hospital Data, LSOA Boundaries, NIHR Projects) in the project directory.
2. Run the script using:
   ```bash
   python research_site_allocation.py
   ```
   or execute it in a Jupyter/Colab notebook environment.

## Outcome

The script outputs a ranked list of the top 10 LSOAs based on their composite research site suitability score. This table provides key information for each top-ranked LSOA, including its name, IMD Decile, population, minimum distance to a hospital, minimum distance to an NIHR project, and the calculated total score. This ranking helps identify areas that are potentially most suitable for establishing new research sites based on the defined criteria.

## Interactive Map (optimized_research_site_allocation_map.html)

The generated `optimized_research_site_allocation_map.html` file is an interactive HTML map that visualizes the results of the analysis. You can open this file in any web browser.

The map includes:

* **LSOA Markers**: Representing the Lower Layer Super Output Areas, colored according to their composite research site suitability score. A color scale is provided on the map to interpret the scores.
* **Hospital Markers**: Indicating the locations of hospitals. Clicking on a marker will display the hospital's name and postcode.
* **NIHR Project Markers**: Showing the locations of existing NIHR infrastructure supported projects. Clicking on a marker will display the centre name, research theme, and postcode.
* **Layer Control**: Allows you to toggle the visibility of the LSOA, Hospital, and NIHR Project layers on the map.

This interactive visualization provides a geographical context for the ranked list, allowing users to explore the spatial relationships between potential research sites, areas of deprivation, and existing healthcare and research facilities.

## Author
- **Chiderah Onwumelu**  
- **Email**: chiderahonwumelu@gmail.com