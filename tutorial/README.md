# Configuring, preparing and running a human perception survey

## Before getting started

We assume some basic familiarity with Linux and being able to run commands from the shell. Being able to copy-paste examples from here, and adjust them slightly to fit your needs (as described), is sufficient. It may be possible to do everything within other compatible systems such as BSD or Windows Subsystem for Linux but for the sake of simplicity we will focus here on Linux as generically as possible. Packages from Debian or Redhat may be mentioned, for convenience, and the shell is assumed to be `bash` but commands should largely work regardless of which shell you are using.

### Mapillary API key

You will need a Mapillary developer API key in order to run the script that downloads imagery. This can be obtained free-of-charge from the [Mapillary Developer Dashboard](https://www.mapillary.com/dashboard/developers). You will need to register a Mapillary account first, and then you can register an application on the dashboard. Please simply choose something reasonably descriptive for the App Name and Description. Remember that you are ultimately responsible for using the developer access according to the [terms of service](https://www.mapillary.com/terms); read everything of course, but in particular, consider section 11 **Additional Terms for Developers**. Our scripts are provided as-is and come with NO warranty and NO guarantee. To the best of our knowledge these scripts comply with the terms of service for non-commercial usage, provided that you attribute the downloaded imagery to Mapillary if and when you serve them from your own servers.

In the end, you will need to copy the text in the field `Client Token` from the developer dashboard and save it in a file, we recommend calling that file `token.txt` and putting that file in the `percept-vsvi-filter` directory that you will `git-clone` in the next step, where the `mapillary_jpg_download.py` script lives.

## Downloading the imagery from Mapillary

Please see [downloading.md](downloading.md).

## Processing and filtering the imagery

## Configuring the backend

## Configuring the frontend

(under construction)

