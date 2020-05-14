## Music Downloader  :notes: :headphones: :dancer: :unicorn:

### Features

- download songs, albums, public playlists from Deezer.com (account is required, free plan is enough)
- download Spotify playlists (by parsing the Spotify website and download the songs from Deezer)
- download as zip file (including m3u8 playlist file)
- 320 kbit/s mp3s with ID3-Tags and album cover
- download songs via youtube-dl
- KISS front end
- MPD integration (use it on a Raspberry Pi!)
- simple REST api



### How to use it

There is a settings file template called `settings.ini.example`. You can specify the download directory with  `download_dir`. Pressing the download button only downloads the song/album/playlist. If you set `use_mpd=True` in the `settings.ini` the backend will connect to mpd (localhost:6600) and update the music database. Pressing the play button will download the music. If `use_mpd=True`  is set the mpd database will be updated and the song/album/playlist will be added to the playlist. In `settings.ini` `music_dir` should be the music root location of mpd. The `download_dir` must be a subdirectory of `music_dir`. 

As Deezer sometimes requires a captcha to login the auto login features was removed. Instead you have to manually insert a valid Deezer cookie to the `settings.ini`. The relevant cookie is the `sid` cookie. A dedicated thread in the background is used to keep the cookie alive.



### Run it as a service

We use it with nginx and [ympd](https://github.com/notandy/ympd) as mpd frontend

- / goes to ympd
- /d/ goes to the downloader

The deployment directory contains a systemd unit file and a nginx vhost config file. There is also a [patch](https://github.com/kmille/music-ansible/blob/master/roles/ympd/files/fix_header.patch) to add a link to the ympd frontend. The `debug` tab will show you the debug output of the app.



### Try it out

```bash	
vagrant up
vagrant ssh
sudo vim /opt/deezer/app/settings.ini # insert your Deezer cookie
/opt/deezer/app/venv/bin/python /opt/deezer/app/app.py # start the backend

# On the host:
xdg-open http://localhost:5000 # view frontend in the browser
ncmpcpp -h 127.0.0.1 # try the mpd client
```


### Deployment
```bash
apt-get update -q
apt-get install -qy vim tmux git gcc
apt-get install -qy python3-virtualenv python3-dev
git clone https://github.com/kmille/deezer-downloader.git /opt/deezer
python3 -m virtualenv -p python3 /opt/deezer/app/venv
source /opt/deezer/app/venv/bin/activate
pip install -r /opt/deezer/requirements.txt
pip install -U youtube-dl
cp /opt/deezer/app/settings.ini.example /opt/deezer/app/settings.ini
# Adjust /opt/deezer/app/settings.ini
/opt/deezer/app/venv/bin/python /opt/deezer/app/app.py

# for mpd
apt-get install -yq mpd ncmpcpp
# adjust the file paths in /etc/mpd.conf and settings.ini
systemctl restart mpd
ncmpcpp -h 127.0.0.1
```



### Shortcuts
ctrl-m: focus search bar  
Enter: serach for songs   
Alt+Enter: search for albums  
ctrl-b: go to / (this is where our ympd is)  
ctrl-shift-[1-7] switch tabs    



### Some screenshots

Search for songs. You can listen to a 30 second preview in the browser.  

![](/screenshots/2020-05-13-211356_screenshot.png)  

Search for albums. You can download them as zip file.  

![](/screenshots/2020-05-13-213544_screenshot.png)

List songs of an album.

![](/screenshots/2020-05-13-211528_screenshot.png)

Download songs with youtube-dl  

![](/screenshots/2020-05-13-211622_screenshot.png)

Download a Spotify playlist.   

![](/screenshots/2020-05-13-211629_screenshot.png)  

Download a Deezer playlist.    

![](/screenshots/2020-05-13-211633_screenshot.png)  

ncmpcpp mpd client.  

![](/screenshots/2020-05-13-212025_screenshot.png)  



### Tests

python -m unittest -f tests.py  



### Deployment with ansible (includig mpd and ympd)
https://github.com/kmille/music-ansible (almost always outdated)



### Changelog

#### Version 1.1 (13.05.2020)

- thanks to [luelista](https://github.com/luelista) for the contribution!
- play 30 second preview in browser
- add Vagrantfile
- show album cover in search results
- use a threaded queue for download tasks
- list album songs