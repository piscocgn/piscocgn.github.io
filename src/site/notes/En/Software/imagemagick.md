---
{"dg-publish":true,"dg-permalink":"en/software/imagemagick","permalink":"/en/software/imagemagick/"}
---

#dtp #console #linux
### TIFF
Convert a multipage TIFF-file into single TIFF-files:

`convert multipage-file.tiff singlepage-split-%d.tiff`

Convert single TIFF-files into a multipage TIFF-file:

`convert singlepage-split-*.tiff -adjoin multipage-file.tiff`

