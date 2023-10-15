# Taming Big Data with Spatial Indexing

## Slides

The slide deck for this workshop can be found [here](https://docs.google.com/presentation/d/174nAsJsfP_HJryF5L3H01lZaTscO7TuTvUB3r2_kKXg/edit?usp=sharing).

## Data

The data used in this workshop has been imported into BigQuery in advance. The tables we'll use include:

- `SDSC23_NYC_indexes.manhattan_311_rodent_service_requests`: Point data showing all service requests related to rodents in Manhattan

- `SDSC23_NYC_indexes.manhattan`: Polygon geometry of Manhattan

- `SDSC23_NYC_indexes.spatial_features_manhattan`: Dataset already in H3 format (resolution 8) with a large number of demographic, POI, and weather attributes, covering Manhattan

If you're working in your own CARTO organization, you can import the CSV files in the `/data/` sub-folder into your own data warehouse.

## Code Samples

### Spatial Join

```sql
with _area_of_interest as (
  select st_buffer(
    	st_geogpoint(-73.9860685114144, 40.75338081863744),
    	2000
  ) as geom
),
_h3_area_of_interest as (
  select
    h3
  from
    _area_of_interest,
      unnest(`carto-un`.carto.H3_POLYFILL(geom, 8)) as h3
)
select *
from SDSC23_NYC_indexes.spatial_features_manhattan
inner join _h3_area_of_interest using (h3)

```

### Aggregation

```sql
with _data_as_h3 as (
  select
    `carto-un`.carto.H3_FROMGEOGPOINT(geom, 10) as h3
  from
    SDSC23_NYC_indexes.manhattan_311_rodent_service_requests
)
select
  h3,
  count(*) as num_requests
from _data_as_h3
group by h3
```

### SQL Parameter within Builder

```sql
with _data_as_h3 as (
  select `carto-un`.carto.H3_FROMGEOGPOINT(geom, 10) as h3
  from SDSC23_NYC_indexes.manhattan_311_rodent_service_requests
  where parse_date('%m/%d/%Y', split(created_date, ' ')[ordinal(1)])
  	between {{date_from}} and {{date_to}}
)
select h3, count(*) as num_requests
from _data_as_h3
group by h3
```
