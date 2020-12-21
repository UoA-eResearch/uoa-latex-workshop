---
layout: workshop 
title: "Automatically set your desktop background based on the weather at your current location (Ubuntu)"
date: 2019-05-28
tags: bash ubuntu api weather
---

*This one's a bit more involved than my usual `blog` posts, so I made it a `tutorial`... because I can.*

## Description
* Set a different desktop background based on the current weather at your location (queries a free weather API every 5 minutes by default).

##  Explanation
* The way the script works is quite simple: it `curl`s the [wttr.in](https://github.com/chubin/wttr.in/) weather API using the command `curl --silent wttr.in?format="%C"`.
* The API responds with a `single line` of output describing the weather at your current location, for example:

```shell
[sam@samantha] λ curl --silent wttr.in?format="%C"
Light Rain
```

* Based on this output, the script looks in a directory for an image with a filename matching the weather condition, e.g. `~/Pictures/weather-wallpapers/Light Rain.jpg` and then sets this as the wallpaper.
* This script simply has to be added as a `cron job` to execute every `5` minutes (or however often you want to query for weather updates).

## Setup & Code
* Setup requires 3 steps:
    1. Create a `weather-wallpapers/` folder and populate it with your desired background images for each weather condition.
    2. Save the `weather` script somewhere.
    3. Add a `cron job` to run the script every `5` minutes.

### 1. Create `weather-wallpapers/` folder
* Create a folder and fill it with your desired wallpapers for each weather condition.
* If your wallpaper ever goes black, that's because you don't have an image file for the current weather condition, simply run `curl --silent wttr.in?format="%C"` and add a new image file with the same name as the command's output.
* **Note:** If you're lazy like me, you can always duplicate your images for similar weather conditions, e.g. I have two identical images, `Light Rain.jpg`, and `Light Rain, Rain Shower.jpg`. Here's a few file names to get you started:

<img class="screenshot bordered"  src="/assets/misc/weather-wallpapers.png" alt="Example Weather Wallpapers" />

### 2. Save the `weather` script somewhere
* Save the following `weather` script somewhere, I saved it as `~/.rc/weather`:
* **Note:** the last line will need to be edited to point to your `weather-wallpapers/` folder.

```shell
#!/bin/bash

# Get the DBUS_SESSION_BUS_ADDRESS environment variable which gsettings needs but cron by default runs without
PID=$(pgrep gnome-session | tail -n1)
export DBUS_SESSION_BUS_ADDRESS=$(grep -z DBUS_SESSION_BUS_ADDRESS /proc/$PID/environ|cut -d= -f2-)

 # Get the current weather in your location
CUR_WEATHER=$(curl --silent wttr.in?format="%C")

# Look in a folder 'weather-wallpapers' for an image with a filename corresponding to the weather condition
# and sets that as the desktop background. NOTE: The path below will need to be updated to be edited to point to your folder.
gsettings set org.gnome.desktop.background picture-uri "file:///home/sam/Pictures/weather-wallpapers/${CUR_WEATHER}.jpg"
```

* To be safe I would also recommend running `chmod +x ~/.rc/weather` on the script file.

### 3. Add the script as a `cron job`
* To schedule the script to run periodically, simply edit your crontab (`crontab -e`), then append the line below to the end of the file.
* **Note:** Adjust the path to point to wherever you saved your `weather` script.

```shell
*/5 * * * * /home/sam/.rc/weather # Adjust the '5' to however often you want to query for weather updates.
```

* You may have to restart cron for the job to be added: `service cron restart`.
* **Note**: The above cron jobs will execute every 5th minute *of the hour*, i.e. 12:00, 12:05, 12:15 etc.

# Done! 😎