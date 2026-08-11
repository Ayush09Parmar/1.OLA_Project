# Ola Ride Booking Analysis
Dashboard Link: https://app.powerbi.com/links/qlZm37W7pi?ctid=56c1d497-700b-49cf-8f8d-3dd6b20d522f&pbi_source=linkShare

## Problem Statement

I wanted to dig into how Ola's bookings actually perform once you look past the "total rides" number. A lot of bookings never turn into a completed ride, and I was curious why: is it mostly customers backing out, drivers cancelling, or drivers just not being found at all? So I picked up a ~100,000 row bookings dataset and built this out to find where things break down and why.

Along the way I also wanted to see how revenue splits across payment methods and vehicle types, and whether ratings actually differ much between something like an Auto and a Prime Sedan, or if that's not really where the story is.

Turns out only 62.1% of the 79,843 bookings I looked at were actually successful. The rest is split pretty unevenly - drivers cancelled 17.9% of the time, customers cancelled 10.2%, and 9.8% ended up as "driver not found." That's a big enough chunk that it felt worth breaking down further, which is basically what the Cancellation page is for.

## Steps Followed

1. Loaded the bookings dataset into Power BI Desktop.

2. Went into Power Query Editor and checked column distribution, column quality, and column profile for each field, mainly to catch missing or weird values before building anything on top of them.

3. Cleaned up the cancellation-related columns - customer-side and driver-side cancellation reasons were sitting in separate fields, so I made sure they lined up properly before charting them.

4. Set up five pages: Overall, Vehicle Type, Revenue, Cancellation, and Ratings, with nav buttons on the left so you can click between them.

5. Added a date slicer on every page so the whole report can be filtered down to a specific window.

6. #### Overall page: a pie chart for booking status, a line chart for ride volume over time, and a couple of card visuals up top for total bookings.(https://github.com/user-attachments/assets/c6c2bfd8-2a73-4065-9c3d-c28a9c01d3fe)

7. #### Vehicle Type page: a table with total booking value, successful booking value, average distance, and total distance, one row per vehicle type (Prime Sedan, Prime SUV, Prime Plus, Mini, Auto, Bike, E-Bike).(https://github.com/user-attachments/assets/6884b2b3-ced1-475c-a152-d21ceb1f5c87)

8. #### Revenue page: bar charts for revenue by payment method (both as totals and day by day), plus a small table listing the top 5 customers by total booking value.( https://github.com/user-attachments/assets/296a593d-7f1b-4ef4-9cbc-2f1b42af5773)

9. #### Cancellation page: built a Cancellation Rate measure, then two pie charts - one for why customers cancel, one for why drivers cancel.(https://github.com/user-attachments/assets/7b94af7b-e359-48d1-af43-7772222ed10d)

10. #### Ratings page: cards comparing average driver and customer ratings across all seven vehicle types.(https://github.com/user-attachments/assets/360daf4e-61c1-479e-9d8a-8e48e4eaf986)

11. Separately from the dashboard, I wrote 10 SQL queries against the same dataset to answer some more specific questions (full list in [`queries.sql`](./queries.sql)). A couple of examples:

```sql
-- Average ride distance per vehicle type
CREATE VIEW Ride_Distance_For_Each_Vehicle AS
SELECT Vehicle_Type, AVG(Ride_Distance) AS avg_distance
FROM bookings
GROUP BY Vehicle_Type;

-- Top 5 customers by number of rides
CREATE VIEW Top_5_Customers AS
SELECT Customer_ID, COUNT(Booking_ID) AS total_rides
FROM bookings
GROUP BY Customer_ID
ORDER BY total_rides DESC
LIMIT 5;
```

The rest cover things like successful bookings, cancellation counts by reason, max/min driver ratings for Prime Sedan, UPI-only payments, average customer rating by vehicle type, total value of successful rides, and incomplete rides with their reasons.

## Insights

**Booking status** (out of 79,843 total bookings)
- Success - 49.58K (62.1%)
- Cancelled by driver - 14.32K (17.94%)
- Cancelled by customer - 8.11K (10.15%)
- Driver not found - 7.83K (9.81%)

So more than a third of bookings don't end in a completed ride, which was honestly higher than I expected going in.

**Why customers cancel**
- Driver not moving towards pickup - 30.38%
- Driver asked to cancel - 25.23%
- Change of plans - 19.99%
- AC not working - 14.69%
- Wrong address - 9.71%

**Why drivers cancel**
- Personal & car related issue - 35.58%
- Customer related issue - 29.32%
- Customer was coughing / illness related - 19.84%
- More than permitted passengers - 15.25%

The interesting part here is that the top reason on the driver side (personal/car issues) is something Ola could actually influence with better driver support or vehicle checks, whereas a lot of the customer-side reasons feel more situational.

**Cancellation rate:** 28.09% overall (30,259 out of 79,843), split fairly evenly between the two sides above.

**Revenue by payment method**
- Cash - around 15M
- UPI - around 11M
- Credit + debit card combined - under 1M

Cash and UPI are clearly doing all the work here, card payments barely register.

**Vehicle type**
- Every vehicle type except Auto averages roughly 25 km per ride (24.86-25.10 km).
- Auto is the outlier at around 10 km average, which makes sense if people are mostly using it for short, local trips rather than longer journeys.
- Booking value is close to even across vehicle types (~4.01M each), though Prime Sedan converts the highest share of that into successful rides, while Prime SUV and Mini convert the least.

**Ratings**
- Driver ratings sit between 3.98 and 4.01 across every vehicle type.
- Customer ratings are in the same narrow band, 3.98 to 4.01.

Ratings basically don't move much by vehicle type, so if there's a service quality story here, it's probably not about which vehicle type someone books.

