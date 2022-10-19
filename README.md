# podman-nginx-dash-hls

Dash and HLS streaming server using Podman and Nginx.

## Setup

Create a new user for Podman.

`useradd nginx`

Give your new user systemd linger privleges.

`loginctl enable-linger nginx`

Clone the repo to your home folder.

`git clone https://github.com/codebam/podman-nginx-dash-hls.git /home/nginx`

Configure your DNS to point to your server IP.

`A dash.your-website.com 127.0.0.1`

Generate LetsEncrypt certificates.

```
podman run -it --rm \
    --name certbot \
    -v /home/nginx/etc/letsencrypt:/etc/letsencrypt:z \
    -v /home/nginx/var/lib/letsencrypt:/var/lib/letsencrypt:z \
    -v /home/nginx/usr/share/nginx:/usr/share/nginx:z \
    certbot/certbot certonly --webroot-path /usr/share/nginx/html
```

Modify your nginx config.

`etc/nginx/sites-available/default`

Create stream directory.

`mkdir /home/nginx/stream`

Enable systemd services.

`systemctl --user enable --now nginx nginx-hls certbot.timer`

Stream.

```
live="rtmp://dash.seanbehan.ca/live_your_password/stream"

while true
do
	# notify-send -u critical "stream started"
        ffmpeg -f mpegts -i udp://localhost:2000?buffer_size=1000000000 \
            -c:v libx264 -x264opts bitrate=10000:vbv-maxrate=50000:vbv-bufsize=150000 -rtbufsize 150M \
            -crf 30 -profile:v high -preset:v ultrafast -tune zerolatency -movflags faststart \
            -c:a aac \
            -f flv $live
	sleep 0.1
done
```

Put your stream URL in dash.js or hls.js, or VLC.

`https://dash.seanbehan.ca/dash/stream.mpd`

`https://dash.seanbehan.ca/hls/stream.m3u8`

