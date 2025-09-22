# RaspberryPiCamera
Code for setting up a Camera and video stream to the web with a raspberry pi zero 2 w

## Techstack
Python, Flask, libcamera

## Components
- [Raspberry Pi Zero 2 W](https://www.berrybase.de/raspberry-pi-zero-2-w) + SD-Card
- [Raspberry Pi Camera Module 8MP v2](https://www.berrybase.de/raspberry-pi-camera-module-8mp-v2)
- [Gehäuse](https://www.berrybase.de/offizielles-gehaeuse-fuer-raspberry-pi-zero-rot-weiss) - important for the included adapter to connect the camera to the pi zero
- [Kamera-Mount](https://www.berrybase.de/mount-fuer-raspberry-pi-kameras-1-4-stativgewinde-inkl.-mini-stativ)

## Procedure
- flashing the sd-card with the software via raspberry imager
- ssh to the pi

## Implementing video stream to a web page
### Shell commands
```
sudo apt install python3-flask python3-opencv python3-picamera2 -y
mkdir ~/cam_stream
cd ~/cam_stream
nano stream.py
```
### Python script
```
from flask import Flask, render_template, Response
import cv2
from picamera2 import Picamera2

app = Flask(__name__)
picam2 = Picamera2()
picam2.configure(picam2.create_video_configuration(main={"size": (640, 480)}))
picam2.start()

def gen_frames():
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_RGB2BGR)
        ret, buffer = cv2.imencode('.jpg', frame)
        frame = buffer.tobytes()
        yield (b'--frame\r\n'
               b'Content-Type: image/jpeg\r\n\r\n' + frame + b'\r\n')

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/video_feed')
def video_feed():
    return Response(gen_frames(),
                    mimetype='multipart/x-mixed-replace; boundary=frame')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```
### Shell commands
```
mkdir templates
nano templates/index.html
```
### HTML Script
```
<!DOCTYPE html>
<html>
<head>
    <title>Live Kamera Stream</title>
</head>
<body>
    <img src="{{ url_for('video_feed') }}" width="640" height="480">
</body>
</html>
```

## Autostart
### Shell Commands
```
sudo nano /etc/systemd/system/camstream.service
```
### Service Config
[User] has to be inserted
```
[Unit]
Description=Raspberry Pi Camera Livestream
After=network.target

[Service]
ExecStart=/usr/bin/python3 /home/[user]/cam_stream/stream.py
WorkingDirectory=/home/[user]/cam_stream
StandardOutput=inherit
StandardError=inherit
Restart=always
User=pi

[Install]
WantedBy=multi-user.target
```
### Shell Commands
```
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable camstream.service
sudo systemctl start camstream.service
sudo systemctl status
```

## Tailscale
- [Taiscale](https://login.tailscale.com/admin/machines)
- allows authenticated devices to connect to each other and be seen
- login with github


