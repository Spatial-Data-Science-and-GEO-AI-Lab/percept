# Processing the downloaded imagery

## Parameters

We assume you have downloaded our software and imagery as per the instructions in [downloading](downloading.md), that you are working within the `percept-vsvi-filter` directory with our software, and you have the imagery stored in the directory `<seqdir>` and the Mapillary tiles GeoJSON cache stored in `<tile_cache_dir>`. For the first step below (image segmentation) you only need to know `<seqdir>`, which you should already have, so you can get image segmentation started while deciding on the rest of these parameters below.

You are eventually going to need to decide where imagery is going to live in your filesystem and also where on your website it is going to be accessible. We call these the `<system_path_base>` and `<web_path_base>` pathname prefices, respectively. For example, if you have set aside a data partition named `/data` on your filesystem then it may be a good idea to set the `<system_path_base>` to '`/data/img`'. Then you can configure your web server (later) to serve all the imagery from locations under `/img` of your website, so then a reasonable value for `<web_path_base>` would be '`/img`'.

You will also need to pick a short name that we will call `<cityname>` for the city or region that you are covering; this is mainly used for organizing the imagery in the filesystem and the database, so a short and simple name will do. For example, if you are covering Amsterdam then perhaps 'ams_nl', or for New York City then 'nyc_ny_usa' would be a reasonable value for `<cityname>`. However, in the end, it only needs to be unique amongst the projects that you might be conducting on the same system, so feel free to pick something simpler if there are no conflicts. In the end, the system will expect to find imagery within the filesystem (and web URLs) at `<system_path_base>/<cityname>` (and `<web_path_base>/<cityname>` for web URLs) so be aware that the identifier you choose will form part of the pathname of every image that you serve.

You will also need a place to temporarily store thousands of generated SQL files (probably adding up to no more than a few dozen megabytes). We call this temporary directory `<sqldir>` and often simply just set it to `sqldir/` relative to the current working directory.

## torch_segm_images.py -- Image Segmentation

You may run our script to analyze all the imagery and produce image segmentation results for each image with the following command, which assumes you wish to use a GPU to speed things up:

`./torch_segm_images.py --gpu -v -r <seqdir>`

The command recursively works through all the images inside the `<seqdir>` directory. This may take several hours depending on how many images you have. For example, we downloaded about 700,000 images from the Amsterdam area. Each image took about a tenth of a second to process with our GPU, therefore the entire process took about a day to complete. You can find out how many images you have by running the command `find <seqdir> -name '*.jpg' | wc -l`.

It is possible to run this without a GPU but we highly advise against it, as the processing time will be heavily increased.

The end result will be that `<seqdir>` will be populated with `.npz` (compressed numpy matrix data) files alongside each `.jpg` image file.

The process is interruptible and restartable. You can press Control-C to interrupt it. When you restart the command, it will find where it left off and pick up from there, unless you specify the `--overwrite` option.

Once finished you should create a list of all the `.npz` files and save it in a text file (one name per line). This can be done easily using `find <seqdir> -name '*.npz' > list-of-npz-files.txt`, for example.

## make_tiles_db.py -- Build a quick-look-up database of the tile GeoJSON information

We recommend taking a moment to build a tile cache database using the `make_tiles_db.py` script, to speed up subsequent steps. Just run a command like:

`./make_tiles_db.py -o tiles.pkl <tile_cache_dir>`

Then the tile cache database will be stored in `tiles.pkl` (or whichever filename you choose to use).

## torch_process_segm.py -- Find road center-lines and analyze image quality

### Set-up

If you have not yet installed all the dependencies using `pip install -r requirements.txt` and you are not using the [Docker option](https://github.com/Spatial-Data-Science-and-GEO-AI-Lab/percept-vsvi-filter#alternative-docker-set-up), then now is the time to install those dependencies. Manually, that means `pip install numpy torch transformers scipy scikit-image opencv-python-headless`.

### Getting a list of `.npz` files

If you did not make a list of `.npz` files in a previous step then we advise doing it now, using a command like `find <seqdir> -name '*.npz' > list-of-npz-files.txt` to make a list of filenames, one per line, in a file named `list-of-npz-files.txt` (or whatever name you like). For the time being, the following script also expects that each `.npz` file sits alongside its source `.jpg` file in the same directory, which is where the `torch_segm_images.py` script outputs by default, so this should not be an issue unless you move things around.

### Processing the segmented imagery

Given that you have chosen suitable values for `<system_path_base>`, `<web_path_base>`, `<cityname>` and `<sqldir>` as described in the Parameters section above, and assuming you have built a `tiles.pkl` and `list-of-npz-files.txt` (or whichever names you chose), then simply run:

`./torch_process_segm.py -v --log --fast -T tiles.pkl -D <system_path_base> -U <web_path_base> -C <cityname> -S <sqldir> -F list-of-npz-files.txt`

This will take a while (again, on 700,000 images, it took most of a day) and will produce, for each input `.npz` file, several outputs:
- Alongside each `.npz` file will now sit a `.out` file as well (thanks to the `--log` option), and this `.out` file contains image quality and road center information that we will use later.
- Panoramic imagery will also be cropped according to the procedure described in [our paper](https://arxiv.org/abs/2403.00174), section 3.2 'Imagery downloading and processing' and the resulting output `.jpg` files can be found alongside the source `.jpg` and `.npz` files. They will have names of the following format: `<orig_imgid>_x<number>.jpg`, where `<orig_imgid>` is the original image ID from the source `.jpg` file and `<number>` corresponds to the X-coordinate of the left-hand side of the cropped sub-image within the original image.
- The `<sqldir>` will contain two `.sql` files for each input `.jpg` (in this case, that means the cropped sub-images of panoramic imagery, not the original panoramic image itself; in other words, there will be SQL information for every possible candidate image that could be used for the survey). Those SQL files will have names ending in either `_insert.sql` or `_enable.sql`. The insert-files contain the information needed to populate the backend database with information about every image, but by default they will leave the images *disabled* for the purposes of the survey. Later, when you decide which images to actually use in the survey, the enable-files allow you to quickly enable the associated images. In any case, everything in the `<sqldir>` can be ignored for now and saved until you reach the database backend set-up stage.

Since this is a long-running process (albeit not as long as downloading, usually), it is interruptible and restartable. If you need to interrupt it, press Control-C. When you restart it, the script will automatically skip over files that have already been processed. There will be warnings about the `.sql` and cropped image files already existing -- these can be ignored. If you wish to overwrite these files, then just add `--overwrite` to the command line options.
