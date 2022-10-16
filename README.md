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
dash="rtmp://localhost:1935/dash/stream"
hls="rtmp://localhost:1936/hls/stream"

while true
do
	# notify-send -u critical "stream started"
	ffmpeg -re -start_at_zero -copyts -ss $(( $(ffprobe -v error -select_streams v:0 -count_packets -show_entries stream=nb_read_packets -of csv=p=0 /home/codebam/livestream/*.flv) / 60 )) -i /home/codebam/livestream/*.flv \
            -framerate 60 \
            -c:v libx264 -x264opts bitrate=1000:vbv-maxrate=50000:vbv-bufsize=50000 -rtbufsize 100M \
            -c:a copy -preset:v superfast -tune zerolatency -movflags faststart \
            -f flv "$dash" \
            -framerate 60 \
            -c:v libx264 -x264opts bitrate=1000:vbv-maxrate=50000:vbv-bufsize=50000 -rtbufsize 100M \
            -c:a copy -preset:v superfast -tune zerolatency -movflags faststart \
            -f flv "$hls"
	sleep 0.1
done
```

Put your stream URL in dash.js or hls.js, or VLC.

`https://localhost/dash/stream.mpd`

`https://localhost/hls/stream.m3u8`

