# Be Careful!
Downloading content from the internet for personal use (not distribution)
is not illegal (criminal law or copyright infringement); but by using this
script, you are breaking Youtube's Terms of Service (civil law).

# Summary
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

# Suggestions
Use Swinsian instead of iTunes for playback. The two dealbreakers are:
1) iTunes can't "watch" folders, and the only hope is some complicated/non-trivial hack involving with 
the "Automatically add to iTunes" folder (which couldn't get to work; maybe have to close/re-open) 
2) browsing between files is difficult (there is no filename table, and
typing out the name of something to jump to that point in the list is, inexplicably, extremely slow)
