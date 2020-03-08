# ZClipsy

ZClipsy is an interactive video clip catalog generator.

Given an inventory of YouTube video ids and timestamped descriptions of clips within those videos, ZClipsy generates
a standalone website containing a catalog of the clips for convenient looped playback at different speeds. The clip
catalog is paginated, searchable, and sortable.

I originally created this tool for organizing juggling pattern research clips, but you can use it for anything. It can
support 300+ videos, and 5000+ clips without any known performance problems.

[Here](http://demo-zclipsy.s3-website-us-west-2.amazonaws.com/index.html) is an example generated site from which
the below screenshot was taken.

![Example screenshot](https://raw.githubusercontent.com/noslowerdna/zclipsy/screenshots/example.png "Example screenshot")

## Installation

Only Mac OSX is currently supported for running the catalog site generator, although it should not be terribly difficult
to figure out how to run the code on Linux or Windows. The generated site can be accessed by any modern web browser on any
type of device, including tablets and smartphones.

To download the files, first run these commands to install [Homebrew](http://brew.sh/) and [Git](https://git-scm.com/).

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install git
```

You can then clone this code repository.

```
git clone https://github.com/noslowerdna/zclipsy.git
```

Alternatively, you can [download a ZIP file](https://github.com/noslowerdna/zclipsy/archive/master.zip) of the latest
code and `unzip` it in the directory of your choice.

To generate an example catalog site, just `chmod` the `zclipsy` script to be executable and then run it.

```
chmod u+x zclipsy
./zclipsy
open index.html
```

## Configuration

Three input files are required: a configuration file, a video inventory, and a clip inventory. These files all need to
reside in the same directory as the code.

#### Configuration file

The configuration file is named `zclipsy.yaml`. You can edit it with your favorite text editor application. The
following properties can be configured using standard `property: value` YAML syntax.

* **videos** - Name of your video inventory file. Default: "videos.txt"
* **clips** - Name of your clip inventory file. Default: "clips.txt"
* **title** - Title of the generated HTML page. Default: "Generated by ZClipsy"
* **speed** - Optional speed scaling factor. For example, you can set this to 2 if your videos were filmed at half-speed
or 4 for quarter-speed. Default: 1

#### Video inventory

The video inventory file should contain one or more lines in this format:

```
VIDEO_NAME DATE YOUTUBE_ID LENGTH
```

* **VIDEO_NAME** - Short name for the video, for example `Res-96`. Cannot contain spaces, must be different for each video.
* **DATE** - Video date in `MonYYYY` format, for example `Oct2015`. This is tagged to each clip in the video so you can search
by date.
* **YOUTUBE_ID** - Unique YouTube video identifier, for example `Jjxg6kxJ62Y`.
* **LENGTH** - Video length in `mm:ss` or `hh:mm:ss` format, for example `0:45` (45 seconds), `5:34` (5 minutes, 34 seconds), `2:30:15` (2 hours, 30 minutes, 15 seconds).

#### Clip inventory

The clip inventory file should contain one or more lines in this format:

```
VIDEO_NAME START_TIME DESCRIPTION
```

* **VIDEO_NAME** - Short name of the video containing the clip as listed in the video inventory file.
* **START_TIME** - Clip's start time in `mm:ss` or `hh:mm:ss` format. All clips in a video must be on consecutive lines and listed
sequentially in ascending order. Must be less than the video length as declared in the video inventory file.
* **DESCRIPTION** - Text description of the clip. May contain spaces. A system that works well is for this to be a collection of tags or labels.

## How it works

The ZClipsy generator is primarily written in [Ruby](https://www.ruby-lang.org/).

The generated site is self-contained and consists of JavaScript, CSS, and HTML files.

The [List.js](http://www.listjs.com/) JavaScript library is used to provide pagination, sorting, and searching functionality
for an HTML table of clips. 

[jQuery](https://jquery.com/) is also used for some functionality.

#### Video playback

The [YouTube API](https://developers.google.com/youtube/v3/) is used to play a segment of a video in a repeating loop. The clip
can be played at regular speed, half-speed, or quarter-speed. Internet connectivity is required in order to play clips.

The privacy setting for the YouTube videos can be public, unlisted, or private. However, in order to play private videos,
you'll need to be the video owner and logged in to your YouTube account.

#### Searching

When you enter one or more search terms separated by spaces, the clip table updates itself dynamically as each letter
is typed, filtering out clips that do not match all of the criteria. Some minor enhancements were made to the List.js
search syntax, in particular:

* Searching across all columns in the table
* Support for negation by prefixing a search term with a `-` (for example `-shiny`), which excludes any matching clip
* Support for exact word match by prefixing a search term with a `+` (for example `+shoe` would match "my shoe" but not "my shoes"), note that exact negation is also supported (for example `-+shoe` would exclude "my shoe" but not "my shoes")

#### Sorting

You can sort the clips by description or duration in ascending or descending order, while also retaining any search
filter.

#### Pagination

20 clips are displayed per page (to change that, you can edit the `clipsPerPage` variable value in `pagify.js`). The
current page is highlighted in `[]`. The first, last, and -2 to +2 pages can be navigated to by clicking the page
number link. Updating the search filter resets the clip table to the first page of results.

## Deployment

A number of deployment options are possible.

#### Local

You can view the site on your local machine by opening the `index.html` file in a web browser. Of course, you'll still
need internet connectivity in order to play the videos that are hosted on YouTube's servers. 

#### Deep Web

I originally used [Dropbox](https://www.dropbox.com/) for hosting my personal clip catalog as a [Deep Web]
(https://en.wikipedia.org/wiki/Deep_web_(search)) site. The details for setting this up are outside the scope of this
documentation, but the gist is that it involved copying everything into a [Public](https://www.dropbox.com/en/help/16)
folder. Whenever the files are updated, they are immediately synced to the Dropbox servers. I also use a [TinyURL]
(http://tinyurl.com/) to conveniently access the site since the URL that Dropbox provides is very long and complex.
I switched to hosting it on [Amazon S3](http://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html) after
Dropbox removed support for this functionality in October 2016.

Besides S3 other Deep Web options might include [JSFiddle](https://jsfiddle.net/), [BitBalloon](https://bitballoon.com/),
and [Google Drive](https://www.google.com/drive/) (until [August 31, 2016](https://support.google.com/drive/answer/2881970)
anyway). [Github Pages](https://pages.github.com/) could also be an option.

#### Public Internet

If you have your own website, you can just upload the files to it. Make sure to upload `index.html` (renaming as necessary), `style.css`, and all of the `.js` files including the generated `cliptable.js` file. Keep in mind that with this option the
content could be potentially indexed by search engines. If your videos are set as unlisted to hide them on YouTube, they may
become publicly discoverable.

## Updates

The process for updating your clip catalog site is fairly straightforward.

1. Upload any new video(s) to YouTube, noting the video id(s) that YouTube assigns to them.
2. Edit the video inventory file.
3. Edit the clip inventory file.
4. Run the `zclipsy` script.

The site can then be viewed locally, or the generated `cliptable.js` and `index.html` files can be deployed to a web
server for viewing over the internet as described in the Deployment section above.

To refresh the ZClipsy code itself, simply run `git pull` from the project installation directory or download the master
branch ZIP file. You can watch the [commit history](https://github.com/noslowerdna/zclipsy/commits/master) for changes to
determine whether you want to update the code.
