---
layout: post
read_time: true
show_date: true
title: IIIF and Omeka S - Lessons Learned
description: Lessons learned during Maastricht University Library's implementation of IIIF in Omeka S
date: 2024-09-19
img: posts/lessons-learned-web.jpg
tags: [ omeka s, iiif ]
author: Maryam Mazaheri
---

### 1. Install and switch to 'vips' from ImageMagick for faster image processing in Omeka S.
Read more on: [Power up! Optimize IIIF user experience](https://maastrichtu-library.github.io/tech-talk/optimize-iiif-performance.html)

### 2. Use 'Tiled tiff' as the tiling method to ensure correct image display.
Read more on: [Power up! Optimize IIIF user experience](https://maastrichtu-library.github.io/tech-talk/optimize-iiif-performance.html)

### 3. Prefer JPEG quality 80 as the source image type for the best balance of performance, disk space usage, and viewing quality.
Avoid JPEG 2000 and uncompressed TIFF due to performance issues and high disk space consumption.

Read more on: [Power up! Optimize IIIF user experience](https://maastrichtu-library.github.io/tech-talk/optimize-iiif-performance.html)

### 4. Avoid relying on client-side annotations in the Mirador viewer due to their non-persistent nature and lack of contribution or curation workflows in Mirador.
Instead, we recommend exploring alternative annotation systems like Madoc.

Read more on: [Choosing an IIIF-Compliant Image Viewer: Mirador vs. Universal Viewer](https://maastrichtu-library.github.io/tech-talk/choosing-iiif-image-viewer.html)

### 5. Manually adjusting visual settings like brightness and contrast on each page in the viewer is inefficient.
Implementing global settings within the viewer enhances user efficiency. This feature is work in progress.

### 6. Use smaller batch sizes when adding bulk imports of large datasets into Omeka S to improve processing speed and reduce system slowdowns.
**Regularly monitoring logs** helps detect any import issues early and ensures data integrity.

### 7. Batching large uploads, monitoring system resources, and testing under real-world conditions are essential to manage performance with multiple tasks and concurrent users.
