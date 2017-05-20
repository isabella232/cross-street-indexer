# Cross Street Indexer

[![Build Status](https://travis-ci.org/DenisCarriere/cross-street-indexer.svg?branch=master)](https://travis-ci.org/DenisCarriere/cross-street-indexer)
[![npm version](https://badge.fury.io/js/cross-street-indexer.svg)](https://badge.fury.io/js/cross-street-indexer)
[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/DenisCarriere/cross-street-indexer/master/LICENSE)

Light weigth reverse geocoding for cross street 100% sourced from [OSM QA Tiles](https://osmlab.github.io/osm-qa-tiles/).

![image](https://cloud.githubusercontent.com/assets/550895/26235719/a8f8e7da-3c21-11e7-9240-c811f9b6a4aa.png)

## Process

- [x] **Step 1**: Filter data from QA-Tiles (`lib/qa-tiles-filter.js`)
- [x] **Step 2**: Extract road intersections from QA-Tiles (`lib/intersections.js`)
- [x] **Step 3**: Normalize street name (`ABBOT AVE. => abbot avenue`)
- [ ] **Step 4**: Convert intersections into multiple points using a combination of `road` & `ref` tags.
- [ ] **Step 5**: Convert points to normalized hashes
- [ ] **Step 6**: Group all hashes into single Quadkey JSON object.
- [ ] **Step 7**: Publish to S3 or LevelDB

## Features

- Easy to use CLI
- Node 6 & 7 compatible
- Only uses Tile Reduce + Turf
- Bundled 5MB QA-Tiles for testing purposes

## Quickstart (in development)

```bash
$ npm install
$ npm test
```

## CLI

```
$ cross-street-indexer --help

  Cross Street Indexer

  Usage:
    $ cross-street-indexer <qa-tiles>
  Options:
    --output    [cross-street-index] Filepath to store outputs
    --bbox      Excludes QATiles by BBox
    --tiles     Excludes QATiles by an Array of Tiles
    --debug     [false] Enables DEBUG mode
    --verbose   [false] Verbose output
  Examples:
    $ cross-street-indexer latest.planet.mbtiles cross-street-index/
```

## OSM Attributes

- `name`: Street name (Abbot Avenue)
- `ref` Reference number normaly used to tag highways numbers
- [`highway`](http://wiki.openstreetmap.org/wiki/Key:highway) classification (residential, primary, secondary)
- [`bridge`](http://wiki.openstreetmap.org/wiki/Key:bridge) yes/no
- [`tunnel`](http://wiki.openstreetmap.org/wiki/Key:tunnel) yes/no
- `@id` OSM ID of street (way)

## Normalization

Normalization should follow the following standards:

- Drop any period if exists
  - ave. => avenue
- Street suffix to full name
  - ave => avenue
  - CIR => circle
  - ln => lane
- Name should be entirely lowercase
  - Parkside Avenue => parkside avenue
- Direction to full word
  - N => north
  - S => south
  - NE => northeast

## Debugging

Adding `--verbose` will `stdout` a JSON object for each QA-Tile with the following information:

```
{
  tile: [ 655, 1586, 12 ],
  quadkey: '023010221131',
  features: 9626,
  intersects: 1053
}
```

Adding `--debug` will store `.geojson` items for each process & for each QA-Tile:

- `debug/<quadkey>/features.geojson` - raw GeoJSON of QA-Tile
- `debug/<quadkey>/lines.geojson` - Filtered (Multi)LinesString from QA-Tile
- `debug/<quadkey>/intersects.geojson` - Point which are intersecting roads

## Limitations

### Loops

Loops would return multiple cross street matches, only the last matched point is stored.

### Turning Circles

Turning Circles without any names are exclude, thus not finding any matches.

![image](https://cloud.githubusercontent.com/assets/550895/26234213/d26554b4-3c17-11e7-8f89-bee790f7118c.png)

## References

- [ ] Review/complete [`osmify`](https://github.com/osmottawa/osmify) to parse random address into an OSM friendly schema.
- [ ] Use [JWT](https://github.com/auth0/node-jsonwebtoken) as hash?
- [ ] [Natural Node](https://github.com/NaturalNode/natural) to create hash
- [ ] [Carmen](https://github.com/mapbox/carmen) to normalize roads
- [ ] [LevelDB](https://github.com/google/leveldb) as storage
- [ ] S3 Bucket Upload to hashes
- [ ] Exact OSM match
- [ ] Fuzzy OSM match
- [x] Use Map & Set