---
name: movingpandas
description: Use when analyzing movement or trajectory data (GPS tracks, vehicle/vessel/AIS data, animal tracking, mobility traces) with the MovingPandas Python library. Covers building trajectories, computing speed/direction, splitting, cleaning, generalizing, stop detection, and plotting.
---

# MovingPandas

Python library for trajectory analysis built on GeoPandas.

## Setup
```bash
pip install movingpandas
```
```python
import movingpandas as mpd
import geopandas as gpd
import pandas as pd
from datetime import timedelta
```

## Building trajectories
- Input needs point geometries, a timestamp column, and (for collections) a trajectory ID column.
- Always confirm the timestamp column is a real datetime (`pd.to_datetime`).
- Always check and set the CRS. Speeds and distances depend on it.

```python
gdf = gpd.GeoDataFrame(df, geometry=gpd.points_from_xy(df.lon, df.lat), crs="EPSG:4326")
tc = mpd.TrajectoryCollection(gdf, traj_id_col="id", t="timestamp")
traj = tc.trajectories[0]   # single mpd.Trajectory
```

## Enriching
```python
tc.add_speed(overwrite=True, units=("km", "h"))
tc.add_direction(overwrite=True)
tc.add_acceleration(overwrite=True)
tc.add_distance(overwrite=True)
tc.add_timedelta(overwrite=True)
```

## Splitting
```python
mpd.ObservationGapSplitter(tc).split(gap=timedelta(minutes=30))
mpd.StopSplitter(tc).split(max_diameter=100, min_duration=timedelta(minutes=5))
mpd.TemporalSplitter(tc).split(mode="day")
```

## Cleaning and generalizing
```python
mpd.OutlierCleaner(tc).clean(v_max=200, units=("km", "h"))
mpd.DouglasPeuckerGeneralizer(tc).generalize(tolerance=0.001)
mpd.MinTimeDeltaGeneralizer(tc).generalize(tolerance=timedelta(minutes=1))
```

## Stop detection
```python
detector = mpd.TrajectoryStopDetector(tc)
stops = detector.get_stop_points(min_duration=timedelta(minutes=5), max_diameter=100)
```

## Exporting and plotting
```python
tc.to_point_gdf(); tc.to_line_gdf(); tc.to_traj_gdf()
tc.plot(column="speed", legend=True)
tc.explore()      # interactive folium map
```

## Pitfalls
- Unsorted or duplicate timestamps cause errors. Sort by ID and time and drop duplicates first.
- With a geographic CRS (EPSG:4326), distances are computed in meters. With a projected CRS, they use the CRS units.
- Many operations return new objects rather than modifying in place, so assign the result.
- Check the installed version (`mpd.__version__`), because APIs change between releases.
