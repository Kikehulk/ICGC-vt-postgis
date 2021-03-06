# ICGC-vt-postgis

A set of PostGIS utility functions for generating and instrumenting (Mapbox
Vector Tiles)[https://www.mapbox.com/vector-tiles/specification/], used at
(Institut Geològic i Cartogràfic de Catalunya - ICGC)[http://www.icgc.cat/] for
generating the _ContextMaps_ product.

# Requirements

The system has been tested in: 

- PostgreSQL 9.5
- PostGIS 2.4

But may work with previous versions. Please let us know your experience with
other versions.
 
# Installation

Everything is installed in the `icgc-vt-postgis-create.sql` sql file, so
installation boils down to importing this file, eg:

```
psql -U <username> -d <dbname> -f icgc-vt-postgis-create.sql
```

Make sure the username has the appropiate privileges to do so.

All functions and expected tables are installed in the `icgc_vt` schema. To
use them from another schema, make sure to fully qualify the schema or to alter
the search path.

# Filling user data

The system expects two tables to be filled by the user:

## layers

This table contains information on which layers will be embedded in each tile protobuffer. 

- name of layer
- kind of geometry (LineString, Polygon,...). 
- zoom bounds where the layer should appear.
- SQL returning query. 

*IMPORTANT:* The SQL query must return a `geom` column with the geometry. 

## tiles

This table should contain the tile coordinates and envelope for each tile that
can be generated in the system. In the `data\tiles.sql` file you can find an
example for the tiles in 3857 schema covering Catalonia between zooms 7-14
(these are the tiles used by ICGC's `ContextMaps` product).

# Usage

Once the data is available in the required tables, generating a VT is as easy
as calling the function:

```sql
SELECT icgc_vt.tile_pbf(z,x,y);
```

This returns a binary that can be saved to a file o returned directly to a
browser client, etc...

# Statistics

When the setting `icgc_vt.log_stats` is set to `on`, data regarding
the rendering time and size for each layer is logged into the `layer_stats`
table. It is up to the user to reset this table whenever it makes sense.

The standard pattern would be to activate logging and reset the stats table:

```sql
DELETE FROM layer_stats;
SELECT pg_catalog.set_config('icgc_vt.log_stats', 'on', false);
```

Then perform the queries that we would like to analyze, eg:

```sql
SELECT icgc_vt.tile_pbf(z,x,y) FROM icgc_vt.tiles;
```

And then perform any statistical analysis on the results.

# Uninstall

Since all data is contained in the `icgc_vt` schema, removal is just a matter
of calling:

```
DROP SCHEMA icgc_vt CASCADE;
```

