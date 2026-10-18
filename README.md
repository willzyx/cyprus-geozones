# Cyprus Drone Geozones Map

Interactive map of drone geozones in Cyprus — prohibited and restricted flight
areas — that tracks your device location in real time and shows the distance to
the nearest zone.

**Live: <https://willzyx.github.io/cyprus-geozones/>**

## Features

- All 141 official geozones (`PROHIBITED` and `RESTRICTED`) rendered with their
  original KML colors, each with a popup describing the zone.
- Zone names and descriptions translated from Greek to English.
- Live geolocation via `watchPosition`, with an accuracy circle around your
  position.
- The nearest zone is highlighted and shown in the bottom panel with the
  distance to its boundary — or a red "You are inside the zone" warning when you
  are within one.

## Running locally

The app is plain static HTML with no build step, but it fetches the KML file, so
it needs to be served over HTTP rather than opened from the filesystem
(geolocation also requires a secure context — `localhost` counts):

```sh
python3 -m http.server 8000
# open http://localhost:8000/
```

## How it works

Everything lives in [`index.html`](index.html): markup, styles, and a single
vanilla-JS IIFE. There are no dependencies to install — [Leaflet
1.9.4](https://leafletjs.com/) and [Turf.js 6](https://turfjs.org/) are loaded
from a CDN, and map tiles come from OpenStreetMap.

On load, the KML is fetched and parsed in the browser with `DOMParser`: the
parser builds an index of `<Style>` and `<StyleMap>` elements, converts KML
`AABBGGRR` colors to RGBA, and extracts a polygon from each `<Placemark>`,
taking its category from the name of the parent `<Folder>`.

The KML text is in Greek. [`zones-en.json`](zones-en.json) is a flat list of
`{"original", "translation"}` pairs, where `original` is a zone's full name or
description exactly as it appears in the KML. While parsing, each string is
looked up there and replaced with its English translation; anything without an
entry stays in Greek and is listed in a single `console.warn`, so after updating
the KML the browser console shows what still needs translating.

Each geolocation update is tested against every zone with Turf —
`booleanPointInPolygon` to detect containment, and `pointToLineDistance` to the
boundary otherwise — and the closest one is highlighted.

## Data

[`CYPRUS_GEO_ZONES_MAP_ver09.kml`](CYPRUS_GEO_ZONES_MAP_ver09.kml) is the
official geozone dataset published by the Cyprus Department of Civil Aviation at
<https://drones.gov.cy/geo-zones-map/>. It is included here for convenience and
is **not** covered by this repository's license; its copyright remains with the
original publisher.

The English translations in [`zones-en.json`](zones-en.json) are unofficial;
the Greek text in the KML is authoritative.

The snapshot is version 09 and is updated manually, so it may lag behind the
official map. This project is an unofficial convenience tool with no warranty —
always check <https://drones.gov.cy> before flying.

## License

The code in this repository is released under the [MIT License](LICENSE).
