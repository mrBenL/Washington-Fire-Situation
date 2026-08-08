# Washington Fire Situational Dashboard

A single-page situational map of Washington wildfire activity.
**Live: <https://mrbenl.github.io/Washington-Fire-Situation/>**

One `index.html`. No build step, no server, no API keys — it reads public
ArcGIS feeds directly from the browser and refreshes hourly.

> **This is an unofficial presentation layer.** It does not produce data. Every
> number on the page is computed in the browser from the public feeds listed
> below, and is **not** a figure published by WA DNR or NIFC. Reporting lags
> real conditions. Do not use it for emergency decisions — see
> [WA DNR](https://dnr.wa.gov/wildfire-resources/current-wildfire-incident-information),
> [InciWeb](https://inciweb.wildfire.gov/), or
> [WA Emergency Alerts](https://mil.wa.gov/alerts).

---

## Where the data comes from

Two feeds, each doing the job it's actually good at.

### 1. Current incidents — NIFC

```
https://services9.arcgis.com/RHVPKKiFTONKtxq3/arcgis/rest/services/USA_Wildfires_v1/FeatureServer/0
where: POOState = 'US-WA'
```

The National Interagency Fire Center's `Current_Incidents` layer — the
interagency roster, covering **all** jurisdictions in Washington: federal,
state, tribal, local. This drives the active statistics and the ember markers.

It is used because DNR's own layer cannot see past DNR's jurisdiction. A fire
on National Park land — Grand Park 2, on the Rainier side — appears here and
nowhere in the DNR feed. It also carries `PercentContained`, which the DNR
layer has no field for at all.

### 2. Season to date — WA DNR

```
https://gis.dnr.wa.gov/site3/rest/services/Public_Wildfire/WADNR_PUBLIC_WD_WildFire_Data/MapServer/1
where: FIREGCAUSE_LABEL_NM <> 'DNR NOT RESPONSIBLE'
```

"Current DNR Fire Statistics" — every fire this calendar year on or threatening
DNR-protected land. This drives the season totals and the grey context dots.

The `where` clause is **not** arbitrary: it's the same definition expression WA
DNR applies to this layer in
[its own Experience Builder dashboard](https://experience.arcgis.com/experience/6cdda73cf6154949a1fae76ccb2900a0).
Layer 1 also carries incidents DNR merely recorded rather than responded to; in
2026 that was 150 records holding 224,331 acres, about 37% of the layer's
acreage. Omitting the filter inflates every season total.

### 3. Address search — Esri World Geocoder

Suggestions and geocoding are constrained to a Washington bounding box, and
results outside it are rejected before being mapped.

---

## What the page derives (and how)

These are this page's editorial choices, not agency definitions.

| Shown as | Rule |
|---|---|
| 🟠 **Active** | NIFC incident with no `FireOutDateTime` and `PercentContained` under 100 |
| 🟤 **Contained · out** | NIFC incident that is out, or 100% contained |
| ⚪ **Season · DNR** | Any DNR record for the current year (mostly already out) |
| Marker size | Acres — `DailyAcres` (NIFC, falling back to `CalculatedAcres`) or `ACRES_BURNED` (DNR) |

**"Active" deliberately does not use DNR's out-date.** A DNR record stays open
until an out-date is filed, and that filing lags the fire by weeks — of 96 open
2026 records, 68 were already controlled and merely awaiting paperwork. Treating
"no out-date" as "burning now" overstates active fire by roughly 7×, so activity
is read from NIFC instead.

**The two feeds overlap.** Around 13 fires appear in both — DNR-protected fires
large enough to reach the interagency roster. They are drawn twice, grey
underneath and ember on top; the ember covers the grey. Season totals and active
totals are therefore *not* additive, and are labelled as separate blocks
because of it. Every tooltip carries a `Source` row naming its feed.

---

## Notes for anyone reading the code

- **Queries are paginated.** The DNR service caps a response at
  `maxRecordCount` 2000, and `f=geojson` does not return
  `exceededTransferLimit`, so overflow would be silent. Washington cleared 2000
  in 2021 (2,221) and 2025 (2,111).
- **Field names are pinned** to the DNR schema, with the original heuristic
  matching retained as a fallback if DNR renames a column.
- **NIFC failing is non-fatal.** DNR season totals still render, with a notice —
  an outage must never read as "no fires burning".
- `index.html` is a self-contained bundle: the app source and Leaflet live in
  base64 blocks at the bottom of the file, unpacked at load.

## Sources

Data © [Washington State DNR](https://dnr.wa.gov/) and the
[National Interagency Fire Center](https://www.nifc.gov/), used as published.
Basemap © [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors,
tiles © [CARTO](https://carto.com/attributions). Geocoding by Esri.
