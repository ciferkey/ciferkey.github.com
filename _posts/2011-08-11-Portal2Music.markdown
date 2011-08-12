---
layout: post
title: Extracting Music From Portal 2
categories:
  - music
  - portal
---

I'm a pretty big fan of Portal and Portal 2.  As such I was sucked into the whole pre-release ARG. I figure that if I spent all that money on the Potato sack, I might as well get some free music along with the game. Luckily there is an immense amount of source files in the game.  Most of them are ambient noise and sound effects but you can get the co-op ending song [Robots FTW](http://www.last.fm/music/Valve/_/Robots+FTW), and the the single player ending song [Want You Gone](http://www.last.fm/music/Jonathan+Coulton/_/Want+You+Gone) by Jonathan Coulton, and the [turret opera](http://www.last.fm/music/Ellen+Mclain/_/Turret+Opera) (Cara bella, cara mia bella!) by Ellen Mclain.

On a OS X use [Source Fangler](http://www.markdouma.com/sourcefinagler/), open

    ~/Library/Application\ Support/Steam/SteamApps/common/portal\ 2/portal2/pak01_dir.vpk

On a Windows use [GCFScrape](http://nemesis.thewavelength.net/index.php?p=26), open

    C:\Program Files (x86)\Steam\steamapps\common\portal 2\portal2\pak01_dir.vpk

For Robots FTW go to sound -> music -> portal2_robots_ftw.wav 

For Want You Gone go to sound -> music -> portal2_robots_ftw.wav 

They're both lossless and you can play them as is or convert it to flac. Getting the turret opera is a bit more involved. Its in a bink video file.

    C:\Program Files (x86)\Steam\steamapps\common\portal 2\portal2\media\sp_30_a4_finale5.bik
    ~/Library/Application\ Support/Steam/SteamApps/common/portal\ 2/portal2/media/sp_30_a4_finale5.bik

On Windows and OS X respectively. On Windows you can convert it with [RAD Video Tools](http://www.radgametools.com/bnkdown.htm). You can play them on OS X. Thanks to the [Steam Forums](http://forums.steampowered.com/forums/showthread.php?t=1854268) for how to solve this one.

**Bonus**

Valve has released part 1 and part 2 (part 3 is coming) of the [Official Soundtrack](http://www.thinkwithportals.com/music.php) for free! The songs above aren't included and the Valve soundtracks are mp3s, so if you're looking for higher quality you're going to have to dig through the source files.  It probably won't be worth it though, because many of the sound track songs are split into multiple source files.

~Matt
