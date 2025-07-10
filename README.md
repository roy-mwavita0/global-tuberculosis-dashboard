# 🌍 Global Tuberculosis (TB) Dashboard

An interactive R Shiny dashboard to analyze and visualize Tuberculosis (TB) trends globally, using population-normalized metrics and WHO data. The dashboard supports Monitoring and Evaluation (M&E) efforts in health programming by offering insights into TB incidence, mortality, and TB/HIV co-infection across countries and over time.

---

## 🔗 Live Demo

[Access the Dashboard Here](https://roymwavita.shinyapps.io/who_tb_data/)

---

## 📦 Features
 ### 1.📊 Interactive plots for TB incidence and mortality (per 100,000 population)
 
  
![Screenshot 2025-07-10 111246](https://github.com/user-attachments/assets/090eb038-3856-4358-816b-815b7c162909)

### 2.🌐 Choropleth map showing average TB incidence by country


![Screenshot 2025-07-10 111140](https://github.com/user-attachments/assets/aebe0af9-1d62-4465-8d91-6961fb9044ad)

### 3.🔎 Country and year filters (multi-country selection supported)


  ![Screenshot 2025-07-10 111159](https://github.com/user-attachments/assets/a1295ba2-9bcf-4607-827e-9d7d33fc0368)

### 4.📈 Yearly trend lines for multiple countries

  
  ![Screenshot 2025-07-10 111230](https://github.com/user-attachments/assets/a9ad3eda-3fbb-4cc6-99ee-c3de6ef33b37)

### 5.🧬 TB/HIV co-infection metrics and visualizations**


---

## 📁 Tabs Overview

- **About:** Overview of the dashboard’s purpose and data source  
- **Overview:** Value boxes and line chart for TB metrics  
- **Trends:** TB incidence trends over time per country  
- **TB/HIV Co-Infection:** Metrics and trends of TB among people living with HIV  
- **Map:** World map visualizing TB incidence rates

---

## 🧠 Smart Features

- All countries selected by default for global insights  
- Dynamically updates charts and maps based on input  
- Handles "All Countries" logic within server logic  
- Uses WHO data directly from the public API endpoint

---

## 📊 Data Source

Data is sourced from the **[WHO Global Tuberculosis Database](https://extranet.who.int/tme/)** via the `estimates` dataset.

---

## 🛠️ How the Dashboard Was Built (Step-by-Step with Code Highlights)

### 1. Load Required Libraries

I used `pacman::p_load()` to simplify package management:

```r
pacman::p_load(
  tidyverse, janitor, tidylog, shinydashboard, shiny, leaflet, sf,
  rnaturalearth, rnaturalearthdata
)
```

This loads all the necessary packages for:

- Data wrangling (tidyverse, janitor)

- UI creation (shiny, shinydashboard)

- Geospatial mapping (sf, leaflet, rnaturalearth)

### 2: 🔄 Load and Clean the WHO TB Data
The dataset was pulled directly from the WHO endpoint:
```r
tb_data <- read_csv("https://extranet.who.int/tme/generateCSV.asp?ds=estimates")
```
I then selected and renamed relevant columns for clarity:
```r
tb_clean <- tb_data %>%
  filter(year >= 2010) %>%
  select(
    country, year, e_pop_num, e_inc_num, e_mort_exc_tbhiv_num,
    e_inc_tbhiv_num, e_mort_tbhiv_num
  ) %>%
  rename(
    population = e_pop_num,
    tb_incidence = e_inc_num,
    tb_mortality = e_mort_exc_tbhiv_num,
    tbhiv_incidence = e_inc_tbhiv_num,
    tbhiv_mortality = e_mort_tbhiv_num
  ) %>%
  drop_na()
```
This ensures clean and focused data for analysis.

### 3. 🗺️ Prepare Map Data
Using the rnaturalearth and sf libraries, I mapped TB incidence rates:
```r
world_sf <- ne_countries(scale = "medium", returnclass = "sf")

tb_map_data <- tb_latest %>%
  group_by(country) %>%
  summarise(avg_incidence = mean(tb_incidence / population * 100000, na.rm = TRUE))

map_data <- world_sf %>%
  left_join(tb_map_data, by = c("name" = "country"))
```
This calculates per-100k incidence rates and joins them to spatial polygons.

### 4. 🧱 UI Design Using shinydashboard
I structured the UI into five tabs using dashboardSidebar() and tabItems():
- About
- Overview (value boxes + summary plot)
- Trends (line chart)
- TB/HIV Co-Infection (cases, deaths, trends)
- Map (choropleth of incidence)

Key inputs:
```r
selectInput("country", ..., multiple = TRUE)
sliderInput("year", ..., value = latest_year)
```
These allow real-time country/year filtering.

### 5. 🔄 Server Logic
**a. Filtering Data Reactively**
```r
tb_filtered <- reactive({
  if ("All Countries" %in% input$country) {
    tb_clean %>% filter(year == input$year)
  } else {
    tb_clean %>% filter(country %in% input$country, year == input$year)
  }
})
```
This enables both single- and multi-country selection dynamically.

**b. Value Boxes**
```r
output$latestIncidence <- renderValueBox({
  val <- tb_filtered() %>%
    summarise(val = sum(tb_incidence / population * 100000)) %>%
    pull(val)
  valueBox(round(val, 1), "Incidence per 100k", ...)
})
```
This summarizes population-normalized incidence values by year and country.

**c. Trends Over Time**
```r
output$trendPlot <- renderPlot({
  data %>%
    ggplot(aes(x = year, y = tb_incidence / population * 100000, color = country)) +
    geom_line()
})
```
I used ggplot2 for clean, color-coded trendlines.

**d. TB/HIV Specific Charts**
```r
output$tbhivIncidencePlot <- renderPlot({
  tb_clean %>%
    filter(country %in% input$country) %>%
    ggplot(aes(x = year, y = tbhiv_incidence, color = country)) +
    geom_line()
})
```
These plots help explore HIV co-infection patterns over time.

**e. Map Rendering**
```r
output$tbMap <- renderLeaflet({
  leaflet(map_data) %>%
    addPolygons(
      fillColor = ~colorNumeric("Reds", avg_incidence)(avg_incidence),
      label = ~paste0(name, ": ", round(avg_incidence, 1), " per 100k")
    )
})
```
This interactive map uses leaflet and includes labels and color gradients.

## 📊 Summary of Metrics Used

| **Metric**         | **Description**                         |
|--------------------|-----------------------------------------|
| `tb_incidence`     | Estimated new TB cases                  |
| `tb_mortality`     | TB deaths (excluding HIV)               |
| `tbhiv_incidence`  | TB cases among people living with HIV   |
| `tbhiv_mortality`  | TB deaths among people living with HIV  |
| `population`       | Population estimate (used for rate per 100k normalization) |

## 👤 Author

**Roy Lennic Mwavita**  
Monitoring & Evaluation Specialist | Data Analyst 
📧 [lennicroy@gmail.com](mailto:lennicroy@gmail.com)  
📞 +254 794 395 145  
🔗 [LinkedIn](https://www.linkedin.com/in/roy-mwavita-495b50220/)  

