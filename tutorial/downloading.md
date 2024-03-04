# Getting started

We recommend having a terabyte available if you plan to download all the imagery within, for example, a 25km x 25km region around a city like Amsterdam. In fact, many cities will have much less; Amsterdam enjoys a very large and dense dataset of panoramic imagery, which tends to take up much more disk space than ordinary photographs. Please investigate your region of interest on the [Mapillary map app](https://www.mapillary.com/app/) to get a feel for how much imagery and how much panoramic imagery is available. We will assume that you are cloning the scripts and working within the drive that has the available space, therefore our examples will use relative paths, but please feel free to use whichever paths make sense for your system.

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

# FAQ

## The script stops running and tells me it is retrying...

Occasionally Mapillary will stop responding, or for some reason a particular image is not available, therefore our scripts back off and wait a while before resuming. By default this happens up to 8 times before giving up and moving to the next image. Most of the time it starts working again after a few minutes. If it fails entirely you can have the script keep track of which image IDs failed by supplying the `--failed-imgid-file list-of-failed-images.txt` option to store the list of failed image IDs in, for example, the `list-of-failed-images.txt` file. You can adjust the number of retries with the `--num-retries` option, e.g. `--num-retries 5` to stop after five retries.

## The script told me there wasn't enough disk space

By default the script won't run if there isn't at least 100GB available on the filesystem where you are storing image sequences. This is to make sure it doesn't eat up all your working space unexpectedly. You can adjust this minimum using the `--required-disk-space` option, which takes a number in gigabytes, e.g. `--required-disk-space 50` to lower the minimum to 50GB.

## My token file is somewhere else

Use the `--token-file /path/to/my/token.txt` to give a specific location. It is possible to specify the token directly on the command line using `--token` but we do not recommend this because commands are often stored in a command-line history and the API key is like a password. If you must, then clear your history after running the command (e.g. in `bash` that is the `history -c` command).

## Forget about resuming where I left off, I just want it to overwrite everything in the directories!

Then use the `--overwrite` option.
