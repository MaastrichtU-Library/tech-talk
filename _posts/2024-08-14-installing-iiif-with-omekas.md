---
layout: post
read_time: true
show_date: true
title: Installing IIIF to Omeka S
description: How to get Omeka S and IIIF running in production
date: 2024-08-14
img: posts/installing-omekas-iiif.jpg
tags: [ omeka s, iiif, docker ]
author: Maarten Coonen
---

In our [previous post](./getting-started-with-iiif-omekas.html) we talked about a quick way to get IIIF and Omeka S
running locally with Docker. In this post we describe the steps to take when installing IIIF to your Omeka S production
environment, including support for the JPEG 2000 image format. 

### Architecture
There are many ways to work with IIIF and Omeka S and there is a lot of free software available. We chose for an 
architecture in which all IIIF components are located within Omeka S. This has its pros and cons (more on
that in another blog post), but the main reason is because of simplicity in data management. Omeka S is already used 
in our library and our collection managers are familiar with its data entry and data management flows. Introducing an
additional "moving part" to the infrastructure would make things more complicated with little added value (at least for 
the scale we're using Omeka S on).

The architecture used in our Omeka S instance is depicted in the two pictures below.

![architecture 1](assets/img/posts/iiif-architecture-1-web.jpg)
A collection manager enters data and metadata in Omeka S, which is stored on disk (data) and in Omeka S's database (metadata).
Four Omeka S modules (Image Server, IIIF server, IIIF search and Mirador) work together to offer IIIF-functionality. 

![architecture 2](assets/img/posts/iiif-architecture-2-web.jpg)
A user accesses the Omeka S frontend site via a web browser. Data and metadata are combined by internal methods of the four
modules and presented to the user.

This blog describes the actions to take to get this architecture reproduced on your Omeka S production environment.

### Before you start
Make sure you have a working Omeka S instance running on your server. The steps below are written for the software stack:
- Ubuntu 20.04
- PHP 8.2
- Apache 2.4 configured for PHP-FPM

Other operating systems and versions are supported as well, but not described in the scope of this blog. 

### Install general software packages
First, install the required software via the Ubuntu package manager.
`libfreetype6-dev` is a dependency for the gd graphics processing library that Omeka S uses as PHP extension.
The other ones are dependencies for other PHP extensions that Omeka S uses for compression or image processing.
```bash
sudo apt-get install \
    libfreetype6-dev \
    zlib1g-dev \
    libpng-dev \
    libjpeg-dev \
    libtiff-dev
```

### JPEG 2000 compatibility
The default ImageMagick version in Ubuntu 20.04 lacks jp2 support, so we need to compile and install specific versions of
ImageMagick, libvips and libopenjp2 to make it work. 

**If you're using Debian or a higher version of Ubuntu**, you might be able to install the versions from your OS's package
manager directly and can probably continue with the next section **Configure Apache and PHP**.

#### 1. Imagemagick
Build ImageMagick with JP2 support:
```bash
git clone https://github.com/SoftCreatR/imei.git
cd imei 
chmod +x imei.sh
./imei.sh --imagemagick-version 6.9.11-60
```

Install Imagick PHP extension:
```bash
sudo apt-get install php8.2-dev php-pear
sudo pecl install imagick
```

Create config file `/etc/php/8.2/fpm/conf.d/imagick.ini` and add these contents:
```bash
extension=imagick.so
```

Enable the PHP extension
```bash
sudo phpenmod imagick
sudo systemctl reload php8.2-fpm
sudo systemctl restart apache2
```

#### 2. OpenJPEG
`libopenjp2-7-dev` is a library for handling the JPEG 2000 image compression format. `vips` depends on `libopenjp2-7-dev`
version 2.4.0 or higher, whereas 2.3.1 is the highest version provided by the Ubuntu 20.04 package repos. Compiling and 
building this package is rather complex, so we're happy to offer you a .deb package we compiled ourselves.

Download the `libopenjp2-7-dev` 2.5.0 package from our GitHub release
```bash
cd ~
wget https://github.com/MaastrichtU-Library/tech-talk/releases/download/1.0.0-deb/libopenjp2-7-dev_2.5.0-1_amd64.deb
```

Install it to your system with:
```bash
sudo dpkg -i libopenjp2-7-dev_2.5.0-1_amd64.deb
```

Confirm that the correct version is installed with:
```bash
dpkg --list | grep libopenjp2
```

#### 3. libvips
libvips is a fast image processing library with low memory needs ([website](https://www.libvips.org/)). The version
offered by the Ubuntu 20.04 package repos is not compatible with the ImageMagick version we installed above, so we need
to build it from source ourselves. 

Compiling and
building this package is rather complex, so we're happy to offer you a .deb package we compiled ourselves.

Remove the preinstalled version of vips
```bash
sudo apt-get autoremove libvips-tools
```

Install support for GIF using this ppa:
```bash
sudo add-apt-repository ppa:lovell/cgif
sudo apt-get update
sudo apt-get install libcgif-dev
```

Download the `vips` 8.14.1 package from our GitHub release
```bash
cd ~
https://github.com/MaastrichtU-Library/tech-talk/releases/download/1.0.0-deb/vips_8.14.1-1_amd64.deb
```

Install the dependencies with:
```bash
sudo apt-get install ./vips_8.14.1-1_amd64.deb
# Please ignore the error that vips itself could not be installed
```

Install the package with:
```bash
sudo dpkg -i vips_8.14.1-1_amd64.deb
```

Confirm that vips is installed and returns no errors about missing dependencies.
```bash
which vips
vips --version
dpkg --list | grep vips
```

### Configure Apache and PHP
For optimal performance of the IIIF functionalities, we need to enable HTTP/2 and set CORS headers in the Apache web
server.

Enable the two Apache modules
```bash
a2enmod http2
a2enmod headers
```

Set the CORS headers in the virtual host config file, such as `/etc/apache2/sites-available/YOUR_SITE_NAME.CONF`
```apacheconf
# CORS access for some files. (e.g. serving IIIF images to external clients)
<IfModule mod_headers.c>
	Header setIfEmpty Access-Control-Allow-Origin "*"
	Header setIfEmpty Access-Control-Allow-Headers "origin, x-requested-with, content-type"
	Header setIfEmpty Access-Control-Allow-Methods "GET, POST"
</IfModule>
```

Restart the Apache web server
```bash
sudo systemctl restart apache2
```

In order to prevent errors like 'CSRF: Value is required and can't be empty' when editing Omeka S items consisting of many
media files, you need to increase maximum allowed input.

Edit the file `/etc/php/8.2/fpm/conf.d/99-omeka.ini` and insert the following
```ini
max_input_vars = 3000
max_multipart_body_parts = 3000
```

Restart PHP-FPM with
```bash
sudo systemctl restart php8.2-fpm.service
```


### Install Omeka S modules for IIIF

Install the following Omeka S modules using your preferred installation approach. 

- [**ImageServer**](https://github.com/Daniel-KM/Omeka-S-module-ImageServer/releases) version 3.6.17. 
This module implements the [IIIF Image API](https://iiif.io/api/)
- [**IiifServer**](https://github.com/Daniel-KM/Omeka-S-module-IiifServer/releases) version 3.6.16. 
This module implements the [IIIF Presentation API](https://iiif.io/api/) 
- [**IiifSearch**](https://github.com/smachefert/Omeka-S-module-IiifSearch/releases) version 3.4.7.
This module implements the [IIIF Content Search API](https://iiif.io/api/)
- [**Mirador**](https://github.com/Daniel-KM/Omeka-S-module-Mirador/releases) version 3.4.7.16 or
[**UniversalViewer**](https://github.com/Daniel-KM/Omeka-S-module-UniversalViewer/releases) version 3.6.9 as the IIIF-
compliant image viewer.

**Remark**. Please be aware that this is a proven and working combination of module versions. Deviating from these
versions might result in broken functionality. For instance, version 3.6.18 of IiifServer is known to break the
Mirador's OCR helper plugin and versions 3.6.19 and 3.6.20 break IiifSearch functionality. Other "golden combinations"
might exist and we hope that these bugs appear less in future versions of these modules.


### Configuration in Omeka S
Some settings have to be made in Omeka S to make everything run smoothly. Login to your Omeka S admin backend and 
navigate to the following sections.

**Site settings**
Repeat this for every site in your Omeka S instance
- Uncheck “Show - Embed media on item pages (legacy)”
- Enable the following plugins at “Players - Mirador plugins for v3“:
  - Download files 
  - Image tools
  - OCR helper
  - Share
- Add the following json-content at “Players - Mirador config as object string for v3 (item)“
```json
{
    "window": {
	    "imageToolsEnabled": true,
		"imageToolsOpen": true
    },
    "thumbnailNavigation": {
        "defaultPosition": "far-bottom",
        "displaySettings": true
    }
}
```

**Settings**
- Add `application/alto+xml` to "Allowed media types"
- Add `xml` to “Allowed file extensions”

**Module settings**
- ImageServer
  - Set the tiling method to “Tiled tiff”
- IiifServer
  - Set “default manifest version” to 2
  - Uncheck “Cache manifests for instant access (require module Derivative Media)”



### Final words
Following this extensive manual has hopefully resulted in a working IIIF-OmekaS instance. Now you can start your 
IIIF-journey by creating new items in Omeka S and uploading image files to it. The IIIF-manifests will be populated with
the item- and media-metadata you enter in Omeka S. Mirador or Universal will render the images you've attached to the item.

Good luck and have fun! 

Some sample URLs:
- Frontend view: https://digitalcollections.library.maastrichtuniversity.nl/s/medicine/item/3525
- IIIF manifest: https://digitalcollections.library.maastrichtuniversity.nl/iiif/2/3525/manifest

![herbarius](assets/img/posts/herbarius-web.jpg)