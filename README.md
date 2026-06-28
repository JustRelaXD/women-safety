# Women Safety Drone System

## Project Purpose

This project is a real-time drone dispatch and monitoring visualization system built to enhance women's safety in Mangalore. It features an interactive map displaying drone patrol routes, high-risk danger zones (heatmaps), and live tracking of autonomous drones.

The system provides a command center interface to:
- Monitor drones as they continuously patrol predefined safe corridors and high-risk zones.
- Respond to SOS alerts by automatically dispatching the nearest available drone to the incident location.
- Provide a "Safe Walk" feature, where a drone escorts a user from their origin to their destination.
- Visualize real-time tracking, response metrics, and operational timelines.

## Tech Stack

- **Frontend:** React, Vite
- **Mapping:** MapLibre GL
- **Geospatial Analysis:** Turf.js (for route calculations, segment lengths, and distance checks)
- **Animations:** GSAP (for smooth, constant-speed drone movement along paths)
- **Styling:** Framer Motion, Lucide-React (icons)
- **Backend:** Express / Node.js (for serving API/static assets)

## Main Features

### 1. Continuous Patrol Loops
Drones continuously travel along predefined `LineString` patrol paths. The movement is calculated segment-by-segment using Turf.js and GSAP to ensure drones strictly adhere to their designated routes at a constant speed, without cutting corners.

### 2. SOS Emergency Dispatch
When an SOS alert is triggered, the system identifies the nearest available drone and dispatches it immediately. 
- The drone leaves its patrol route and takes a direct path to the SOS target.
- Upon arrival, it initiates an orbit pattern around the incident zone.
- Simulated telemetry (Audio Recording, Video Broadcast) is displayed to the command center.

### 3. Safe Walk Escort
Users can request a "Safe Walk" drone escort.
- The user selects an origin and destination.
- The nearest drone is dispatched to the user's origin.
- The drone then escorts the user to their destination along a generated safe route.
- Progress and ETA are updated dynamically on the map and the sidebar.

### 4. Danger Zones & Heatmaps
High-risk areas are identified and displayed on the map using a customized heatmap layer. Danger zone markers sit directly on the patrol paths, ensuring drones pass through these high-priority areas regularly during their normal patrols.

### 5. Dynamic Zoom Visibility
Map layers dynamically adjust based on zoom levels to reduce clutter:
- **Zoom ≥ 11.6:** Full detail (patrol routes, hotspots, drone stations, drones).
- **Zoom 10.8 to 11.6:** Only drones and the heatmap are visible, allowing operators to track fleet positions easily from a higher level.
- **Zoom < 10.8:** Drones fade out to show only the macro-level heatmap.

## Running the Project

### Installation
Make sure you have Node.js installed, then run:
```bash
npm install
```

### Development Server
You can start the development server using Vite:
```bash
npm run dev
```
Alternatively, you can run the provided `run_dev.sh` script.

The application will be accessible at:
`http://localhost:5173`

### Production Build
To build the project for production:
```bash
npm run build
```
To serve the production build:
```bash
npm run server
```
