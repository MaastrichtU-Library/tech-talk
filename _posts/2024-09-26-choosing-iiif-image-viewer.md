---
layout: post
read_time: true
show_date: true
title: Choosing an IIIF-Compliant Image Viewer - Mirador vs. Universal Viewer
description: TODO
date: 2024-09-26
img: posts/choosing-iiif-viewer-web.jpg
tags: [ omeka s, iiif ]
author: Maryam Mazaheri
---

## Introduction

When integrating IIIF capabilities into our Omeka S environment, we needed to choose the right viewer to ensure optimal
user experience for accessing digital collections. After extensive testing of the available options, we selected
**Mirador** over **Universal Viewer** for a number of compelling reasons:

### 1. Faster Image Loading

Mirador outperformed Universa Viewer in loading high-resolution images, providing a smoother user experience.

### 2. Modern Interface

Its clean, modern design offers easy navigation and a visually engaging way to explore digital collections.

### 3. Advanced Features

Mirador supports useful tools like annotations, **OCR overlays**, and image adjustments (brightness, contrast),
enhancing research capabilities.

## Mirador Technical Considerations

Mirador allows users to add annotations directly within the viewer, but these are stored locally and can be lost if
cookies are deleted. Consequently, we have found Mirador's annotation functionality too limited for our
needs. Client-side annotations are not persistent, and there is no established contribution or curation workflow. We
recommend exploring alternative systems for annotation, such as [**Madoc**](https://docs.madoc.io/), which appear more
promising in this regard.

## Conclusion

For a project that depends heavily on user interaction and the flexibility to tailor the viewer to specific use cases,
Mirador is the clear choice. Its customization capabilities make it particularly suited for projects requiring advanced
image handling and specific interaction functionalities.
