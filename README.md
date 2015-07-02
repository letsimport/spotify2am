# Spotify to Apple Music
Some simple Python 3 scripts to help you into importing your existing Spotify library into Apple Music.  
**Work in progress !**

[CHANGES FROM ORIGINAL]
1. Searches for artist in addtional to title in iTunes. Corrects issue where more than 25 results exist for title and artist desired isn't in top 25
2. instead of exact match of artist and title used SequenceMatcher to match with 90% ratio. Higher risk for incorrect match, but eliminates a lot of skips from small spacing, formatting, spelling issues.
3. Updated to allow for Rdio csv (changed row indexes in line 38)
4. Match success for me: original script 518/783. This version 661/783. YMMV.

## Usage

### 1(a). Export the Spotify songs to an CSV File
The first step is getting the songs you want to import into Apple Music into a CSV file. The simplest way to do this is using [Exportify](https://rawgit.com/watsonbox/exportify/master/exportify.html).  
If you want to export you whole Spotify library, simply create a new playlist called *All* and drag your whole library into it using the Spotify desktop app. You can then export the playlist *All* using *Exportify*. Save the resulting file as *export.csv* in the same directory as the directory you cloned this repo into. Open csv file and delete column A.

### 1(a). Export the Spotify songs to an CSV File
Export Rdio playlist or library with RdioEnhancer (https://chrome.google.com/webstore/detail/rdio-enhancer/hmaalfaappddkggilhahaebfhdmmmngf?hl=en) 
Save the resulting file as *export.csv* in the same directory as the directory you cloned this repo into.
I found that it did some weird unicode formatting on some songs. To fix, I used "Import" in excel and selected Unicode 16.0 UTF-8 as input format then re-saved that as export.csv.

### 2. Match the Spotify songs with their Apple Music identifier
In order to add songs to our Apple Music library, we need their Apple Music identifier. Running `python3 retrieve-identifiers.py` will use the *export.csv* file to create a new file *itunes.csv* with each line consisting of the Apple Music identifier of a song in your Spotify/Rdio library.

### 3. Use an intercepting proxy to retrieve the Apple Music request headers
Start iTunes and [Charles](http://www.charlesproxy.com) (or another intercepting proxy you like). Make sure SSL Proxying is enabled and working correctly. Next, select a random song on Apple Music you don't have in your library yet, right click and choose 'Add to library'. If everything went well, you're now able to view all the request headers in Charles of a request to `https://ld-4.itunes.apple.com/WebObjects/MZDaap.woa/daap/databases/1/cloud-add`. We're only interested in `Cookie`, `X-Dsid` and `X-Guid`. Copy the value of these header and paste them in the appropriate place  in `insert-songs.py` (line *29* and further).  
Next, run `python3 insert-songs.py` and go grab a coffee. You're songs are now being imported into Apple Music.


## Current issues

### API limit rate
Apple Music doesn't like it when we're adding ~50 songs in a few minutes. If we do so, the API responds to all further request with `403 Too many requests`. After this, you're blocked from the API for an undetermined amount of time. We're currently trying to avoid this by sleeping 5 seconds after each request. As I'm currently blocked myself (pro-tip: 1 second delay is too low), I don't know if this long enough to prevent the API from blocking us

### Missing songs
The script I'm using to retrieve the Apple Music identifier for a Spotify song is quite basic. It simply compares the title and artist to find out if a Spotify and Apple Music song match. Some songs don't have the exact same title (extraneous spacing for example) in both services. This results in the script failing to retrieve an identifier for some songs.
