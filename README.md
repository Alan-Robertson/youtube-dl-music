<!-- ## Be Careful! -->
<!-- Downloading content from the internet for personal use (not distribution) is not illegal (criminal law or copyright infringement); but by using this script, you are breaking Youtube's Terms of Service (civil law). Then again, that's a problem for the `youtube-dl` devs, not us :) -->
## Overview
<!-- [![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](lukelbd@gmail.com) -->
This script uses `youtube-dl` to download and save audio from youtube into `aac`/`m4a` files (the native format of youtube audio), normalizes the audio/volume using `ffmpeg-normalize`, and adds metadata tags using a python script I created called `metadata`.

## Installation and Setup
This library is intended for UNIX shells -- i.e. MacOS, Ubuntu, perhaps the Windows 10 UNIX terminal (untested). Install this library by navigating to your **home** directory in the terminal, and entering

    git clone https://github.com/lukelbd/youtubetag

This should create a directory named `youtubetag`. You then need to make sure the `youtube` and `metadata` executables are in your `$PATH` variable. This is up to you; the simplest option may be to place in your `.bash_profile` or `.bashrc` the line 

    export PATH="$HOME/youtubetag:$PATH"

then restart the shell or "source" the file with `source ~/.bash_profile`. Now, the tools you need will be accessible every time you start a terminal.

<!-- Just make -->
<!-- Run `setup` command once. -->
Before using the `youtube` and `metadata` tools, you need to do the following:

  1. Install command-line tools needed for `youtube` (see below).
  2. Make sure your version of `python3` is python3.6+; check this with `python3 --version`. The `metadata` script has f-strings, which are a python 3.6+ feature. f-strings are totally awesome and you should be using them (: .
  3. Run the `setup` command. This does the following:
      * Installs the python packages needed for `metadata` (see below).
      * Creates a file named `config` in the youtubetag directory.
  4. Add the following lines to the file `config` in the format `key = value` (note whitespace doesn't matter, and entries don't need to be quoted):
      * To set the download location: `directory = <your music folder here>`.
      * **Create a Discogs account** and create a token for the API; add that to `config` with `token = <your token here>`.
      * **Create a MusicBrainz account** and the API simply uses your account username and password; add them to `config` with `username = <your username here>` and `password = <your password here>`.

Discogs and MusicBrainz are the two major online discography databases, each with their strengths and weaknesses each with public python APIs. So, why don't we use both? :)

If the `youtube` script **stops working**, it is often because `youtube.com` has changed how they store video/audio. The `youtube-dl` developers are very active and usually will release an updated version within a couple days; just call `youtube-dl -U` ("update") and it should start working again.
<!-- If you just want to download the `m4a` files and do nothing else, comment out parts of the `youtube` script that adjust volume/add metadata. -->

## Command-Line Dependencies
  * [youtube-dl](https://github.com/rg3/youtube-dl): script for downloading youtube media; `brew install youtube-dl` (Homebrew on Mac).
  * [ffmpeg](https://github.com/FFmpeg/FFmpeg): batteries-included package for creating/modifying media files; `conda install ffmpeg` (any anaconda distribution) or `pip install ffmpeg` or `brew install ffmpeg` (Homebrew on Mac).
  * [ffmpeg-normalize](https://github.com/slhck/ffmpeg-normalize): python package for normalizing volume, requires ffmpeg accessible from shell; `pip install ffmpeg-normalize`.

## Python Package Dependencies
  * [mutagen](https://github.com/quodlibet/mutagen): for writing metadata.
  * [musicbrainzngs](https://github.com/alastair/python-musicbrainzngs): for interacting with MusicBrainz database.
  * [discogs_client](https://github.com/discogs/discogs_client): for interacting with Discogs database.
  * [BeautifulSoup](https://pypi.python.org/pypi/beautifulsoup4): for parsing the "release group" HTML webpage and finding any linked discogs "master releases". Necessary because there is no way of retreiving this link from the MusicBrainz API directly.
  * [pandas](https://github.com/pandas-dev/pandas): for inputting/outputting tabulated data.
  * [Python Imaging Library](https://pypi.python.org/pypi/PIL): for warping cover art images to square.
  * [unidecode](https://pypi.python.org/pypi/Unidecode): for translating accented characters to ASCII, so you don't have to get accents perfectly correct in your filename.
  <!-- * [mutagen](https://github.com/quodlibet/mutagen): for writing metadata; `conda install mutagen` (any anaconda distribution) or `pip install mutagen`. -->
  <!-- * [musicbrainzngs](https://github.com/alastair/python-musicbrainzngs): for interacting with MusicBrainz database; `pip install musicbrainzngs`. -->
  <!-- * [discogs_client](https://github.com/discogs/discogs_client): for interacting with Discogs database; `pip install discogs_client`. -->
  <!-- * [BeautifulSoup](https://pypi.python.org/pypi/beautifulsoup4): for parsing the "release group" HTML webpage and finding any linked discogs "master releases"; `pip install bs4`. Necessary because there is no way of retreiving this link from the MusicBrainz API directly. -->
  <!-- * [pandas](https://github.com/pandas-dev/pandas): for inputting/outputting tabulated data; `pip install pandas`. -->
  <!-- * [Python Imaging Library](https://pypi.python.org/pypi/PIL): for warping Album Art images; `pip install PIL`. -->
  <!-- * [unidecode](https://pypi.python.org/pypi/Unidecode): for translating accented characters to ASCII, so you don't have to get accents perfectly correct in your filename; `pip install unidecode`. -->

## Usage

    youtube <URL> <filename>

Filename can have un-escaped spaces. Escape any quotes/apostrophes. **Metadata will be inferred from 
filename with the `metadata` script if you follow the format `<artist> - <song>`**; everything left of space-dash-space is artist, everything to the right is the song name. Filename is not inferred from the youtube URL, because youtube video-naming is often inconsistent.

## Usage of metadata script
The metadata script is called automatically by `youtube` but you may want to use it or re-use it on existing files. Usage is as follows:

    metadata <flags> <filename(s)>

This time the filename(s) must have escaped spaces. The following flag options are available:

* `--url=<url>`: Add a URL to file metadata as a "Comment". The `youtube` script automatically passes this argument.
* `--confirm`: Prompt user to confirm release group/release to be used for artwork.
* `--genreonly`: Only add genre metadata, nothing else.
* `--forget`: Do not read previous user responses to ambiguous artist names from the config file.
* `--filter`: Extra filter on artist names and recording names. Probably excessive; may be removed.
* `--debug`: Echo some extra information during tagging process.

This tagging script is certainly not the fastest out there. For example the builtin PowerAmp Android app cover art-downloader is pretty darn fast.

Instead, it is designed to **never, ever tag music with the incorrect information**. This is my pet peeve. So it is slow, but it is very accurate.

<!-- You might ask: why do we run an artist search without also including the recording information, and make the user confirm? This is because I wasn't sure about the behavior of `search_recordings` run in `strict=True` mode when we don't know the artist ID. If artists with similar names (for example, a **tribute band**) share songs with the **same or similar title**, the search may return songs from artists we don't want. -->
<!-- , only have a guess at the approximate artist name (e.g. we say `Animals` or `Tom Petty` but want recordings under `The Animals` or `Tom Petty and the Heartbreakers`). -->
<!-- be some *rare, but very real* situations where the search  -->
<!-- However, if I can figure out a way to automatically filter these rare polluted matches, I may stop making the user confirm the artist ID with manual input. And they are indeed extremely rare. In future, may run search together, and then **only ask for user response if have more than one artist ID in the artist-credit list**. -->
<!-- And actually having trouble finding these purported false positives... maybe I'm crazy and they don't exist. But if a `Tom Petty` search returns `Tom Petty and the Heartbreakers` we should also have `Beatles` search returning `The Beatles Tribute`, with potentially identical recording names. -->

<!-- So, it may be better to have the user explicitly confirm the artist using disambiguation information. Though this needs more testing - if I can't find any examples, may just forget it. -->
<!-- In the future I might work this out, and eliminate the need to search for artists separately. Needs more testing. -->

<!-- You might ask why we do an artist search without also including the recording information? The answer is because of the limitations of the MusicBrainz API searching tools. If you want to name your file `Animals - Around and Around` then run a strict search of the datababase, you will get no results: the database thinks you want some obscure band called `Animals` and not the iconic British invasion band `The Animals`. Same goes for `Tom Petty` vs. `Tom Petty and the Heartbreakers` - most titles are under the latter, but if you want to name your files by the former, you will get no results. -->

<!-- At least this was my thinking before. Now that I've sat down and spelled it out, I think I'm wrong... shouldn't strict artist search include searches with "extra words?" So maybe I can search artists and recordings all at once. -->

## Overview of metadata script
The metadata script is called by default by the youtube-downloading `youtube` script, but it can also be called directly on any `.m4a`, `.aac`, and `.mp3` files in your library. Uses `Mutagen` to write tags; `mp3` metadata is added to the ID3 header, while `m4a` metadata is added in some mysterious way that Apple pioneered, but should still be readable by most media players.

Here's a play-by-play of what the metadata script does:
<!-- 1. Gets the MusicBrainz artist ID from the *filename-inferred artist name*. Search is strict, but a few exceptions. -->

1. Run strict search of MusicBrainz "recordings" matching the *filename-inferred* track name and with artist credits matching the *filename-inferred* artist name. Choose the first credit artist, and ask for user input if the first credit artists in the release list differ.
    * Allows for names starting with or without "the" (e.g. "Animals" vs. "The Animals" - you can choose to forego the "the", and this way the music in your filesystem will be sorted more naturally).
    * Allows for names ending with "&" or "and" something (e.g. "Tom Petty *and the Heartbreakers*").
    * Write "artist" metadata according to the search results.
2. Verify the recordings returned, eliminate some matches but permit others. In general, make sure "meaningful words" in the discovered recording names match the filename-inferred song name.
    * Ignores "Comfortably Numb (remix)" in search for "Comfortably Numb".
    * Ignores "Atom Heart Mother" or "Matilda Mother" but allows "Mother".
    * Allows "The Wall (part 1)" or "The Wall (part i)" for search for "The Wall".
    * Allows "Hush / I'm Alive" in search for "Hush" by choosing optional match on stuff either side of "/".
    * Write "title" metadata according to the search results.
3. Get release "groups" from the release list belonging to each recording, and consolidate the recordings (sorted by unique ID) into their corresponding release groups.
    * Sort the release groups first according to a clever ranking scheme. Every release group has an associated "category", so try to pick singles and albums over compilations or live performances. Also try to pick release groups with releases from earlier years (which are more likely to be "original" versions).
4. Get several *album-related* metadata categories from our ordered hierarchy of releases belonging to unique release groups.
    * Write "year" metadata from the earliest release amongst all members of *all* release groups.
    * Write "album name" metadata from the earliest release amongst all members of the highest-ranked release group.
    * Write "genres" from the first release group in hierarchy for which genres are available.
        * Retrieves the Discogs page of corresponding "master" by searching the MusicBrainz `html` webpage for a linked Discogs "master" ID (not currently implemented in the python API).
        * Also write "genres" from the MusicBrainz "tags" associated with individual recordings.
        * Permitted names are the Discogs "styles" found in the `genrelist` file. Does not allow overly generic names like "rock" and "pop".
    * Write "cover art" metadata from the most *modern* release and most *modern* release format amongst releases in the highest-ranked release group. This gets the nicest-looking cover art available. If `--confirm` was passed, user can choose to bypass release groups/releases for artwork.

## Suggestions for Playback Software
### On macOS
Use **Swinsian** instead of iTunes for playback. Swinsian costs \$20, but IMO it is really worth it. The two iTunes dealbreakers for me were:

  1. iTunes can't "watch" folders while open (i.e. automatically add and remove songs as they appear/disappear from a folder). The only hope is some complicated hack involving the "Automatically add to iTunes" folder (personally I couldn't get this to work; perhaps one must restart iTunes every time or tell it manually to rescan this folder).
  2. Browsing between files is difficult (cannot type out the name of an artist/song/filename and have
  the cursor jump to that row [and when trying to do that, get inexplicable slowdown]; also there is no filename column).

### On Windows
Use **foobar2000**. It looks pretty ugly out of the box, but it is free, it is lighting fast, and you can use custom themes that make it look pretty slick. It's packed with useful features, too. Unfortunately this app is not available for macOS.

