---
title: About
layout: default
nav: true
---

# sphere-view

`sphere-view` uses [Pannellum](https://pannellum.org/) photosphere viewer, in a minimal Jekyll template on GitHub Pages.
Pannellum viewer supports images up to 4096px wide on mobile browsers, or a max of 8192 px wide on desktop.

## Photospheres

I like to incorrectly use photo features to see what interesting stuff happens.

Photospheres are just regular flat JPG images, but the recorded image is a projection (probably "equirectangular")--just like a map is a flat representation of a globe.
Photosphere viewers know how to display the image to correctly unwarp the projection to make it appear like a 360 degree view. 
The viewer relies on some embedded metadata to tell it that the JPG could be viewed in this way.

*The upshot:* if we add the correct metadata to any random image, we can trick viewers into displaying the image in 360, with odd and fascinating results.

As far as I can tell, most viewers require four XMP tags in the GPano namespace (it is similar on other platforms), which in xml would look like:

```
<GPano:FullPanoHeightPixels>5000</GPano:FullPanoHeightPixels>
<GPano:FullPanoWidthPixels>10000</GPano:FullPanoWidthPixels>
<GPano:ProjectionType>equirectangular</GPano:ProjectionType>
<GPano:UsePanoramaViewer>True</GPano:UsePanoramaViewer>
```

> Hint: if you use Pannellum as in `sphere-view` no metadata tags are required, you will only have to do this if you are trying to trick Flickr, Facebook, etc...

## ExifTool

To manipulate image metadata you need [ExifTool](https://sno.phy.queensu.ca/~phil/exiftool/). 
Truthfully, nothing else works reliably. 

> On Linux install ExifTool as the Perl library, e.g. `sudo apt install libimage-exiftool-perl`.
> On Windows download the standalone package.

To use ExifTool, we start with the command `exiftool` and the file name of an image.
For example, the command `exiftool test.jpg` will print out all the current metadata embedded in test.jpg.

From there we add flags to the command to add or change the image's metadata. 
For example, the command `exiftool -all= test.jpg` will delete all embedded metadata, i.e. replacing all values with nothing.

To figure out how to add the photosphere tags dig through the [XMP Tags docs](https://www.sno.phy.queensu.ca/~phil/exiftool/TagNames/XMP.html#GPano) to find the correct keys.
For example, to build up the command flag we want to add XMP metadata (`-XMP`), from GPano namespace (`-GPano`), in the tag ProjectionType (`:ProjectionType`), with the value of "equirectangular" (`="equirectangular"`).
Thus to add/change that one metadata value, the full command would be:
`exiftool -XMP-GPano:ProjectionType="equirectangular" test.jpg`.
We can add multiple tags simply by adding more flags. 

## Fake a Photosphere

Here are the steps:

1. edit image to 2:1 ratio (for reference, Pixel 2 camera shoots real photospheres around 9000px width by 4500px height).
2. remove all existing image metdata: `exiftool -all= test.jpg`
3. add the new fake photosphere metadata (be sure to replace the pixel dimensions with the actual size of your image): `exiftool -XMP-GPano:FullPanoHeightPixels="4500" -XMP-GPano:FullPanoWidthPixels="9000" -XMP-GPano:ProjectionType="equirectangular" -XMP-GPano:UsePanoramaViewer="True" test.jpg`
4. upload to Flickr or other viewer
