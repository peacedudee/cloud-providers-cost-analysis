# AWS Service Cost Research: Amazon Location Service

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Location Service is a fully managed geospatial service that makes it easy for developers to add maps, points of interest (geocoding & place search), routing, tracking, and geofencing capabilities to their applications securely. It incorporates high-quality data from global location data providers Esri and HERE, alongside open data options. Amazon Location Service is serverless, billing strictly per 1,000 API requests across maps, place searches, route calculations, position tracking, and geofence evaluations.

---

## 2. Billing Mechanics
Amazon Location Service pricing is calculated per 1,000 requests across 5 core functional APIs:
1. **Maps (Tile & Static Requests):** Billed per 1,000 map tile or static map image requests ($0.03 to $0.50 per 1,000 requests).
2. **Places (Geocoding & Reverse Geocoding):** Billed per 1,000 place search or geocoding requests across Label, Core, Advanced, and Stored tiers ($0.20 to $4.00 per 1,000 requests).
3. **Routes (Routing & Directions):** Billed per 1,000 route calculations across Core and Advanced modes ($0.50 to $1.50 per 1,000 requests).
4. **Geofences (Geofence Position Evaluations):** Billed per 1,000 position evaluations against geofences ($0.03 per 1,000 evaluations).
5. **Trackers (Device Location Updates):** Billed per 1,000 location updates stored ($0.05 per 1,000 updates).

---

## 3. Key Cost Dimensions

### A. Maps API Rates (us-east-1)
* **Standard Map Tiles:** **$0.04 per 1,000 tiles** ($40.00 per Million tiles).
* **Open Data Map Tiles:** **$0.03 per 1,000 tiles** ($30.00 per Million tiles).
* **Static Map Images:** **$0.50 per 1,000 requests**.

### B. Places API Rates (Geocoding & Place Lookups)
* **Places Label (ID and label only):** **$0.20 per 1,000 requests**.
* **Places Core (Addresses, Categories):** **$0.50 per 1,000 requests** ($500.00 per Million).
* **Places Advanced (Business hours, contact info):** **$1.50 per 1,000 requests**.
* **Places Stored (Caching/Saving results):** **$4.00 per 1,000 requests** (required if storing geocoded coordinates in databases).

### C. Routes API Rates
* **Routes Core (Car, truck, pedestrian routing):** **$0.50 per 1,000 requests**.
* **Routes Advanced (Specialized modes like scooter/commercial freight):** **$1.50 per 1,000 requests**.

### D. Trackers & Geofences
* **Geofence Position Evaluations:** **$0.03 per 1,000 evaluations**.
* **Tracker Location Updates:** **$0.05 per 1,000 updates**.

---

## 4. Detailed Pricing Rates (us-east-1)

| Location Service Feature | API Sub-Tier | Rate (us-east-1) | Price for 100,000 Requests |
|--------------------------|--------------|------------------|----------------------------|
| **Maps** | Open Data Tiles | **$0.03 / 1,000 tiles** | **$3.00** |
| **Maps** | Standard Tiles | **$0.04 / 1,000 tiles** | **$4.00** |
| **Maps** | Static Map Image | **$0.50 / 1,000 reqs** | **$50.00** |
| **Places** | Label Only | **$0.20 / 1,000 reqs** | **$20.00** |
| **Places** | Core Geocoding | **$0.50 / 1,000 reqs** | **$50.00** |
| **Places** | Advanced Place Info | **$1.50 / 1,000 reqs** | **$150.00** |
| **Places** | Stored / Cached | **$4.00 / 1,000 reqs** | **$400.00** |
| **Routes** | Core Routing | **$0.50 / 1,000 reqs** | **$50.00** |
| **Geofences** | Position Evals | **$0.03 / 1,000 evals** | **$3.00** |
| **Trackers** | Location Updates | **$0.05 / 1,000 updates**| **$5.00** |

*Note: Custom volume discounts apply for monthly bills exceeding $5,000.*

---

## 5. AWS Free Tier Coverage
* **Amazon Location Service Free Tier:** Includes a 3-month free tier of **10,000 map tiles**, 10,000 place requests, 1,000 route calculations, 10,000 geofence evaluations, and 10,000 tracker position updates per month.

---

## 6. Common Cost Hotspots & Pitfalls
* **Evaluating Geofences for Stationary Devices:** Sending continuous GPS ping updates to Geofence Evaluator every second for parked vehicles or stationary IoT devices ($0.03/1k evals = $77.76/mo per 1,000 stationary pings).
* **Using Places Stored Tier Unnecessarily:** Selecting the Stored Places API ($4.00/1k) when coordinates are only rendered transiently in user browser sessions ($0.50/1k).

---

## 7. Actionable Cost Optimization Strategies
1. **Filter Stationary Device Location Pings:**
   * Implement client-side or Lambda filtering logic to suppress location updates if the device has moved less than 10 meters.
   * **The Savings:** Reduces geofence position evaluation billing ($0.03/1k) by **80–90%**.
2. **Enable Client-Side Map Tile Caching:** Configure mobile (iOS/Android SDK) and web map views to cache map tiles locally during user sessions.
   * **The Savings:** Slashes tile request volumes ($0.04/1k) by **50%**.
3. **Use Places Core ($0.50/1k) vs Stored ($4.00/1k) Appropriately:**
   * Only use the **Places Stored** tier ($4.00/1k) when your application terms explicitly require storing geocoded lat/long coordinates persistently in a database (e.g., customer address records).
   * For dynamic, real-time map search dropdowns, use **Places Core ($0.50/1k)** to save **87.5%**.
