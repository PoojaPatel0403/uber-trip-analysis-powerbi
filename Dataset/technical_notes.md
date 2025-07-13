# 🧠 Technical Notes – Uber Trip Analysis

This document summarizes the DAX measures and data modeling strategies used in the Uber Trip Analysis project. These formulas power the KPIs and interactivity in the Overview, Time Analysis, and Details dashboards.

---

## 📊 KPI Calculations – Overview Dashboard

Total Bookings  
Total Bookings = COUNT('Trip Details'[Trip ID])  
Counts all completed trip records.

Total Booking Amount  
Total Booking Amount = SUM('Trip Details'[fare_amount]) + SUM('Trip Details'[Surge Fee])  
Calculates total trip revenue including surge pricing.

Average Booking Amount  
Avg Booking Amount = DIVIDE('Trip Details'[Total Booking Amount], [Total Bookings], BLANK())  
Calculates average revenue per trip with safe divide.

Total Trip Distance  
Total Trip Distance =  
VAR TotalMiles = SUM('Trip Details'[trip_distance]) / 1000  
RETURN FORMAT(TotalMiles, "0") & "K miles"  
Sums all trip distances and formats the result in thousands of miles.

Average Trip Distance  
Avg Trip Distance =  
VAR AvgMiles = ROUND(AVERAGE('Trip Details'[trip_distance]), 0)  
RETURN AvgMiles & " miles"  
Rounds the average distance to nearest whole number with unit formatting.

Average Trip Time  
Avg Trip Time =  
VAR AvgTime = AVERAGEX('Trip Details', DATEDIFF('Trip Details'[Pickup Time], 'Trip Details'[Drop Off Time], MINUTE))  
RETURN FORMAT(AvgTime, "0") & " min"  
Calculates the average trip duration using DATEDIFF.

---

## 📍 Location Analysis Measures

Farthest Trip  
Farthest Trip =  
VAR MaxDistance = MAX('Trip Details'[trip_distance])  
VAR PickupLocation = LOOKUPVALUE('Location Table'[Location], 'Location Table'[LocationID], CALCULATE(SELECTEDVALUE('Trip Details'[PULocationID]), 'Trip Details'[trip_distance] = MaxDistance))  
VAR DropoffLocation = LOOKUPVALUE('Location Table'[Location], 'Location Table'[LocationID], CALCULATE(SELECTEDVALUE('Trip Details'[DOLocationID]), 'Trip Details'[trip_distance] = MaxDistance))  
RETURN "Pickup: " & PickupLocation & " → Drop-off: " & DropoffLocation & " (" & FORMAT(MaxDistance, "0.0") & " miles)"  
Finds the trip with the longest distance and formats a descriptive string.

Most Frequent Pickup Point  
Most Frequent Pickup Point =  
VAR PickPoint = TOPN(1, SUMMARIZE('Trip Details', 'Location Table'[Location], "Pickup Point", COUNT('Trip Details'[Trip ID])), [Pickup Point], DESC)  
RETURN CONCATENATEX(PickPoint, 'Location Table'[Location], ", ")  
Returns the pickup location that appears most often.

Most Frequent Drop-off Point  
Most Frequent Dropoff Point =  
VAR DropOffCounts = ADDCOLUMNS(SUMMARIZE('Trip Details', 'Location Table'[Location]), "DropOffCount", CALCULATE(COUNT('Trip Details'[Trip ID]), USERELATIONSHIP('Trip Details'[DOLocationID], 'Location Table'[LocationID])))  
VAR RankedDropoffs = ADDCOLUMNS(DropOffCounts, "Rank", RANKX(DropOffCounts, [DropOffCount], , DESC, DENSE))  
VAR TopDropoff = FILTER(RankedDropoffs, [Rank] = 1)  
RETURN CONCATENATEX(TopDropoff, 'Location Table'[Location], ", ")  
Calculates most common drop-off using an inactive relationship.

---

## ⏰ Time-Based Calculations

Pickup Date  
Pickup Date = DATE(YEAR('Trip Details'[Pickup Time]), MONTH('Trip Details'[Pickup Time]), DAY('Trip Details'[Pickup Time]))  
Extracts only the date part for daily trend analysis.

Pickup Hour  
Pickup Hour = HOUR('Trip Details'[Pickup Time])  
Returns the hour portion (0–23) for hourly distribution visuals.

---

## 🔗 Data Model Structure

Fact Table: Trip Details  
Dimension Table: Location Table

Relationships:  
- Pickup Location: One-to-many — Location Table[LocationID] → Trip Details[PULocationID]  
- Drop-off Location: Inactive relationship — Location Table[LocationID] → Trip Details[DOLocationID] (activated using USERELATIONSHIP in DAX)

---

## 📌 Summary

These DAX measures and modeling strategies support dynamic visuals, slicer interactions, and location-based insights in the Uber Trip Analysis dashboard. Together with slicers, bookmarks, and drill-throughs, these measures form the backbone of the analytical insights delivered in the report.
```
