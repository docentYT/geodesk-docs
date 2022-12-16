---
id: build-config
layout: default
title: Settings
parent: build
grand_parent: GOL Utility
nav-order: 0
---

# Settings for the `build` Command

## `area-tags`  ~~0.2~~ {#area-tags}

The tags that determine whether a closed OSM way is treated as an area or a linear ring. Rules can be specified for one or more keys. A closed way is treated as an area if it fulfills at least one of these rules (or is explicitly tagged `area=yes`), and is *not* tagged `area=no`.

Key rules have the following format:

<div class="language-plaintext highlighter-rouge">
<pre class="highlight"><code><i>key</i> [ <b>(</b> <b>only</b>|<b>except</b> <i>value</i>+ <b>)</b> ]
</code></pre>
</div>


Multiple key rules and values must be separated by whitespace and/or commas.

Example:

```
area-tags:
  building                            // Any "building" tag (except "building=no")
  barrier (only city_wall, ditch)     // "barrier=city_wall" or "barrier=ditch" 
  man_made (except embankment)        // Any "man_made" tag (but not "man_made=embankment" 
                                      // or "man_made=no")     
```

## `id-indexing` ~~0.3~~ {#id-indexing}

Value: `yes` / `no` (defaults to value of [`updatable`](#updatable))

If enabled, instructs the `build` command to retain the external ID indexes used during building, so incremental updates can be processed faster (`updatable` must be enabled).

In case this option is disabled, the [`update`](/gol/update) command can also re-create these indexes if needed.


## `indexed-keys`

To enhance query performance, GOLs organize features into separate indexes based on their tags. The `index-keys` section specifies which keys should be considered for indexing. The ideal keys for indexing are those that create categories of features (similar to *layers* in a traditional GIS database), such as `highway`, `landuse` or `shop`. As the number of indexes is limited (see [`max-key-indexes`](#max-key-indexes)), multiple keys may be consolidated into one index (This is done automatically on a per-type, per-tile basis). Features whose tags have multiple indexed keys (e.g. `tourism` and `amenity` for a hotel that is also a restaurant) are consolidated with features with the same key, or placed into a separate mixed-key index.

Keys that should always be placed into the same index can be specified as *key-pairs* by placing forward slahes between these keys (useful for rare-but-similar categories like `telecom`/`communication`).

Example:

```
indexed-keys:
  amenity
  building
  highway
  natural/geological
  shop
```


## `key-index-min-features`

Value: 0 &ndash; 1,000,000 (default: 300)

If there are fewer features in a key index than this number, these features will be consolidated into another index.

Used with [`indexed-keys`](#indexed-keys) and [`max-key-indexes`](#max-key-indexes).


## `max-key-indexes`

Value: 0 &ndash; 30 (default: 8)

The maximum number of key-based indexes to create, per feature type (*node*, *way*, *area*, *relation*). A higher number boosts the performance of queries that make use of indexed keys (queries that require the presence of a key/tag). However, a higher number of key indexes may reduce the performance of queries not based on indexed keys. Key indexes are very storage-efficient, so specifying a higher number has a minimal impact on file size.

If the number of key indexes is lower than the number of keys and key-pairs in [`indexed-keys`](#indexed-keys), features with less frequent keys will be consolidated in one or more combined indexes. Index consolidation also happens if the number of features in an index is below [`key-index-min-features`](#key-index-min-features)

Specifying `0` disables key indexing.


## `max-strings`

Value: 256 &ndash; 65,535 (default: 16,384)

The maximum number of strings that will be stored in the GOL's Global String Table. A higher value results in a smaller GOL file and increased query performance. (Loading a larger string table consumes more memory and may cause a slight delay when opening a GOL file, but the impact of this is generally negligible.)

The actual number of strings will be less if fewer strings meet the minimum usage threshold ([`min-string-usage`](#min-string-usage))


## `max-tiles`

Value: 1 &ndash; 8,000,000 (default: 65,535)

The maximum number of tiles into which the features of the GOL are organized. The actual number of tiles may be significantly less, based on the [`min-tile-density`](#min-tile-density). A lower tile count results in a more compact GOL, while a higher tile count improves the performance of certain large-area spatial queries.

A higher setting is also preferred if you intend to host a tile repository, as a more granular tileset reduces the amount of data users will have to download for their regions of interest.

If this number is set too low, a tile may exceed the maximum size of 1 GB (uncompressed). An unreasonably low setting may also cause the build process to fail with an `OutOfMemoryError`.


## `min-string-usage`

Value: 1 &ndash; 100,000,000 (default: 1,000)

Specifies the minimum number of times a string must be used by features (as a tag key or value, or as a role in a relation) in order to be included in the GOL's Global String Table.

See [`max-strings`](#max-strings).


## `min-tile-density`

Value: 1 &ndash; 10,000,000 (default: 25,000)

If there are fewer nodes in a tile area than this number, the tile will be omitted, and all features in the tile area will be placed into tiles at lower zoom levels. A lower threshold will result in more tiles, up to the maximum specified by [`max-tiles`](#max-tiles).  


## `properties`

A section with of key-value pairs that are stored as GOL metadata, which are displayed by [`gol info`](/gol/info) and can be read by other applications. 

Common properties include:

`generator`   | The program used to create the GOL ("geodesk/gol {{ site.geodesk_version }}")
`copyright`   | Text indicating the copyright holder of the data ("OpenStreetMap contributors")
`license`     | The license under which the data is distributed ("Open Database License 1.0")
`license-url` | Link to the website where the license text can be found ("https://opendatacommons.org/licenses/odbl/1-0/")
`tileset-url` | The default URL from which tiles can be downloaded or updated (e.g. "https://data.geodesk/world")

<blockquote class="important" markdown="1">

If you wish to distribute tilesets based on OpenStreetMap data, you must do so in accordance 
with the [Open Database License](https://opendatacommons.org/licenses/odbl/1-0/). You can
use the `build` command to create a GOL from any geodata in OSM-PBF format, so in theory,
GOLs could contain data from non-OSM sources (or very old OSM datasets distributed under a
Creative Commons License) -- but in general, you should not override the defaults for
`copyright`, `license` and `license_url`.

</blockquote>

To set properties from the command line, use <code>--properties:<i>property</i>=<i>value</i></code> or <code>-p:<i>property</i>=<i>value</i></code>.

## `rtree-bucket-size`

Value: 1 &ndash; 256 (default: 16)

GOLs use R-tree indexes to accelerate spatial queries. This setting specifies the maximum number of features (or child buckets) in each bucket (A *bucket* is a node in the R-tree). A larger number reduces the size of the GOL, a smaller number increases query performance (but set too low, it may have the opposite effect). Square numbers (4, 9, 16, 25, etc.) tend to perform best.    


## `tag-duplicate-nodes`

Value: `yes` / `no` (default)

By default, untagged nodes are discarded (unless they are a relation member) -- they simply exist as locations on ways. In certain cases, problems arise if two or more untagged nodes share the same location, because it becomes unclear whether features with coincident geometry are supposed to be connected. Enabling this option causes such duplicate nodes to be tagged with `geodesk:duplicate=true`, which turns them into feature nodes (with a distinct identity).   

## `tag-orphan-nodes`

Value: `yes` / `no` (defaults to value of [`updatable`](#updatable))

By default, any nodes that have no tags and aren't referenced by any ways or relations are discarded. However, if this option is enabled, such *orphan nodes* are tagged `geodesk:orphan=true` and retained as feature nodes. 

If a GOL is [`updatable`](#updatable), this option is enabled by default.  


## `tile-zoom-levels`

The zoom levels at which tile-tree nodes should be created. Together with [`max-tiles`](#max-tiles) and [`min-tile-density`](#min-tile-density), this setting shapes the tile structure of a GOL.

- Zoom levels must be between 0 and 12.
- The difference between zoom levels must not exceed 3 (eg. you can specify `0,3,6,9`, but not `0,4,6,12`).
- The root level (`0`) is always included implicitly.
- Fewer zoom levels result in a flatter tree that may yield better query performance, but cause a higher variance in tile sizes.
- Setting the top zoom level too low may cause the maximum tile size (1 GB uncompressed) to be exceeded. (Very large tiles may also cause the build process to run out of memory.)

## `updatable` ~~0.3~~ {#updatable}

Value: `yes` / `no` (default)

Enables incremental updates to the GOL file (using the [`update`](/gol/update) command). If enabled, additional storage is required for the way-node indexes (an extra 20% above the size of the GOL). The `build` command will also take slightly longer.

For GOLs built from a planet-wide dataset, it is highly recommended to also enable `id-indexing`, which speeds up the processing of updates. The `id-indexing` option does not create extra work for the `build` command, as it has to create these indexes anyway. If kept, these indexes do however consume significant extra storage (25% above the size of the GOL, with a higher ratio for extracts).

If you no longer need to update a GOL, you can delete its external indexes and recover the storage. If you delete the ID indexes (or disable `id-indexing` during the initial build), you can later re-create them with the `-i` option of the `update` command. However, deleting the way-node indexes permanently disables updating of the GOL, as these indexes cannot be re-created (You would then need to run the `build` command again to build a new updatable GOL).
