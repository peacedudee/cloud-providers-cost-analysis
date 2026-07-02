# AWS Service Cost Research: Amazon Location Service

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Location Service is a fully managed geospatial service that makes it easy for developers to add maps, points of interest (geocoding), routing, tracking, and geofencing capabilities to their applications securely. It incorporates high-quality data from global location data providers Esri and HERE. Amazon Location Service is serverless, billing strictly per API request across maps, place searches, route calculations, position tracking, and geofence evaluations.

---

## 2. Billing Mechanics
Amazon Location Service pricing is calculated per 1,000 requests across 5 core functional APIs:
1.  **Maps (Tile Requests):** Billed per 1,000 map tile requests ($0.04 per 1,000 tile requests).
2.  **Places (Geocoding & Reverse Geocoding):** Billed per 1,000 place search or geocoding requests ($0.50 per 1,000 requests).
3.  **Routes (Routing & Directions):** Billed per 1,000 route calculations ($0.50 per 1,000 requests).
4.  **Geofences (Geofence Position Evaluations):** Billed per 1,000 position evaluations against geofences ($0.03 per 1,000 evaluations).
5.  **Trackers (Device Location Updates):** Billed per 1,000 location updates stored ($0.05 per 1,000 updates).

---

## 3. Key Cost Dimensions

| Location Service API | Free Tier (3 Mos) | Rate (us-east-1) | Price for 100,000 Requests |
|----------------------|-------------------|------------------|----------------------------|
| **Map Tiles** | 10,000 tiles / month | **$0.04 / 1,000 tiles** | **$4.00** |
| **Geocoding / Places**| 10,000 reqs / month | **$0.50 / 1,000 reqs** | **$50.00** |
| **Routes** | 1,000 routes / month | **$0.50 / 1,000 reqs** | **$50.00** |
| **Geofence Evals** | 10,000 evals / month | **$0.03 / 1,000 evals** | **$3.00** |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Map Tiles Rate:** $0.04 per 1,000 tile requests ($40.00 per Million tiles).
*   **Geocoding / Places Rate:** $0.50 per 1,000 requests ($500.00 per Million requests).
*   **Routing Rate:** $0.50 per 1,000 route calculations.
*   **Geofence Evaluation Rate:** $0.03 per 1,000 evaluations ($30.00 per Million evaluations).

---

## 5. AWS Free Tier Coverage
*   **Amazon Location Service Free Tier:** Includes a 3-month free tier of **10,000 map tiles**, 10,000 place requests, 1,000 route calculations, 10,000 geofence evaluations, and 10,000 tracker position updates per month.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Evaluating Geofences for Stationary Devices:** Sending continuous GPS ping updates to Geofence Evaluator every second for parked vehicles or stationary IoT devices ($0.03/1k evals = $77.76/mo per 1,000 stationary pings).

---

## 7. Actionable Cost Optimization Strategies
1.  **Filter Stationary Device Location Pings:**
    *   Implement client-side or Lambda filtering logic to suppress location updates if the device has moved less than 10 meters.
    *   **The Savings:** Reduces geofence position evaluation billing ($0.03/1k) by **80–90%**.
2.  **Enable Client-Side Map Tile Caching:** Configure mobile (iOS/Android SDK) and web map views to cache map tiles locally during user sessions.
    *   **The Savings:** Slashes tile request volumes ($0.04/1k) by **50%**.
3.  **Cache Common Geocoding Address Searches:** Cache geocoded latitude/longitude coordinates locally in Amazon ElastiCache or DynamoDB to avoid repeating $0.50/1k geocoding API queries for frequently searched addresses.
