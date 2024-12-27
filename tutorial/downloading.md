# Getting started

We recommend having a terabyte available if you plan to download all the imagery within, for example, a 25km x 25km region around a city like Amsterdam. In fact, many cities will have much less; Amsterdam enjoys a very large and dense dataset of panoramic imagery, which tends to take up much more disk space than ordinary photographs. Please investigate your region of interest on the [Mapillary map app](https://www.mapillary.com/app/) to get a feel for how much imagery and how much panoramic imagery is available. We will assume that you are cloning the scripts and working within the drive that has the available space, therefore our examples will use relative paths, but please feel free to use whichever paths make sense for your system.

## Getting our software

Please clone or download the [percept-vsvi-filter](https://github.com/Spatial-Data-Science-and-GEO-AI-Lab/percept-vsvi-filter) repository and open a shell in that directory.

e.g. `git clone https://github.com/Spatial-Data-Science-and-GEO-AI-Lab/percept-vsvi-filter && cd percept-vsvi-filter`

## Mapillary API key

You will need a Mapillary developer API key in order to run the script that downloads imagery. This can be obtained free-of-charge from the [Mapillary Developer Dashboard](https://www.mapillary.com/dashboard/developers). You will need to register a Mapillary account first, and then you can register an application on the dashboard. Please simply choose something reasonably descriptive for the App Name and Description. Remember that you are ultimately responsible for using the developer access according to the [terms of service](https://www.mapillary.com/terms); read everything of course, but in particular, consider section 11 **Additional Terms for Developers**. Our scripts are provided as-is and come with NO warranty and NO guarantee. To the best of our knowledge these scripts comply with the terms of service for non-commercial usage, provided that you attribute the downloaded imagery to Mapillary if and when you serve them from your own servers.

In the end, you will need to copy the text in the field `Client Token` from the developer dashboard and save it in a file, we recommend calling that file `token.txt` and putting that file in the `percept-vsvi-filter` directory that you obtained in the previous step, where the `mapillary_jpg_download.py` script lives.

Tokens are like passwords, so do NOT commit the token file to a repository, do not include your token in any code file, and in fact if you use `git` then adding `token.txt` to your `.gitignore` file is a good idea (there are similar features in other version control systems too).

## Downloading imagery from Mapillary

## `mapillary_jpg_download.py`

This is the step where you get to decide what region of the world you wish to study. But first, be sure you have obtained the software dependencies. You will need Python v3.6 or higher, and you can either run `pip install -r requirements.txt` to get all the requirements for all of the scripts in the preparation suite, or you can specifically run `pip install mercantile mapbox_vector_tile vt2geojson pillow requests` if you do not plan to use any of the other scripts. There is also a Docker option, see the [README](https://github.com/Spatial-Data-Science-and-GEO-AI-Lab/percept-vsvi-filter#alternative-docker-set-up).

Create a file such as `myconfig.json` using the following template:
``` 
    {
      "bounding_box": {
            "west": 4.7149,
            "south": 52.2818,
            "east": 5.1220,
            "north": 52.4284
      },
      "tile_cache_dir": "<tile directory>",
      "seqdir": "<sequence directory>"
    }
```
- `west` and `east` are longitudes marking the western and eastern boundaries of the region.
- `north` and `south` are latitudes marking the northern and southern boundaries of the region.
- The example above is for a region around Amsterdam.
- You can use [geojson.io](https://www.geojson.io) to draw rectangles on the map and get GeoJSON output. The coordinates will be displayed in `[X,Y],...` format where `X` is longitude and `Y` is latitude. The first four coordinates will define the rectangle and you will see that in fact there are only two different values for longitude and two different values for latitude, corresponding to the west, east, north, south boundaries.
- `<tile_cache_dir>` and `<seqdir>` should be directories with descriptive names where you can store the files that you obtain from Mapillary.
- `<tile_cache_dir>` is used for storing GeoJSON information from Mapillary about the map. You may want to use a name like `ams-tiles/` if you were downloading tiles in the Amsterdam area, for example.
- `<seqdir>` is the main directory where the bulk of the imagery data will be stored, so it must be on a filesystem with a lot of space available. You may want to use a name like `ams-seqs/` if you were downloading Amsterdam imagery, for example.

Assuming that you saved your Mapillary API token into a file named `token.txt`, then you can begin the download process using the following command (be aware, can take a long time to complete):
`./mapillary_jpg_download.py -c myconfig.json`

The download process is interruptible and restartable. So feel free to press Control-C to stop it and simply restart it with the same configuration file later when you feel like it. The script will automatically figure out what has already been downloaded and continue where it left off. For more documentation about script options see the [README](https://github.com/Spatial-Data-Science-and-GEO-AI-Lab/percept-vsvi-filter#mapillary_jpg_downloadpy).

In the end, you will be left with a bunch of GeoJSON files in `<tile_cache_dir>` and many many gigabytes of imagery in `<seqdir>`.

## Alternative: download only the images at intervals of X meters

The previous way of downloading images gathers every possible image within the defined region. This will ensure that you have every possible option, even if you have to throw out a bunch of them for quality reasons. However, if you would rather download many fewer images (saving a lot of time and bandwidth), then you can arrange to only download the images according to the geographic specification that you want. In our case, we have always looked for the SVI that is closest to a network of points that is laid out so that there is a point every X meters along every road, path or way. If you wish to use the same specification then you can use the `make_street_points.py` script to generate a GeoJSON (or other format) file that contains points within the defined region that are spaced at every X meters along every road, etc.

Example: generate points within the region defined by `myconfig.json` (same as above), with coordinate reference system SRID 28992, at intervals of 50 meters, and saved into file `mygrid.geojson`:

`./make_street_points.py -c myconfig.json -S 28992 -I 50 -o mygrid.geojson`

Next, you should obtain the Mapillary metadata using the `--tiles-only` option to `mapillary_jpg_download.py`. This will not download JPG files, it only gets GeoJSON files that contain all the metadata about the SVIs, and saves them in the aforementioned `tile_cache_dir`.

`./mapillary_jpg_download.py -c myconfig.json --tiles-only`

Now we will use PostgreSQL (v 13+)/PostGIS to do the heavy-duty geographic processing. This tutorial calls the working database `percept-preprocess` but you can use any name: *(The following examples use `bash` syntax to set environment variables, if you are using another shell then be aware and use their syntax instead)*

    export DB=percept-preprocess
    sudo -u postgres createdb -O $USER $DB
    sudo -u postgres psql -c 'CREATE EXTENSION postgis;' $DB

Import `mygrid.geojson` using `ogr2ogr` into a table named `mygrid`:

    export GRIDTABLE=mygrid
    ogr2ogr -f PostgreSQL "dbname=$DB" mygrid.geojson $GRIDTABLE

Import the tiles database into a table named `mytiles`:
    export TILESTABLE=mytiles
    for f in <tile_cache_dir>/*; do ogr2ogr -append -f PostgreSQL "PG:dbname=$DB" $f -nln $TILESTABLE; done

For convenience we are defining an env var named $SRID, but again you must
select the right SRID for your country/region. In the case of the Netherlands
we are using SRID=28992.

    export SRID=<srid>

Add local geometry columns and indices:

    psql $DB <<EOF
    ALTER TABLE $GRIDTABLE ADD COLUMN geo GEOMETRY(POINT, $SRID);
    ALTER TABLE $TILESTABLE ADD COLUMN geo GEOMETRY(POINT, $SRID);
    CREATE INDEX ${GRIDTABLE}_geo_idx ON $GRIDTABLE USING GIST(geo);
    CREATE INDEX ${TILESTABLE}_geo_idx ON $TILESTABLE USING GIST(geo);
    UPDATE $GRIDTABLE SET geo=ST_Transform(wkb_geometry, $SRID);
    UPDATE $TILESTABLE SET geo=ST_Transform(wkb_geometry, $SRID);
    EOF

Perform a geo-join to find the relevant Mapillary image IDs that we want to download (might take a while):

    psql $DB <<EOF
    CREATE TABLE myresults AS
    WITH ranked AS (
        SELECT
            g.ogc_fid as grid_id,
            p.ogc_fid as point_id,
            p.id as imgid,
            p.geo as point_geo,
            ST_Distance(g.geo, p.geo) as distance,
            FIRST_VALUE(p.ogc_fid) OVER (PARTITION BY g.ogc_fid ORDER BY ST_Distance(g.geo, p.geo)) as closest_point_id
        FROM $GRIDTABLE g
        JOIN $TILESTABLE p ON ST_DWithin(g.geo, p.geo, 100)
    )
    SELECT DISTINCT grid_id, point_id, imgid, point_geo, distance
    FROM ranked
    WHERE point_id = closest_point_id;
    EOF

Extract the results:

    psql -At -c 'SELECT imgid FROM myresults;' $DB > my_imgid_list.txt

Now you can kick-off the `mapillary_jpg_download.py` process with a set of specific image IDs to fetch:

`./mapillary_jpg_download.py -c myconfig.json --imgid-file my_imgid_list.txt`

The output may look like this, and that is normal:

    [...]
    Image ID 802315961410511 is not in the --imgid-file list, skipping.
    Image ID 508836514276730 is not in the --imgid-file list, skipping.
    Downloading: sequence vOyrCjofpkzRS2PYDLNtbG, image ID 1311892379571420...

It will still take a while if your region is more than trivial, but it will take much less time than getting everything.


# FAQ

## The script stops running and tells me it is retrying...

Occasionally Mapillary will stop responding, or for some reason a particular image is not available, therefore our scripts back off and wait a while before resuming. By default this happens up to 8 times before giving up and moving to the next image. Most of the time it starts working again after a few minutes. If it fails entirely you can have the script keep track of which image IDs failed by supplying the `--failed-imgid-file list-of-failed-images.txt` option to store the list of failed image IDs in, for example, the `list-of-failed-images.txt` file. You can adjust the number of retries with the `--num-retries` option, e.g. `--num-retries 5` to stop after five retries.

## The script told me there wasn't enough disk space

By default the script won't run if there isn't at least 100GB available on the filesystem where you are storing image sequences. This is to make sure it doesn't eat up all your working space unexpectedly. You can adjust this minimum using the `--required-disk-space` option, which takes a number in gigabytes, e.g. `--required-disk-space 50` to lower the minimum to 50GB.

## My token file is somewhere else

Use the `--token-file /path/to/my/token.txt` to give a specific location. It is possible to specify the token directly on the command line using `--token` but we do not recommend this because commands are often stored in a command-line history and the API key is like a password. If you must, then clear your history after running the command (e.g. in `bash` that is the `history -c` command).

## Forget about resuming where I left off, I just want it to overwrite everything in the directories!

Then use the `--overwrite` option.
