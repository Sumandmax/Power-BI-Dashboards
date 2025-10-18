// DAX Measures for EV Dashboard

Total EV Sales = SUM(EV_Sales[Units_Sold])

Total Charging Stations = COUNTROWS(Charging_Stations)

Average Range per EV = AVERAGE(EV_Specs[Range_km])

EV Growth Rate (%) = 
VAR PrevYear = CALCULATE([Total EV Sales], DATEADD(EV_Sales[Date], -1, YEAR))
RETURN
DIVIDE([Total EV Sales] - PrevYear, PrevYear, 0) * 100

Top Selling Model = 
CALCULATE(
    FIRSTNONBLANK(EV_Sales[Model], 1),
    TOPN(1, EV_Sales, EV_Sales[Units_Sold], DESC)
)

Charging Density = 
DIVIDE([Total Charging Stations], DISTINCTCOUNT(EV_Sales[Region]), 0)
// Power Query (M Code) for Data Loading and Cleaning

let
    Source = Excel.Workbook(File.Contents("EV_Data.xlsx"), null, true),
    EV_Sales_Sheet = Source{[Item="EV_Sales",Kind="Sheet"]}[Data],
    Charging_Stations_Sheet = Source{[Item="Charging_Stations",Kind="Sheet"]}[Data],
    EV_Specs_Sheet = Source{[Item="EV_Specs",Kind="Sheet"]}[Data],

    EV_Sales = Table.TransformColumnTypes(
        Table.PromoteHeaders(EV_Sales_Sheet, [PromoteAllScalars=true]),
        {{"Date", type date}, {"Units_Sold", Int64.Type}, {"Region", type text}, {"Model", type text}}
    ),

    Charging_Stations = Table.TransformColumnTypes(
        Table.PromoteHeaders(Charging_Stations_Sheet, [PromoteAllScalars=true]),
        {{"Station_ID", type text}, {"Region", type text}, {"Latitude", type number}, {"Longitude", type number}}
    ),

    EV_Specs = Table.TransformColumnTypes(
        Table.PromoteHeaders(EV_Specs_Sheet, [PromoteAllScalars=true]),
        {{"Model", type text}, {"Range_km", type number}, {"Battery_kWh", type number}}
    )
in
    [EV_Sales=EV_Sales, Charging_Stations=Charging_Stations, EV_Specs=EV_Specs]
// Visual Layout & Styling Guide

Theme Colors:
- Background: #F5F5F5 (light gray)
- EV Sales: #00BFFF (electric blue)
- Charging Stations: #32CD32 (lime green)
- Growth Rate: #FFA500 (orange)
- Cards: White with shadow

Visuals:
- Bar Chart: EV Sales by Region
- Map: Charging Station Locations (Latitude/Longitude)
- Line Chart: EV Growth Rate over Time
- Gauge: Average Range per EV
- Cards: Total EV Sales, Total Charging Stations, Top Selling Model
- Table: EV Specs with conditional formatting on Range

Interactions:
- Slicers for Region, Model, and Year
- Tooltip with Charging Density
- Bookmark for “Top Models” view

