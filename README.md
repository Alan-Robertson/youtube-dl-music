## Be Careful!
Downloading content from the internet for personal use (not distribution)
is not illegal (criminal law or copyright infringement); but by using this
script, you are breaking Youtube's Terms of Service (civil law). Then again, that's a problem for the `youtube-dl` devs, not us :)

## Overview
This script downloads and saves audio from a youtube url into an aac/m4a file (the native
format of youtube audio), applies metadata based on the user-input name, and normalizes the audio/volume.
  * To change the download location, edit the `directory` variable.
  * If you just want to download the m4a files and do nothing else, comment out
      parts of the `youtube` script that adjust volume/add metadata.
  * If youtube-dl **stops working**, it is often because youtube.com has changed how they store
      video/audio. The youtube-dl developers are very active and usually will release an updated
      version within a couple days; just call `youtube-dl -U` ("update") and it should start working again.

## Dependencies
  * access to a UNIX shell.
  * python 3.6+: `metadata` script has f-strings, which is a python 3.6+ feature (f-strings are totally awesome and you should be using them (: ).
  * [youtube-dl](https://github.com/rg3/youtube-dl): script for downloading youtube media; `brew install youtube-dl` (Homebrew on Mac).
  * [ffmpeg](https://github.com/FFmpeg/FFmpeg): batteries-included package for creating/modifying media files; `conda install ffmpeg` (any anaconda distribution) or `pip install ffmpeg` or `brew install ffmpeg` (Homebrew on Mac).
  * [ffmpeg-normalize](https://github.com/slhck/ffmpeg-normalize): python package for normalizing volume, requires ffmpeg accessible from shell; `pip install ffmpeg-normalize`.
  * [mutagen](https://github.com/quodlibet/mutagen): python package for writing metadata; `conda install mutagen` (any anaconda distribution) or `pip install mutagen`.

## Usage

    youtube <URL> <filename>

Filename can have un-escaped spaces. Escape any quotes/apostrophes. Metadata will be INFERRED from 
filename with the `metadata` script if you follow the format `<artist> - <song>`; everything left of space-dash-space is artist, everything to the right is the song name. Filename is not inferred from the youtube URL, because youtube video-naming is often inconsistent.

## Philosophy of metadata script
This tagging script is certainly not the fastest out there. For example the builtin PowerAmp Android app cover art-downloader is pretty darn fast.

Instead, it is designed to **never, ever tag music with the incorrect information**. This is my pet peeve. So it is slow, but it is very accurate.

You might ask: why do we run an artist search without also including the recording information, and make the user confirm? This is because I wasn't sure about the behavior of `search_recordings` run in `strict=True` mode when we don't know the artist ID. If artists with similar names (for example, a **tribute band**) share songs with the **same or similar title**, the search may return songs from artists we don't want.
<!-- , only have a guess at the approximate artist name (e.g. we say `Animals` or `Tom Petty` but want recordings under `The Animals` or `Tom Petty and the Heartbreakers`). -->
<!-- be some *rare, but very real* situations where the search  -->

However, if I can figure out a way to automatically figure out these rare polluted matches, I may stop making the user confirm the artist ID with manual input. And actually having trouble finding these purported false positives... maybe I'm crazy and they don't exist. But if a `Tom Petty` search returns `Tom Petty and the Heartbreakers` we should also have `Beatles` search returning `The Beatles Tribute`, with potentially identical recording names.

<!-- So, it may be better to have the user explicitly confirm the artist using disambiguation information. Though this needs more testing - if I can't find any examples, may just forget it. -->
<!-- In the future I might work this out, and eliminate the need to search for artists separately. Needs more testing. -->

<!-- You might ask why we do an artist search without also including the recording information? The answer is because of the limitations of the MusicBrainz API searching tools. If you want to name your file `Animals - Around and Around` then run a strict search of the datababase, you will get no results: the database thinks you want some obscure band called `Animals` and not the iconic British invasion band `The Animals`. Same goes for `Tom Petty` vs. `Tom Petty and the Heartbreakers` - most titles are under the latter, but if you want to name your files by the former, you will get no results. -->

<!-- At least this was my thinking before. Now that I've sat down and spelled it out, I think I'm wrong... shouldn't strict artist search include searches with "extra words?" So maybe I can search artists and recordings all at once. -->

## Overview of metadata script
There are two major online discography databases: Discogs and MusicBrainz. Both have their strengths and weaknesses, and both have python APIs. So, why don't we use both? :)
  * Add lines that look like `key = value` to the "config" file to enable metadata tagging. Whitespace doesn't matter, and single/double quotes are allowed but not necessary.
  * Create a Discogs account and create a token for the API; add that to the `config` file with `token=<your token here>`.
  * Create a MusicBrainz account and you can simply use your account username and password; add them to the `config` file with `username=<your username here>` and `password=<your password here>`.

The metadata script is called by default by the youtube-downloading `youtube` script, but it can also be called directly on any `.m4a`, `.aac`, and `.mp3` files in your library. Uses `Mutagen` to write tags.

Here's a play-by-play of what the metadata script does:
1. Gets the MusicBrainz artist ID from the *filename-inferred artist name*. Search is strict, but a few exceptions.
    * Allows for names starting with or without "the" (e.g. "Animals" vs. "The Animals" -- you can choose to forego the "the", and this way the music in your filesystem will be sorted more naturally).
    * Allows for names ending with "&" or "and" something (e.g. "Tom Petty *and the Heartbreakers*").
    * Asks for user input if artist-search returns more than one option. Includes available *disambiguation* metadata in parentheses.
    * Records user responses in a hidden file stored in the download location; so user only has to specify once.
2. Searches MusicBrainz "recordings" under the artist ID by the *filename-inferred song name*. Makes sure "meaningful words" in the discovered recording names match the filename-inferred song name. Some examples:
    * Ignores "Comfortably Numb (remix)" in search for "Comfortably Numb".
    * Ignores "Atom Heart Mother" or "Matilda Mother" but allows "Mother".
    * Allows "The Wall (part 1)" or "The Wall (part i)" for search for "The Wall".
3. Gets release-groups from the release-list of each recording.
4. Groups the recordings into their corresponding release-groups (we might have in our list the same recording IDs from different releases/release groups).
5. Writes release-name title from shortest titles amongst all releases in the *chosen release group*.
    * Writes "The Wall" instead of "The Wall (anniversary edition)".
6. Writes release-year from earliest year amongst all releases in *all release groups*.
    * This gives the "actual" year of publication, not some random re-release or anniversary edition.
7. Writes cover-art from most *modern* release and most *modern* release format amongst releases in release group.
    * By choosing the most modern versions, we can get the nicest-looking cover art available.
    * Optionally, the script prompts user to choose release group.
8. Gets genre from Discogs page of corresponding release-group, and from MusicBrainz.
    * Discogs has specific genre-fields corresponding to release-groups or "master"s. MusicBrainz only has "tags"
    associated with individual recordings, and there seems to be zero standardization there.
    * The metadata script attempts its own standardization of genre names, to avoid duplicates. Also very generic genres like "rock", "pop", "rap", or "country" are ignored.

## Suggestions for Playback Software
### On macOS
Use Swinsian instead of iTunes for playback. The two dealbreakers for me were:
  1. iTunes can't "watch" folders while open (i.e. automatically add and remove songs as they appear/disappear from a folder). The only hope is some complicated hack involving 
  the "Automatically add to iTunes" folder (personally I couldn't get this to work; perhaps one must restart iTunes every time or tell it manually to rescan this folder).
  2. Browsing between files is difficult (cannot type out the name of an artist/song/filename and have
  the cursor jump to that row [and when trying to do that, get inexplicable slowdown]; also there is no filename column).
Swinsian costs \$20, but IMO it is really worth it.
### On Windows
Use `foobar2000`. It looks pretty ugly out of the box, but it is free, it is lighting fast, and you can use custom themes that make it look pretty slick. It's packed with useful features, too. Unfortunately this app is not available for macOS.

