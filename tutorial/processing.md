# Processing the downloaded imagery

We assume you have downloaded imagery as per the instructions in [downloading](downloading.md) and you have the imagery stored in the directory `<seqdir>`.

## torch_segm_images.py -- Image Segmentation

You may run our script to analyze all the imagery and produce image segmentation results for each image with the following command, which assumes you wish to use a GPU to speed things up:

`./torch_segm_images.py --gpu -v -r <seqdir>`

The command recursively works through all the images inside the `<seqdir>` directory. This may take several hours depending on how many images you have. For example, we downloaded about 700,000 images from the Amsterdam area. Each image took about a tenth of a second to process with our GPU, therefore the entire process took about a day to complete. You can find out how many images you have by running the command `find <seqdir> -name '*.jpg' | wc -l`.

It is possible to run this without a GPU but we highly advise against it, as the processing time will be heavily increased.

The end result will be that `<seqdir>` will be populated with `.npz` (compressed numpy matrix data) files alongside each `.jpg` image file.

The process is interruptible and restartable. You can press Control-C to interrupt it. When you restart the command, it will find where it left off and pick up from there, unless you specify the `--overwrite` option.

Once finished you should create a list of all the `.npz` files and save it in a text file (one name per line). This can be done easily using `find <seqdir> -name '*.npz' > list-of-npz-files.txt`, for example.
