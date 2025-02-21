---
title: FAQ
description: 
published: true
date: 2021-05-25T20:01:09.320Z
tags: 
editor: markdown
dateCreated: 2021-05-25T20:01:09.320Z
---

## How does Readarr work? 
- Readarr relies on RSS feeds to automate grabbing of releases as they are posted, for both new releases as well as previously released releases being released or re-released. The RSS feed is the latest releases from a site, typically between 50 and 100 releases, though some sites provide more and some less. The RSS feed is comprised of all releases recently available, including releases for requested media you do not follow, if you look at debug logs you will see these releases being processed, which is completely normal.
- Readarr enforces a minimum of 10 minutes on the RSS Sync interval and a maximum of 2 hours. 15 minutes is the minimum recommended by most indexers, though some do allow lower intervals and 2 hours ensures Readarr is checking frequently enough to not miss a release (even though it can page through the RSS feed on many indexers to help with that). Some indexers allow clients to perform an RSS sync more frequently than 10 minutes, in those scenarios we recommend using Readarr's Release-Push API endpoint along with an IRC announce channel to push releases to Readarr for processing which can happen in near real time and with less overhead on the indexer and Readarr as Readarr doesn’t need to request the RSS feed too frequently and process the same releases over and over.

## How does Readarr find books? 
- Readarr does ''not'' regularly search for book files that are missing or have not met their quality goals. Instead, it fairly frequently queries your indexers and trackers for ''all'' the newly posted books, then compares that with its list of books that are missing or need to be upgraded. Any matches are downloaded. This lets Readarr cover a library of ''any size'' with just 24-100 queries per day (RSS interval of 15-60 minutes). If you understand this, you'll realize that it only covers the ''future'' though.
- So how do you deal with the present and past? When you're adding a book, you'll need to set the correct path, profile and monitoring status then use the Start search for missing book checkbox. If the book hasn't been released yet, you don't need to initiate a search.
- Put another way, Readarr will only find books that are newly uploaded to your indexers. It will not actively try to find books you want that were uploaded in the past.
- If you've already added the book, but now you want to search for it, you have a few choices. You can go to the book's page and use the search button, which will do a search and then automatically pick one. You can use the Search tab and see ''all'' the results, hand picking the one you want. Or you can use the filters of <code>Missing</code>, <code>Wanted</code>, or <code>Cut-off Unmet</code>.
- If Readarr has been offline for an extended period of time, Readarr will attempt to page back to find the last release it processed in an attempt to avoid missing a release. As long as your indexer supports paging and it hasn't been too long Readarr will be able to process the releases it would have missed and avoid you needing to perform a search for the missed books.

## How are possible downloads compared? 
`Generally Quality Trumps All`

The current logic [can be found here](https://github.com/Readarr/Readarr/blob/develop/src/NzbDrone.Core/DecisionEngine/DownloadDecisionComparer.cs).

As of 1/19/2021 the logic is as follows:
- Quality
- Custom Format Score
- Protocol
- Indexer Priority
- Indexer Flags
- Seeds/Peers (If Torrent)
- Age (If Usenet)
- Size

## What are Lists and what can they do for me? 
- Lists are a part of Readarr that allow you to follow a given list creator.
- Let's say that you follow a given list creator on Trakt/TMDb and really like their Marvel Cinematic Universe film section and want to watch every book on their list. You look in your Readarr and realize that you don't have those books. Well instead of searching one by one and adding those lists and then searching your indexers for those books. You can do this all at once with a List. The Lists can be set to import all the books on that curators list as well as be set to automatically assign a quality profile, automatically add, and automatically monitor that book.
- `CAUTION:` If lists are done improperly they will absolutely wreck your library with a bunch of trash you have no intention of watching. So make sure of what you're importing before you click save. 
- ie. physically look at the list before you even go to Readarr.

## Why can't I add a new book or author to Readarr? 
- Readarr uses [http://goodreads.com GoodReads] for book and author information and images like cover and author art, banners and backgrounds. Generally, there are a few reasons why you may not be able to add a book:
- GoodReads doesn't like special characters to be used when searching for books through the API (which Readarr uses), so try searching a translated name, and/or without special characters.
- You can also add books and authors by GoodReads ID, ISBN ID, or ASIN ID.
- The book has an issue with GoodReads API data. Unfortunately this is something we probably can't solve for you.
- The book falls below the settings you've chosen (votes, pages, etc.) to appear in Readarr.

## How can I rename my author folders? 

- Library 
- Mass Editor
- Select what authors need their folder renamed
- Change Root Folder to the same Root Folder that the authors currently exist in
- Select "Yes move files"

## How can I mass delete authors from the wanted list? 
- Use Mass Editor -> Select authors you want to delete -> Delete

## Why doesn't Readarr work behind a reverse proxy 
- Readarr uses .NET Core and a new webserver. In order for SignalR to work, the UI buttons to work, database changes to take, and other items. It requires the following addition to the location block for Readarr:
 proxy_http_version 1.1;
 proxy_set_header Upgrade $http_upgrade;
 proxy_set_header Connection $http_connection;

- Make sure you `do not` include <code>proxy_set_header Connection &quot;Upgrade&quot;;</code> as suggested by the nginx documentation. `THIS WILL NOT WORK`
- [See this ASP.NET Core issue](https://github.com/aspnet/AspNetCore/issues/17081)
- If you are using a CDN like Cloudflare ensure websockets are enabled to allow websocket connections.

## How do I update Readarr? 
- Go to Settings and then the General tab and show advanced settings (use the toggle by the save button).
- Under the Development section change the branch name to <code>master</code> or <code>develop</code>
- Save
- This will not install the bits from that branch immediately, it will happen during the next update.


- Note: If your install is through Docker append <code>:release</code>,  <code>:latest</code>, or  <code>:testing</code> to the end of your container tag depending on who makes your builds.


## Installing a newer version

If Native:
- Go to System and then the Updates tab
- Newer versions that are not yet installed will have an update button next to them, clicking that button will install the update.

If Docker:
- Repull your tag and update your container

## Can I switch between branches? 
- You can (almost) always increase your risk. 
- <code>master</code> can go to <code>develop</code> or <code>nightly</code>
- <code>develop</code> can go to <code>nightly</code>
- Check with the development team to see if you can switch from <code>nightly</code> to <code>master</code>; <code>nightly</code> to <code>develop</code>; or <code>develop</code> to <code>master</code> for your given build.
- Failure to follow these instructions may result in your Readarr becoming unusable or throwing errors. You have been warned.
- The most common error is something like <code>Error parsing column 45 (Language=31 - Int64)</code> or other similar database errors around missing columns or tables.

## How does readarr handle foreign books or foreign titles? 
- Readarr uses both Alt Titles and Translations for parsing and searching. Search will use the Original Title, English Title, and Translated Title from whatever languages you have preferred (in profile and CFs). Parsing should look for a match in all Translations and Alt Titles.

- To get a book in a foreign language set your Profile Language to Original (Movie's Original Language), a specific language for that profile, or any and use custom formats to determine which language to grab.
- Note that this does not include any indexer languages specified as multi.

## Help, Movie Added, But Not Searched 

- Neither Readarr *actively* search for missing books automatically. Instead, a periodic query of new posts is made to all indexers configured for RSS. When a wanted or cutoff unmet book shows up in that list, it gets downloaded. This means that until a book is posted (or reposted), it won’t get downloaded.
- If you’re adding an author with books that you want now, the best option is to check the “Start search for missing books” box, to the left of the *Add Author* button. You can also go to the page for an author you’ve added and click the magnifying glass *Search* button next to the book you want, or use the Wanted view to search for Missing or Cutoff Unmet books.

## Root path for authors imported from lists becomes “C:” or other weird paths 
- Sometimes you can get a problem that authors that are imported from your lists, gets imported with the root path set to “C:” or other weird paths.

- This is a known issue for when the root path is either not setup during the creation of the list, or if the root path has been deleted after the list was created. Note that this problem can still occur even if the list is `edited` and the correct root path is set.

- Use the Mass Editor to fix paths of existing authors.

## Book Imported, But Source File And Torrent Not Deleted 
- Check if you have Completed Download Handling - Remove turned on. (This does not work if you are using rtorrent.)

- If you are using deluge make sure auto-managed is turned on. And that torrents get paused when they reach specified seeding quota.


## I am using a Pi and Raspbian and Readarr will not launch ## 

- Possible Solution:

Managed to fix the issue by installing the backport from debian repo. Generally not recommended to use backport in blanket upgrade mode. Installation of a single package may be ok but may also cause issues. So got to understand what you are doing.

Steps to fix:

First ensure you are running Raspbian buster e.g using <code>lsb_release -a</code>

Distributor ID: Raspbian

Description: Raspbian GNU/Linux 10 (buster)

Release: 10

Codename: buster

- If you are using buster:

 <code>Add the line `deb <nowiki>http://deb.debian.org/debian</nowiki> buster-backports` main to `/etc/apt/sources.list`
 Run apt-get update
 apt-get -t buster-backports install libseccomp2</code>

## Why are lists sync times so long and can I change it? 

Lists never were nor are intended to be <code>add it now</code> they are <code>hey I want this, add it eventually</code> tools.

You can trigger a list refresh manually, script it and trigger it via the API, or add the books directly to readarr.

This change was due to not have our server get killed by people updating lists every 10 minutes.

## Can I disable the refresh books task 

No, nor should you through any SQL hackery.  The refresh books task queries the upstream Servarr proxy and checks to see if the metadata for each book (ids, cast, summary, rating, translations, alt titles, etc.) has updated compared to what is currently in Readarr. If necessary, it will then update the applicable books.

A common complaint is the Refresh task causes heavy I/O usage.  One setting that can cause issues is "Rescan Author Folder after Refresh".  If your disk I/O usage spikes during a Refresh then you may want to change the Rescan setting to <code>Manual</code>.  Do not change this to <code>Never</code> unless all changes to your library (new books, upgrades, deletions etc) are done through Readarr.  If you delete book files manually or a third party program, do not set this to <code>Never</code>.


