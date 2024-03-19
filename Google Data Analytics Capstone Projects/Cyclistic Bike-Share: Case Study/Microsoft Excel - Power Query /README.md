## 🔄 The 'Process' Phase

### Tool
>*Microsoft Excel and Power Query*
+ **Microsoft Excel** and **Power Query** were chosen for this analysis due to their widespread accessibility, robust data manipulation capabilities, and seamless integration with external data sources.
+ Excel Functions provide powerful tools for performing various calculations, aggregations, and data manipulations, enhancing the analytical capabilities of the platform.
+ Power Query complements Excel's functionalities by offering advanced data shaping and transformation capabilities. Leveraging Power Query M Language, it simplifies tasks such as data cleaning, integration, and transformation, streamlining the data preparation process.
+ Additionally, Excel's visualization tools enable effective summarization and presentation of analysis results, facilitating insightful data visualization for stakeholders.

### Data Preparation
+ Imported the 12 CSV files into Excel, converting them to XLSX for enhanced formatting and analysis.
+ Conducted an initial data review.

### Data Exploration
+ Generated table charts noting record counts, duplicates, and null/blank values, and identified specific stations and IDs related to testing, warehouses, charging stations, and repairs.

#### Counts of Records, Duplicates, and Null/Blank Values
>Calculated using `=COLUMNS`, `=ROWS`, `=COUNTIF`, and `=COUNTBLANK` functions.

![1](https://github.com/chaanalyst/Portfolio-Projects/assets/154933301/2ae1b0b0-96aa-4533-a7df-fc56d724f48d)

#### Count of Stations and IDs for Testing, Warehouses, Charging, and Repairs
>Calculated using `=COUNTIF` functions.

![2](https://github.com/chaanalyst/Portfolio-Projects/assets/154933301/841785a3-55fd-4418-b76f-1d7aec974b51)

### Summary of Initial Review
+ Each file consists of 13 columns.
+ Total number of rows across all files is 5,719,877.
+ Total count of null/blank values is 3,624,089.
+ Missing data primarily occurs in columns:
    * start_station_name
    * start_station_id
    * end_station_name
    * end_station_id
    * end_lat
    * end_lng
+ Following further exploration, a decision will be made regarding whether to keep or drop missing data.

### Data Consolidation
+ Using Power Query to combine all 12 XLSX files into one file, following the initial review.

```ruby
= Table.TransformColumnTypes(#"Expanded Table Column1",{{"ride_id", type text}, {"rideable_type", type text}, {"started_at", type datetime}, {"ended_at", type datetime}, {"start_station_name", type text}, {"start_station_id", type text}, {"end_station_name", type text}, {"end_station_id", type text}, {"start_lat", type number}, {"start_lng", type number}, {"end_lat", type number}, {"end_lng", type number}, {"member_casual", type text}})
```

### Data Transformation and Cleaning

#### Renaming a Column
+ `member_casual` to `user_type`.
```ruby
= Table.RenameColumns(#"Changed Type",{{"member_casual", "user_type"}})
```

#### Removing Duplicates
+ Removing duplicates from column `ride_id`.
```ruby
= Table.Distinct(#"Rounded Off", {"ride_id"})
```

#### Ensure Consistency 
+ Rounding up `start_lat`, `start_lng`, `end_lat`, and `end_lng` by 2 decmial. 
```ruby
= Table.TransformColumns(#"Replaced Value",{{"start_lat", each Number.Round(_, 2), type number}, {"start_lng", each Number.Round(_, 2), type number}, {"end_lat", each Number.Round(_, 2), type number}, {"end_lng", each Number.Round(_, 2), type number}})
```

#### Adding Columns
+ Duplicating the column `started_at` (i.e. 1/21/2023 8:16:33 PM) and splitting column into six different column.
+ `month` (i.e. **1**/21/2023 8:16:33 PM)
```ruby
= Table.TransformColumns(#"Duplicated Column6", {{"started_at - Copy", each Date.MonthName(_), type text}})
```
+ `day` (i.e. 1/**21**/2023 8:16:33 PM)
```ruby
= Table.TransformColumns(#"Renamed Columns1",{{"started_at - Copy - Copy", Date.Day, Int64.Type}})
```
+ `year` (i.e. 1/21/**2023** 8:16:33 PM)
```ruby
= Table.TransformColumns(#"Renamed Columns2",{{"year", Date.Year, Int64.Type}})
```
+ `day_of_week` (i.e. 1/**21**/2023 8:16:33 PM)
```ruby
= Table.TransformColumns(#"Renamed Columns3", {{"day_of_week", each Date.DayOfWeekName(_), type text}})
```
+ `hour` (i.e. 1/21/2023 **8**:16:33 PM)
```ruby
= Table.TransformColumns(#"Extracted Day Name",{{"started_at - Copy - Copy.1 - Copy.1", Time.Hour, Int64.Type}})
```
+ `quarter`
```ruby
= Table.TransformColumns(#"Duplicated Column5",{{"started_at - Copy", Date.QuarterOfYear, Int64.Type}})
```

#### Adding a Custom Column `ride_length_min`
+ Add a custom column called `ride_length_min` measuring the difference between `ended_at` and `started_at`.
```ruby
= Table.AddColumn(#"Renamed Columns5", "Custom", each [ended_at] - [started_at])
```
+ Converting the `ride_length_min` measurement from hours to minutes to get a more accurate reading.
```ruby
= Table.TransformColumns(#"Renamed Columns6",{{"ride_length_min", Duration.TotalMinutes, type number}})
```
+ Rounding up `ride_length_min` by 2 decimal.
```ruby
= Table.TransformColumns(#"Calculated Total Minutes",{{"ride_length_min", each Number.Round(_, 2), type number}})
```

#### Adding Custom Column `ride_distance`
+ Add a custom column called `ride_distance` measuring the distance between the start and end points and converting from kilometers to miles.
```ruby
= Table.AddColumn(#"Renamed Columns7", "Custom", each let
    startLat = [start_lat],
    startLng = [start_lng],
    endLat = [end_lat],
    endLng = [end_lng],
    earthRadiusMiles = 3958.8, // Earth radius in miles
    toRadians = (angle) => angle * (Number.PI / 180),
    dLat = toRadians(endLat - startLat),
    dLon = toRadians(endLng - startLng),
    a = Number.Sin(dLat / 2) * Number.Sin(dLat / 2) + Number.Cos(toRadians(startLat)) * Number.Cos(toRadians(endLat)) * Number.Sin(dLon / 2) * Number.Sin(dLon / 2),
    c = 2 * Number.Atan2(Number.Sqrt(a), Number.Sqrt(1 - a)),
    distanceMiles = earthRadiusMiles * c
in
    distanceMiles)
```
+ Rounding up `ride_distance` by 2 decimal.
```ruby
= Table.AddColumn(#"Changed Type3", "Round", each Number.Round([ride_distance], 2), type number)
```

#### Data Cleaning
+ Cleaning columns by trimming any leading and trailing spaces.
```ruby
= Table.TransformColumns(Table.TransformColumnTypes(#"Filtered Rows", {{"started_at", type text}, {"ended_at", type text}, {"start_lat", type text}, {"start_lng", type text}, {"end_lat", type text}, {"end_lng", type text}, {"month", type text}, {"day", type text}, {"year", type text}, {"hour", type text}, {"quarter", type text}, {"ride_length_min", type text}}, "en-US"),{{"ride_id", Text.Trim, type text}, {"rideable_type", Text.Trim, type text}, {"started_at", Text.Trim, type text}, {"ended_at", Text.Trim, type text}, {"start_station_name", Text.Trim, type text}, {"start_station_id", Text.Trim, type text}, {"end_station_name", Text.Trim, type text}, {"end_station_id", Text.Trim, type text}, {"start_lat", Text.Trim, type text}, {"start_lng", Text.Trim, type text}, {"end_lat", Text.Trim, type text}, {"end_lng", Text.Trim, type text}, {"user_type", Text.Trim, type text}, {"month", Text.Trim, type text}, {"day", Text.Trim, type text}, {"year", Text.Trim, type text}, {"day_of_week", Text.Trim, type text}, {"hour", Text.Trim, type text}, {"quarter", Text.Trim, type text}, {"ride_length_min", Text.Trim, type text}})
```
+ Cleaning columns by removing any non-printable characters.
```ruby
= Table.TransformColumns(#"Trimmed Text",{{"ride_id", Text.Clean, type text}, {"rideable_type", Text.Clean, type text}, {"started_at", Text.Clean, type text}, {"ended_at", Text.Clean, type text}, {"start_station_name", Text.Clean, type text}, {"start_station_id", Text.Clean, type text}, {"end_station_name", Text.Clean, type text}, {"end_station_id", Text.Clean, type text}, {"start_lat", Text.Clean, type text}, {"start_lng", Text.Clean, type text}, {"end_lat", Text.Clean, type text}, {"end_lng", Text.Clean, type text}, {"user_type", Text.Clean, type text}, {"month", Text.Clean, type text}, {"day", Text.Clean, type text}, {"year", Text.Clean, type text}, {"day_of_week", Text.Clean, type text}, {"hour", Text.Clean, type text}, {"quarter", Text.Clean, type text}, {"ride_length_min", Text.Clean, type text}})
```
+ Filtering out outliers from the `ride_length_min` column. Removing any ride duration that is less or equal to 1 ( 60 seconds) or greater or equal to 1440 (24 hours).
```ruby
= Table.SelectRows(#"Rounded Off1", each [ride_length_min] >= 1 and [ride_length_min] <= 1440)
```

### Key Tasks
- [ ]  Check the data for errors.
- [ ]  Choose your tools.
- [ ]  Transform the data so you can work with it effectively.
- [ ]  Document the cleaning process.

### Deliverable 
- [ ]  A summary of your analysis.