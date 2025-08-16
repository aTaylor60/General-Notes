# Install
## System
DietPI Installed per guidance for specific device

To enable SCP for file interaction `OpenSSH Client` i.e. not OpenSSH server to enable SCP for file interaction

Add sound services through dietpi-config, this should install ALSA etc.

Install  ffmpeg
```
apt-get install ffmpeg
```
Install MediaMTX
```
wget https://github.com/bluenviron/mediamtx/releases/latest/download/mediamtx_linux_arm64.tar.gz
tar xzvf mediamtx_*_linux_arm64v8.tar.gz
sudo mv mediamtx /usr/local/bin/
```
Might need to point it at specific file as this was 404 despite finding the correct link

## Configuration
Identify the specific sound device with `arecord -L`
>hw:CARD=Device,DEV=0

Edit mediamtx.yml to configre the stream
```
paths:
  stream:
    runOnInit: ffmpeg -nostdin -f alsa -channels 1 -i hw:CARD=Device,DEV=0 -acodec pcm_s16le -ar 48000 -f rtsp -acodec pcm_s16be rtsp://localhost:8554/birdmic -rtsp_transport tcp
    runOnInitRestart: yes
```
ffmpeg does the heavy lift with the input format specified `-f alsa -channels 1 -i hw:CARD=Device,DEV=0 -acodec pcm_s16le -ar 48000`

alsa trips on assumption of 2 channels so specify 1 `-channels 1`, specify input to use as above `-i hw:CARD=Device,DEV=0` and create a 48kHz PCM stream `-acodec pcm_s16le -ar 48000`

Then output format is specified as rtsp `-f rtsp -acodec pcm_s16be rtsp://localhost:8554/birdmic -rtsp_transport tcp`

The stream, once up will be available at `rtsp://<IP ADDRESS>:8554/birdmic`

## Service
Install MediaMTX as service per:
https://github.com/bluenviron/mediamtx?tab=readme-ov-file#linux

Move the server executable and configuration in global folders:
```
sudo mv mediamtx /usr/local/bin/
sudo mv mediamtx.yml /usr/local/etc/
```
Create a systemd service:
```
sudo tee /etc/systemd/system/mediamtx.service >/dev/null << EOF
[Unit]
Wants=network.target
[Service]
ExecStart=/usr/local/bin/mediamtx /usr/local/etc/mediamtx.yml
[Install]
WantedBy=multi-user.target
EOF
```
Enable and start the service:
```
sudo systemctl daemon-reload
sudo systemctl enable mediamtx
sudo systemctl start mediamtx
```

# TODO:
* Reboot every night if needed?
