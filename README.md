# Mangalore Smart Recycle Routing Map

## Project Purpose

This project is a route optimization and map visualization system for waste and recycling logistics in Mangalore. It connects depots, customer pickup points, vendors, trucks, service orders, and the road network into one workflow where routes can be generated, stored, animated, imported, cleared, and inspected on an interactive map.

The system does more than display markers. It prepares a PostGIS and pgRouting database, snaps locations to the road network, calculates shortest paths, groups pending pickup orders into clustered area-wise plans, and renders the result through a React and MapLibre frontend.

## Tech Stack

- React and Vite frontend
- Express backend
- PostgreSQL with PostGIS
- pgRouting
- MapLibre GL
- Turf.js
- GSAP
- Framer Motion
- Lucide icons

## Main Files

- `01_schema.sql`: database schema, triggers, routing tables, sample logistics data
- `server.js`: Express API and PostGIS/pgRouting SQL
- `src/main.tsx`: React map application and route controls
- `src/styles.css`: frontend styling and animations
- `demo.sql`: sample SQL upload file for testing map imports
- `vercel.json`: deployment routing

## Database Work

The database uses a dedicated `mangalore` schema and expects an existing road network with:

- `mangalore.roads`
- `mangalore.roads_noded`
- `mangalore.roads_noded_vertices_pgr`

PostGIS and pgRouting are required.

The schema includes tables for:

- `locations`: depots, customers, vendors, bins, and truck stops
- `location_network_nodes`: nearest routing-network vertex for each location
- `customers`: customer information linked to locations
- `vendors`: vendor/recycling destinations and accepted waste types
- `trucks`: fleet vehicles
- `truck_availability`: truck availability windows
- `service_orders`: pending pickup jobs
- `route_plans`: full generated route plans
- `route_plan_stops`: ordered route stops
- `route_plan_segments`: road-network route geometry between stops
- `routing_edges`: pgRouting-compatible edges built from the noded road network
- `location_backup_snapshots`: local JSON backups created before import/clear actions

## Location Snapping

When a location is inserted or updated:

1. Latitude and longitude are converted to EPSG:3857 geometry.
2. The nearest road segment is found.
3. The closest point on that road is calculated.
4. The nearest pgRouting vertex is selected.
5. The snapped result is stored in `location_network_nodes`.

This lets the app keep user-facing coordinates while pgRouting works with network vertices.

## Backend API

The backend in `server.js` serves both the API and the built frontend from `dist`.

Main API features:

- health check
- table summary
- locations as GeoJSON
- roadside detections as GeoJSON
- snapped points as GeoJSON
- snap lines as GeoJSON
- routing edges as GeoJSON
- highway class summaries
- route plans
- route segments
- location create/update
- SQL location import
- clear locations
- mark map features as completed
- local backup restore endpoint
- point-to-point route preview
- manual route-plan creation
- clustered pickup route-plan generation

GeoJSON responses are normalized so empty result sets return `features: []`, not `features: null`. This prevents frontend crashes when no saved route segments exist.

## SQL Import Support

The app includes an **Import SQL** button in the Layers panel under Data Tools.

The backend can parse location data from:

- `INSERT INTO mangalore.locations (...) VALUES (...)`
- `COPY mangalore.locations (...) FROM stdin`

Imported locations are upserted by `name`. Database triggers then handle geometry conversion and snapping.

The included `demo.sql` file contains sample Mangalore markers that can be uploaded through the website for testing.

## Clear Locations

The Data Tools panel includes a **Clear locations** button.

Before clearing, the backend creates a local backup snapshot in `mangalore.location_backup_snapshots`. The clear action removes location-dependent routing data and location rows so the map can be emptied for testing.

The temporary Undo button was removed because it caused workflow issues. The backup table remains available on the backend for recovery/debugging if needed.

## Completing Map Features

Clicking supported map items opens a feature telemetry popup with a **Mark completed** action.

Supported completion targets:

- location markers
- snapped location points
- roadside detection markers
- saved route segments / generated cluster routes
- animated saved route lines

Completed location and roadside detection markers are hidden using `mangalore.completed_map_features`, so they disappear from the map without deleting the original source rows.

Completed clustered routes update `route_plans.status` to `COMPLETED`. Related service orders are also marked `COMPLETED`, and the completed route no longer appears in the route plan list or saved segment layer.

## Route Preview

`POST /api/routes/preview` takes a start location and destination location.

It looks up the snapped pgRouting vertices for both locations, runs `pgr_dijkstra` on `mangalore.routing_edges`, and returns the route as GeoJSON.

## Route Plan Creation

When a route plan is created:

1. A row is inserted into `route_plans`.
2. Ordered stops are inserted into `route_plan_stops`.
3. Consecutive stop pairs are calculated.
4. pgRouting shortest paths are generated between each pair.
5. The geometry is stored in `route_plan_segments`.
6. Total distance and cost are updated on the plan.

## Clustered Pickup Optimization

`POST /api/route-plans/generate-clustered-pickups` generates area-wise pickup plans from pending service orders.

The backend uses `pgr_dijkstraCostMatrix` across depots, pickup locations, and vendor locations. It then:

1. Finds a depot.
2. Finds pending service orders.
3. Finds active vendors.
4. Builds a network cost matrix.
5. Picks cluster seeds based on distance from depot.
6. Assigns orders to the nearest seed.
7. Checks waste types in each cluster.
8. Selects the best compatible vendor.
9. Orders pickups inside each cluster.
10. Builds route stops: depot start, pickups, vendor dropoff, depot return.
11. Creates route plans and route segments.
12. Marks included service orders as `PLANNED`.

Generated plan codes look like `AREA-01-XXXXXX`.

The frontend now animates clustered-plan regeneration: old plan rows collapse/fade out, a loading row appears, and new plan rows slide/fade back in after refresh.

## Frontend Map Application

The React frontend renders a CARTO light basemap and custom GeoJSON layers.

It loads:

- locations
- roadside detections
- snapped points
- snap lines
- saved route segments
- route plans
- highway summaries
- health/status information

Routing edges are no longer loaded on every refresh by default. Because that payload can be large, the app loads road edges only when the Routing edges layer is enabled.

## Map Layers

The map includes layers for:

- roadside detections
- real location points
- location halos
- snapped network points
- snap lines
- routing edges
- saved route segments
- active route remaining
- active route traveled
- destination beacon

Snap lines are off by default. The toggle remains available in the Layers panel.

The marker legend is positioned at the bottom-right corner of the map.

## Route Controls

The route panel lets the user:

- select a start location
- select a destination
- generate a route
- generate clustered pickup plans
- view recent generated plans
- play a saved route plan
- pause/play animation
- replay animation
- scrub route progress
- adjust animation speed

During route playback, Turf calculates total distance, traveled distance, remaining distance, ETA, and the current point along the route. GSAP animates route progress from 0 to 1.

## Feature Telemetry

Clicking map features opens a popup with feature properties.

This works for:

- roadside detections
- locations
- snapped points
- routing edges
- saved route segments
- active route lines

A click pulse animation is shown at the clicked point.

For supported markers and generated route clusters, the popup also includes a **Mark completed** button that updates the database and refreshes the map so the completed item disappears.

## Stability Updates

Recent stability work includes:

- fixed empty GeoJSON responses so `features` is always an array
- added frontend GeoJSON normalization as a defensive guard
- added error handling around route preview, clustered generation, route playback, and refresh
- avoided loading hidden routing-edge geometry by default
- disabled snap lines by default to reduce visual clutter
- removed a duplicate roadside detection layer registration that could break map initialization

## Running the Project

Install dependencies:

```bash
npm install
```

Build the frontend:

```bash
npm run build
```

Run the Express server:

```bash
npm start
```

The app runs at:

```text
http://localhost:3000
```

For development:

```bash
npm run dev
```

## Overall Summary

The project now provides a complete routing and visualization pipeline:

1. Store real logistics locations.
2. Convert locations into PostGIS geometries.
3. Snap each location to the routing network.
4. Build pgRouting-compatible road edges.
5. Store customers, vendors, trucks, and service orders.
6. Import test locations from SQL files.
7. Clear map locations for repeated testing.
8. Generate shortest routes between selected locations.
9. Generate clustered area pickup plans from pending orders.
10. Match clusters to suitable vendors.
11. Save route plans, ordered stops, and route geometries.
12. Display operational layers on an interactive map.
13. Animate route playback and clustered-plan refreshes.
14. Mark completed markers and generated route clusters so they disappear from the map.
15. Provide controls for layers, route playback, imports, clearing, and visual tuning.

In short, this project turns Mangalore road-network and recycling logistics data into a working route planning, optimization, and map visualization system.
