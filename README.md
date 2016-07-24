# Welcome Home

## Introduction

When I got myself a [Pine64](https://www.pine64.com/), I wondered for a bit about what to do with it. [@RaitoBezarius](https://github.com/RaitoBezarius) suggested that I set up a music server on it, which got my attention as I don't have a lot of hardware to buy for that (none, actually, except if you want to plug in a cheap sound system).

I set up a [Mopidy](https://www.mopidy.com/) server on the card, installed with Pip (since the Pine64 runs an ARM64 architecture and Mopidy's Debian/Ubuntu packages don't support it).

When I got that done, I wanted to enhance it. Since I didn't want just some central sound system I could control from my phone (really, my apartment is not that big, I don't need that), I thought of some system which would detect when I get home after work and automatically start my music.

## Welcome Home

Welcome Home is a bash script which detects when a selected device is connected to your local network. If it detects the device, it will communicate with Mopidy and start playing a selected stream (set in the configuration file) on the Pine64's jack output.

In my example, when I get home and my phone connects to my apartment's wifi, it gets a static IP address (as I configured it in my DHCP server). Welcome Home detects it, which means that I'm home or almost there, and starts some nice Trance music for me.

When Welcome Home will stop detecting the device, it will automatically tell Mopidy to stop playing the stream. Which means that if I want to stop the music and am too lazy to turn off my sound system, I can also stop the wifi on my phone.

Detection of both connection and disconnection takes up to 6s maximum with the default settings.

## Files

There are three files in this repository (except for those in the `mopidy` directory which are Pine64-related):

* `welcomehome` contains the bash script and is to move to `/usr/local/bin`.
* `welcomehome.example.conf` is a configuration example, and is to edit as described below and to move as `/etc/welcomehome/welcomehome.conf` (you'll need to create the `/etc/welcomehome` directory).
* `welcomehome.service` is the systemd service for Welcome Home, and is to move to `/etc/systemd/system`

## Configuration

Right now, there's only three configurable values:

* `device_address` is the static IP address bound to the device you wish to detect.
* `device_threshold` is the detection threshold. It defaults to 3. A greater threshold will mean a connection or a disconnection will take more time to detect, but you'll be able to detect them with more accuracy if your device's wifi is working weakly.
* `stream` is the URL of the audio stream you want Welcome Home to play when you get home. For now, it has to be an HTTP(S) stream.

For more info on the configuration syntax, have a look at the `welcomehome.example.conf` file.

## Starting Welcome Home

Once you moved all the needed files, and filled your configuration file, you need to start Welcome Home as a daemon. I use [systemd](https://freedesktop.org/wiki/Software/systemd/) for that (with the `welcomehome.service` file), but you're free to use whatever you want (and even to send my your configuration file in a [pull request](https://github.com/babolivier/welcome-home/pulls) so I can add it to this repi :wink:).

To start Welcome Home, and launch it at the machine's startup, with systemd, just run:

```shell
systemctl enable welcomehome
systemctl start welcomehome
```

## Bonus: Running Mopidy on Pine64

I had some troubles setting up Mopidy on my Pine64, so here's how I did it.

### No sound playing

I had some troubles setting up the Pine's soundcard, but finally managed it by installing the latest Ubuntu image, and following [these instructions](http://forum.pine64.org/showthread.php?tid=1832&pid=16434#pid16434).

### Binding with alsa

After following the instructions above, I also installed the `gstreamer1.0-alsa` package, which got me some nice ALSA plugins for Gstreamer. I edited the `/usr/share/alsa/alsa.conf` file and set both the `defaults.ctl.card` and the `defaults.pcm.card` properties from 0 to 1, so ALSA doesn't have to wonder too much about what soundcard to use (0 is the HDMI output and 1 is the jack output). I then set Mopidy's configuration file to what you can find in `mopidy/mopidy-pine64.conf`. It's basically the default configuration file, but with the right bindings.

### Launch Mopidy at startup

Since you installed Mopidy from sources, there's no provided way to automatically launch Mopidy when your Pine64 starts. I did it with the systemd service file you can find in `mopidy/mopidy.service`, which I moved to `/etc/systemd/system`. Then, the usual (as root):

```shell
systemctl enable mopidy
systemctl start mopidy
```