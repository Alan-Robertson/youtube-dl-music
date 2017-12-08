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
  * [youtube-dl](https://github.com/rg3/youtube-dl): script for downloading youtube media; `brew install youtube-dl` (Homebrew on Mac).
  * [ffmpeg](https://github.com/FFmpeg/FFmpeg): batteries-included package for creating/modifying media files; `conda install ffmpeg` (any anaconda distribution) or `pip install ffmpeg` or `brew install ffmpeg` (Homebrew on Mac).
  * [ffmpeg-normalize](https://github.com/slhck/ffmpeg-normalize): python package for normalizing volume, requires ffmpeg accessible from shell; `pip install ffmpeg-normalize`.
  * [mutagen](https://github.com/quodlibet/mutagen): python package for writing metadata; `conda install mutagen` (any anaconda distribution) or `pip install mutagen`.

## Usage

    youtube <URL> <filename>

Filename can have un-escaped spaces. Escape any quotes/apostrophes. Metadata will be INFERRED from 
filename with the `metadata` script if you follow the format `<artist> - <song>`; everything left of space-dash-space is artist, everything to the right is the song name. Filename is not inferred from the youtube URL, because youtube video-naming is often inconsistent.

## Overview of metadata script
There are two major online discography databases: Discogs and MusicBrainz. Both have their strengths and weaknesses, and both have python APIs. So, why don't we use both? :)
  * Add lines that look like `key = value` to the "config" file to enable metadata tagging. Whitespace doesn't matter, and single/double quotes are allowed but not necessary.
  * Create a Discogs account and create a token for the API; add that to the `config` file with `token=<your token here>`.
  * Create a MusicBrainz account and you can simply use your account username and password; add them to the `config` file with `username=<your username here>` and `password=<your password here>`.

The metadata script is called by default by the youtube-downloading `youtube` script, but it can also be called directly on any `.m4a`, `.aac`, and `.mp3` files in your library. Uses `Mutagen` to write tags.

Here's a play-by-play of what the metadata script does:
1. Gets the MusicBrainz artist ID from the *filename-inferred artist name*.
    * Asks for user input if artist-search has ambiguous (e.g. "Animals" vs. "The Animals").
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

## Suggestions
Use Swinsian instead of iTunes for playback. The two dealbreakers are:
  1. iTunes can't "watch" folders, and the only hope is some complicated/non-trivial hack involving with 
  the "Automatically add to iTunes" folder (which couldn't get to work; maybe have to close/re-open).
  2. Browsing between files is difficult (cannot type out the name of an artist/song/filename and have
  the cursor jump to that row [and when trying to do that, get inexplicable slowdown]; also there is no filename column).
Swinsian costs \$20, but IMO it is really worth it.

