## Be Careful!
Downloading content from the internet for personal use (not distribution)
is not illegal (criminal law or copyright infringement); but by using this
script, you are breaking Youtube's Terms of Service (civil law). Then again, that's a problem for the `youtube-dl` devs, not us :)

## Overview
This script downloads and saves audio from a youtube url into an aac/m4a file (the native
format of youtube audio), applies metadata based on the user-input name, and normalizes the audio/volume.
  * If you just want to download the m4a files and do nothing else, comment out
      the parts below that adjust the volume/add metadata.
  * To change the download location, edit the `directory` variable.
  * If youtube-dl **stops working**, it is often because youtube.com has changed how they store
      video/audio. The youtube-dl developers are very active and usually will release an updated
      version within a couple days; just call `youtube-dl -U` ("update") and it should start working again.

## Requirements
  * youtube-dl (`brew install youtube-dl` with Homebrew on Mac;
      see github: https://github.com/rg3/youtube-dl)
  * ffmpeg (`conda install ffmpeg` [any anaconda distribution],
      or `brew install ffmpeg` with Homebrew on Mac)
  * ffmpeg-normalize (python package for normalizing volume, requires ffmpeg accessible from shell;
      `pip install ffmpeg-normalize`; see github: https://github.com/slhck/ffmpeg-normalize)
  * mutagen (python package for writine metadata; `conda install mutagen`, `pip install mutagen`, etc;
      see github: https://github.com/quodlibet/mutagen)

## Usage

    youtube <URL> <filename>

Filename can have un-escaped spaces. Escape any quotes/apostraphes. Metadata will be INFERRED from 
filename if you follow the format `<Artist> - <Title>`; everything left of space-dash-space
is artist, everything to the right is title. Makes life easier when using media player.

## Overview of metadata script
1. uses extra-strict search to narrow down artist and songnames (strict search still allows extra words; 
so we make sure words are all identical). 
2. gets the artist ID (and asks for user input if artist-search has ambiguous (e.g. Animals vs. The Animals) and
record user responses in a file
3. runs strict search with artist ID of recordings, and filters recordings based on names
    * ignores Comfortably Numb (remix) in search for Comfortably Numb
    * allows The Wall (part 1) or The Wall (part i) for search for The Wall
    * ignores Atom Heart Mother or Matilda Mother but allows Mother
4. gets release-groups from the release-list of each recording
5. groups the releases into release-groups (can be overlap)
6. writes release-name title from shortest titles amongst all releases in *chosen release group*
    * writes The Wall instead of The Wall (anniversary edition)
7. writes release-year from earliest year amongst all releases in *all release groups*
8. writes cover-art from most *modern* release and most *modern* release format amongst releases in release group
and gets genre from Discogs page of corresponding release-group
    * optionally prompt user to choose release group
    * optionally keep iterating through release-groups if no Discogs genre found

## Suggestions
Use Swinsian instead of iTunes for playback. The two dealbreakers are:
1) iTunes can't "watch" folders, and the only hope is some complicated/non-trivial hack involving with 
the "Automatically add to iTunes" folder (which couldn't get to work; maybe have to close/re-open) 
2) browsing between files is difficult (there is no filename table, and
typing out the name of something to jump to that point in the list is, inexplicably, extremely slow)

