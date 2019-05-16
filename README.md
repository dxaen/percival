# Percival: Making In-browser Perceptual Ad Blocking Practical with Deep Learning
In this project, we present Percival, a browser-embedded, lighweight, deep learning powered ad blocker. Percival is built into [Blink](https://www.chromium.org/blink) - Chromium Rendering Engine. Percival embeds itself into the image rendering pipeline, which makes it possible to intercept rendering of iframes, images created by complex JavaScript transformations as well as Gifs and regular images. Percival inspects these image frames and performs blocking based on deep learning image classification.

- [Overall Architecture](#overall-architecture)
- [Install and Run Percival](#running-percival)
     - [Run Time Performance Evaluation](#run-time-performance-evaluation)
- [Browsing with Percival](#browsing-with-percival)
- [Google Image Search Results](#google-image-search-results)
     - [Images with High ad intent](#images-with-high-ad-intent)
     - [Images with low ad intent](#images-with-low-ad-intent)
     - [Neutral Images](#neutral-images)
- [Detailed Architecture](#detailed-architecture)
- [Acknowledgement](#acknowledgement)
### Overall Architecture


<img src="https://github.com/dxaen/percival/blob/master/screenshots/Arch.png" width ="850"/>
Percival is positioned in the renderer process-which is responsible for creating rasterized pixels from
HTML, CSS, JavaScript. As the renderer process creates the DOM and decodes and rasterizes all image frames,
these are first passed through Percival. Percival blocks the frames that are classified as ads. The
corresponding output with ads removed is shown above(right)


### Running Percival
### System Requirements

This code was tested with Chromium 74.0.3691.0(64 bit) on Ubuntu(16.04), MacOSX High Sierra 10.13.6.

```
* A 64 bit intel machine with 8GB of RAM. 16GB is recommended.
* 100 GB free disk space
* Python v2
```
#### Clone the depot_tools repository and add to path.

```
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH="$PATH:/path/to/depot_tools"
```
#### Download chromium code and install additional build dependencies and run chromium-specific hooks

```
mkdir chromium && cd chromium
fetch --nohooks chromium
cd src
./build/install-build-deps.sh
gclient runhooks
```
#### Set up the build
```
gn args out/fastbuild
```
This will open an editor, add the following lines to the file
```
is_debug = false
is_component_build = true
use_jumbo_build = true
symbol_level = 0
enable_nacl = false
```
After you exit out, it will generate the build files. For this project, we didn't use Icecc distributed compiler. We didn't use ccache either.

#### Build Chromium
```
autoninja -C out/fastbuild chrome
```
#### Apply darknight.patch
Copy darknight.patch from /patches/darknight.patch to src directory
```
git apply darknight.patch
```
#### Place the model in the Home directory for the browser to load.
In Browser model coming soon. TODO

#### Run Percival reinforced Chromium, need to pass no-sandbox flag
```
./out/fastbuild/chrome --no-sandbox
```
## Run-Time Performance Evaluation
To evaluate the performance of our system, we used top 5,000 URLs from Alexa to test against Chromium
compiled on Ubuntu Linux 16.04, with and without our system activated. We also tested Percival in [Brave](https://www.brave.com), a
privacy-oriented Chromium-based browser, which blocks ads using filter lists by default. For each experiment we
measured render time. In our evaluation we show an increase of 178.23 ms of render time when running Percival in the rendering critical path of Chromium and 281.85 ms when running inside Brave browser with ad-blocker and shields turned on.

This delay is a function to the number and complexity of the images on the page and the time the classifier takes to classify each of them. We measure the rendering time impact when we classify each
image synchronously.

### Render Time evaluation in [Chromium](https://www.chromium.org) and [Brave browser](https://www.brave.com).

<img src="https://github.com/dxaen/percival/blob/master/screenshots/Perf.png" width ="700"/>

## Browsing with Percival

Results from browsing the web with Percival. Percival can block ads and sponsored content from popular websites.

<img src="https://github.com/dxaen/percival/blob/master/screenshots/facebook.png" width ="800"/>

#### Browsing [yahoo.com](https://www.yahoo.com) with Percival

<img src="https://github.com/dxaen/percival/blob/master/screenshots/yahoo1.png" width ="800" />

#### Browsing [cnn.com](https://www.cnn.com) with Percival

<img src="https://github.com/dxaen/percival/blob/master/screenshots/cnn1.png" width ="800" />

## Google Image Search Results

#### Images-With-High-Ad-Intent
We used Google Images as a way to fetch images from distributions that have high or low ad intent. For example, we fetched the results with the search query "Advertisement".

<img src="https://github.com/dxaen/percival/blob/master/screenshots/Advertisement.png" width ="800"/>

#### Images-With-Low-Ad-Intent

Google image search results for query "Obama"

<img src="https://github.com/dxaen/percival/blob/master/screenshots/Obama.png" width ="800"/>

Google image search results for query "Liverpool"
<img src="https://github.com/dxaen/percival/blob/master/screenshots/Liverpool2.png" width ="800"/>

#### Neutral Images

Google Image Search Results for query "Pastry"

<img src="https://github.com/dxaen/percival/blob/master/screenshots/Pastry.png" width ="800"/>

Google Image search results for query "Coffee"
<img src="https://github.com/dxaen/percival/blob/master/screenshots/Coffee.png" width ="800"/>


## Detailed Architecture
Any web page can be thought of as a collection of HTML, CSS, and JavaScript code which is delivered over the network;
the rendering engine parses this code and builds the DOM tree and Layout Tree and issues the OpenGL calls via [Skia](https://skia.org)
(Google Graphics library)

The browser, having built the DOM tree and processed the style-sheets calls the rendering engine next to determine
the visual geometry of all the elements, i.e. compute the coordinates of the rectangles corresponding to the regions
these elements occupy on the screen; this is called layout stage, and the information is stored in the layout tree.
Once the geometry of the objects is known, blink paints these elements, i.e. recording a paint operation in a list of
display items (an abstraction for objects the user will see in the content area).

This is followed by the rasterization process, which takes these display items and turns them into bitmaps. Rasterization
issues OpenGL draw calls via the Skia library which abstracts hardware operations.


<img src="https://github.com/dxaen/percival/blob/master/screenshots/DetailArch.png" width ="800"/>

## Acknowledgement
We would like to thank [Steven Kobes](https://stevenkobes.com/about) [Vladimir Levin](https://www.chromium.org/teams/rendering) [Aleksandar Stojiljkovic](https://01.org/users/astojilj) and the entire [Chromium Graphics Team](https://www.chromium.org/teams/rendering) for the extensive documentation, presentations and google docs detailing various parts of the graphics pipeline.

We would also like to thank [Tobias Hermann](https://github.com/Dobiasd) for [Frugally Deep](https://github.com/Dobiasd/frugally-deep). Percival uses a fork of [Frugally Deep](https://github.com/Dobiasd/frugally-deep).

