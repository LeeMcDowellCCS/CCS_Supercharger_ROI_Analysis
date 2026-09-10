# Tesla Supercharger Busy Times — Georgia

`tesla_busy_times_ga.json` holds the hourly "Busy Times & Price Per kWh" profile that tesla.com/findus
shows for every open Georgia Supercharger site, captured 2026-09-10. `tesla_busy_times_screens/<slug>.png`
is the chart canvas exported from tesla.com for each site so the numbers can be audited against the rendering.

## Where the numbers come from

The findus page draws the chart with Chart.js from the JSON returned by

```
GET https://www.tesla.com/api/findus/get-charger-details?locationSlug=<slug>&programType=supercharger&locale=en-US&isInHkMoTw=false
```

`data.data.availabilityProfile.availabilityProfile.<weekday>.congestionValue` is a 24-value array per weekday
(Tesla's hourly share of stalls occupied, 0..1). The arrays are indexed by **UTC hour**; the page shifts them with
`utcOffset` (-4 h in EDT) so that index 8 is the 4a bar, and the 8p..3a bars come from the next weekday's
indices 0..7. The chart shows the current weekday, starts at 4a, draws the current hour black, and puts a dashed
line at congestion 1.0 (all stalls busy); in the captured charts that line only appears on sites with TOU price
regions, flat-rate sites draw bars alone. Price regions come from `effectivePricebooks` (Tesla-member TOU rates).

Site slugs came from `GET /api/findus/get-locations?country=US&view=map`, filtered to a Georgia bounding box and
`location_type` in `supercharger`, `nacs` (open to non-Tesla EVs) or `party` (host-owned sites such as ChargedEV
Norcross, which Tesla lists separately). Sites were then kept only when the detail record's state is GA.

## Fields per site

| field | meaning |
| --- | --- |
| `slug`, `name`, `common_site_name`, `address`, `lat`, `lon` | Tesla identifiers; `?location=<slug>` opens the site on tesla.com/findus |
| `stalls`, `max_power_kw`, `open_to_non_tesla`, `amenities` | from the detail record |
| `host_branded`, `brand_name`, `ownership_type`, `findus_location_type` | `CUSTOMER_OWNED` + a brand name marks a host-branded site |
| `site_type_guess` | heuristic from host name and location: `metro-retail`, `regional-retail`, `highway`, `urban`, `hotel`, `other` |
| `price_bands`, `price_bands_non_tesla`, `congestion_fee_per_min` | per-kWh bands starting at midnight (`0a`), Tesla-member and non-Tesla |
| `hours` | chart order, `4a` … `3a` |
| `bar_shares` | 7-day mean congestion normalized to sum 1.0, in `hours` order |
| `bar_shares_weekday`, `bar_shares_weekend`, `bar_shares_by_day` | same normalization for Mon–Fri, Sat–Sun, and each weekday |
| `congestion_mean`, `congestion_by_day` | the raw occupancy values (0..1) behind the shares |
| `dashed_line_value`, `dashed_line_ratio` | dashed line is congestion 1.0; ratio = 1.0 / tallest mean bar |
| `profile_created_at`, `utc_offset_hours`, `captured_at`, `source` | provenance (`source` = `json`) |
| `screenshot`, `screenshot_day`, `screenshot_chart_matches_json` | audit image and whether the rendered dataset equalled the API profile that day |

The reference site digitized by hand earlier (ChargedEV Norcross, Jimmy Carter Blvd) is slug `487103`.
