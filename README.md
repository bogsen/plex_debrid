# plex_debrid
Plex torrent streaming through Debrid Services, using the new Plex Discover feature.

**Using the new Plex Discover Feature, your Plex Home users can add movies/shows to their watchlist and they become available to stream in seconds.**

## In Action:

![Alt Text](https://github.com/itsToggle/plex_debrid/blob/main/cool.gif)

## UI:
Main Menu             |  Example Run
:-------------------------:|:-------------------------:
![Alt Text](https://i.ibb.co/2sJpQxp/Screenshot-2022-05-19-145135.png)  | ![Alt Text](https://i.ibb.co/Fg2cwkh/Screenshot-2022-05-19-144919.png)
Settings            |  Ingored Media
![Alt Text](https://i.ibb.co/X49r7Pn/Screenshot-2022-05-19-145706.png)  | ![Alt Text](https://i.ibb.co/TLRysHD/Screenshot-2022-05-19-145915.png)



## Description:

*For this download automation to work, you need to mount your debrid service as a virtual drive. After creating a plex library of this virtual drive, you can stream torrents that are cached on your debrid service without the need to download them first.* 

The plex watchlist of specified users is constantly checked for newly added movies/shows and newly released episodes of watchlisted shows.
Once new content is found, torrent indexers are scraped for the best, cached release on selected debrid services. The torrent is then added to a suitable debrid service and a library refresh is performed to make the newly added content available. 

- For a movie or a one-season tv shows this takes about 10-20 seconds.
- Thanks to multithreading, multi-season tv shows will also download entirely in about 10-20 seconds.

This is a pre-alpha release. shits not ready! Feel free to check it out though, I will continously improve the speed, reliability and user-friendlyness.

## Setup:

*Debrid services like premiumize can be mounted with the official rclone software. Other debrid services, like Debrid-Link or All-Debrid can be mounted through their WebDav implementation.*

**The only service currently available for this download automation is Real-Debrid. I have written an rclone fork, which allows you to mount the RealDebrid /torrent directory as a virtual drive.**

**Pre-Setup:**

1. Install my rclone fork, follow the instructions: https://github.com/itsToggle/rclone_RD
2. Create a plex 'movie' library of the mounted virtual drive or add the mounted virtual drive to an existing 'movie' library.
3. Create a plex 'shows' library of the mounted virtual drive or add the mounted virtual drive to an existing 'shows' library.
4. *Recommendation: disable 'video preview thumbnails', disable 'intro detection', disable the scheduled task 'perfom extensive media analysis' to reduce the download traffic
5. You and your home users can now stream cached torrents from RealDebrid!

**Setup:**
1. Run the script! (google "how to run a python script" ;) )
2. The script will guide you through the initial setup.
3. You're done!
5. Choose option '1' to run the download automation. Choose option '2' to explore or edit the Settings or open the "settings.json" file the script creates after the first run.

## Usage and Tips:

- The Plex Watchlist of your specified users will polled for changes every 10 seconds, which is when it will try to find newly added content. 
- The Plex Watchlist will be updated entirely every 30 minutes, which is when it will try to find newly released episodes from watchlisted series. This is only done every 30 minutes, because building the whole watchlist can take more than a minute, depending on the amount of shows you have in there.
- Your *entire* Plex Library (including shares) is checked for any existing seasons/episodes of a watchlisted show and will avoid downloading those.
- If you dont want to download a specific episode or season of a show, navigate to that show in the discovery feature and mark the episodes/seasons that you want to ignore as 'watched'. The watch status inside the discovery feature is not connected to the watch status inside your libraries.
- When some content could repeatedly not be downloaded, it will be marked as 'watched' in the Discovery feature of the first specified user. This will cause the scraper to ignore the content, until its marked as 'unwatched' again.
- You can explore and remove ignored content in the main menu.
- You can connect the script to trakt.tv to get more accurate release dates and times for your content. You can also synchronize your trakt watchlist to your plex watchlist.
- This script will automatically pick the best release that could be found. To change how the script picks the best release, check out the next section.
- If you don't want the main menu to show when you start the script and run the download automation right away, you can define this in the 'UI Settings' section of the 'Settings' menu.
- You can return to the main menu at any time.

## Sorting the scraped releases:

The scrapers usually provide a whole bunch of releases. Adding them all to your debrid services would clutter your library and slow things down. This is why this script automatically sorts the releases by completely customizable categories and picks the best one.

The sorting is done by providing an unlimited number of sorting 'rules'. Rules can be added, edited, delted or moved. The first rule has the highest priority, the last one the lowest.

Each rule consist of:
- a regex match group that defines what we are looking for. Check out regexr.com to try out some regex match definitions.
- an attribute definition that defines which attribute of the release we want to look in (can be the title, the source, or the size)
- an interpretation method. This can be either:
  - "number" : the first regex match group will be interpreted as an integer and releases will be ranked by this number.
  - "text" : we will give each release a rank accoring to which match group its in.
- lastly we define wheter to rank the releases in ascending or descending order.

You can now test out your current sorting rules in the scraper settings.

Lets make some rules: 
### Example resolution sorting:
We want to download releases up to our prefered resolution of 1080p.
For this, we will choose the following setup:
- regex definition: "(1080|720|480)(?=p)" - This is one match group, that matches either "1080", "720" or "480", followed by the letter "p". This is a typical Resolution definition of releases.
- attribute definition: "title" - we want to look for this inside the release title
- interpretation method: "number" - we want to sort the releases by the highest number to the lowest number
- ascending/descending: "1" - 1 means descending. We want to sort the releases in decending order to get the highest resolution release.

### Example codec sorting:
We want to download releases that use the x265 Codec, rather then the x264 codec. 
For this, we will choose the following setup:
- regex definition: "(h.?265|x.?265)|(h.?264|x.?264)" - These are two match groups, that match typical codec descriptions in the release titles
- attribute definition: "title" - we want to look for this inside the release title
- interpretation method: "text" - by choosing this, we define that the releases should be sorted by the match group they are in.
- ascending/descending: "1" - 1 means descending. Descending in this context means, that the First matchgroup is preffered over the second matchgroup, and both are prefered over a release that doesnt match.

### Example release exclusion:
We don't want to download releases that are HDR or 3D. For this rule to ne effective, we need to make it our first rule.
For this, we will choose the following setup:
- regex definition: "(\\.HDR\\.|\\.3D\\.)"
- attribute definition: "title" - we want to look for this inside the release title
- interpretation method: "text" - by choosing this, we define that the releases should be sorted by the match group they are in.
- ascending/descending: "0" - 0 means ascending. Ascending in this context means, that releases that don't match are prefered over releases that do.

### Example size sorting:
We want to sort or releases by size - this should be implemented as one of the last rules.
For this, we will choose the following setup:
- regex definition: "(.*)" - This is one match group that simply matches everything.
- attribute definition: "size" - we want to look for this inside the release size
- interpretation method: "number" - by choosing number, we define that the release size should be interpreted as a number.
- ascending/descending: "0" - 0 means ascending. We want to select the smallest release.


## Limitations:
- The plex discover API only provides a release date, not a release time for new episodes. This makes it hard to determined when to start looking for releases and when to ignore an episode. This script will only download episodes when its been a day since the air-date or if plex shows the episode as availabe on streaming services. It is recommended to connect this script to trakt.tv, to allow for more accurate release dates and times.
- Since this is more of a proof of concept at the moment, There are only two scrapers implemented - rarbg and 1337x.
