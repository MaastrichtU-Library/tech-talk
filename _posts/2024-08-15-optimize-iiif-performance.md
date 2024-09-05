---
layout: post
read_time: true
show_date: true
title: Power up! Optimize IIIF user experience
description: Configuration settings for a well performant IIIF user experience with Omeka S 
date: 2024-08-15
img: posts/power-up-web.jpg
tags: [ omeka s, iiif ]
author: Maarten Coonen
---

Running Omeka S with the IIIF modules configured on a machine with average hardware leads to a workable situation in most cases. 
However, there are some things you need to know in order to make your user's experience much better. 


## Boost image processing performance with vips
Image files are being preprocessed (resized and tiled) by Omeka S at time of upload so that the IIIF ImageServer can 
optimally serve the tiles whenever an image is requested over HTTP. Omeka S has support for multiple image processing
libraries that are available as PHP extensions. In most cases, GD or ImageMagick are being used, as these are preinstalled by 
most Linux distributions. The [ImageServer module](https://omeka.org/s/modules/ImageServer/) recommends to use the 
[**vips** image processor](https://www.libvips.org/), as this one is much faster.

Our benchmarks confirm that vips is blazingly fast. Processing a 127 MB TIF file during upload on our test Omeka S system
took 15 minutes with Imagemagick and only 1 second with vips! With the JPEG 2000 format the difference is even more extreme:
44 minutes with Imagemagick vs. 1 second with vips.
As one can imagine, these reductions in processing time greatly enhance the productivity of collection managers working
with the system. 

vips is easy to install. Just use the command below to install it from your distro's package manager if you have 
Debian 12 (or higher) or Ubuntu 24.04 (or higher). People running Ubuntu 20.04 can follow the installation instructions
from our [previous blog](./installing-iiif-with-omekas.html).
```bash
# Debian 12 & Ubuntu 24.04
apt-get install libvips-tools
```

## Choose "Tiled tiff" as tiling method
The image preprocessing (resizing and tiling) jobs in Omeka S ensure that (high-resolution) image files are converted 
to a format that can be optimally served over the internet, while still preserving all details and zoomability. One way
to do this is by using a pyramidal image format like [pyramidal TIFF](https://training.iiif.io/iiif-online-workshop/day-two/fileformats.html).

![pyramidal tiff](assets/img/posts/pyramidal-format.png)
A pyramidal image consists of multiple layers, each representing the image in different resolution. At initial load, 
the layer with the lowest resolution (and smallest size) is served to the image viewer. When a user zooms in, a deeper layer
with higher resolution is used to serve only the tiles that are in the viewpane. The rest of the image is not loaded, which
saves network bandwidth and results in a faster loading time.

Source image files that are being uploaded to Omeka S are stored in the `original/` sub folders. The tiled copies are
stored in the `tile/` sub folders.
```bash
./files/original/<itemID>/<filename>.jpg  ------|
                                                |
./files/original/<itemID>/<filename>.jp2  ------|---- Pyramidal TIFF ----> ./files/tile/<itemID>/<filename>.tif
                                                |
./files/original/<itemID>/<filename>.tif  ------|
```

The Omeka S module [ImageServer](https://omeka.org/s/modules/ImageServer/) provides multiple tiling methods:
- Deep Zoom Image
- Zoomify
- Jpeg 2000
- Tiled tiff

Based on our tests, the **Tiled tiff** method works best, as it saves disk storage capacity, has the fastest upload experience
and has a reasonable page loading time. 

Note: although the "Deep Zoom Image" and "Zoomify" options serve images the fastest, they are still not our preferred format
as they cause HTTP-404 errors for tiles that cannot be loaded. 


## Increase page load performance with parallel processing
The time of single core processors has passed a long time ago. Nowadays, all CPUs have 2 or more cores that can do work 
simultaneously. Apache, PHP-FPM and Omeka S are able to work multi-threaded, thereby putting multiple cores to work in
serving the image tiles to the user requesting it. This greatly enhances performance. 

Modern data centers run virtualized servers and allow you to increase the amount of CPU cores with the click of a
button, as long as you continue to pay your bills. Virtualized cores cost money as well!

We've conducted a series of tests with various source file types (JP2000, TIFF, JPG compression quality 80 (JPGq80) and
JPG compression quality 40 (JPGq40)) to determine the optimal amount of CPU cores. Our base line tests were run on 4 CPU
cores. Subsequently, we've repeated the same tests with 8 and 16 cores.

The results are shown in the figure below. On average, 8 CPU cores load a page 31.3% (8 cores) and 36.5% faster 
(16 cores) compared to 4 cores when tested with these file types. We expect that adding more cores will increase performance
even further, although the gain per added core will be less.
![effect of cpu cores]("assets/img/posts/testing-cpu-cores.png")


## TODO: Save disk space and increase page load performance by using JPG compressed source images
tiled tiff from jp2 processed again by vips at page load. WE don't see this with other formats. This is why jp2 performs least.
TIFF format loads fastest, but takes a lot of disk space.
JPGq80 is the best compromise


## Conclusion
Our preferred setup consists of a machine with the characteristics below.

- CPU: 16 cores or more
- Memory: 16 GB or more
- Disk: 250 GB or more
- Operating System: Ubuntu 20.04 or higher
- Web server: Apache 2.4 or higher with HTTP/2 and PHP-FPM modules and CORS headers enabled (see details in [previous blog](./installing-iiif-with-omekas.html))
- PHP: version 8.2 or higher
- Image processing: vips 8.14.1 or higher & Imagemagick 6.9.11 (with jp2 extension enabled) or higher
- Omeka S module Image server - Image processor: Automatic (Vips when possible, ... (etc.))
- Omeka S module Image server - tiling type: Tiled tiff