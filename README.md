# RaspberryPiCamera
Code for setting up a Camera and video stream to the web with a raspberry pi zero 2 w

## Techstack
Python, Flask, libcamera

## Components
[Raspberry Pi Zero 2 W](https://www.berrybase.de/raspberry-pi-zero-2-w) + SD-Card
[Raspberry Pi Camera Module 8MP v2](https://www.berrybase.de/raspberry-pi-camera-module-8mp-v2)
[Gehäuse](https://www.berrybase.de/offizielles-gehaeuse-fuer-raspberry-pi-zero-rot-weiss) - important for the included adapter to connect the camera to the pi zero
[Kamera-Mount](https://www.berrybase.de/mount-fuer-raspberry-pi-kameras-1-4-stativgewinde-inkl.-mini-stativ)

## Vorgehen
- flashing the sd-card with the software via raspberry imager
- ssh to the pi

'''
sudo apt install python3-flask -y
mkdir ~/cam_web
cd ~/cam_web
nano app.py

from flask import Flask, render_template, redirect, url_for
import subprocess
import os
from datetime import datetime

app = Flask(__name__)
VIDEO_DIR = "static/videos"
os.makedirs(VIDEO_DIR, exist_ok=True)

@app.route("/")
def index():
    videos = sorted(os.listdir(VIDEO_DIR), reverse=True)
    return render_template("index.html", videos=videos)

@app.route("/record")
def record():
    now = datetime.now().strftime("%Y%m%d_%H%M%S")
    filename = f"{VIDEO_DIR}/{now}.h264"
    subprocess.run(["libcamera-vid", "-t", "10000", "-o", filename])  # 10 Sek
    return redirect(url_for("index"))

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)


mkdir templates
nano templates/index.html

<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <title>Kamera-Webinterface</title>
</head>
<body>
    <h1>Raspberry Pi Kamera</h1>
    <a href="/record">🎥 Video aufnehmen (10s)</a>

    <h2>Aufgenommene Videos:</h2>
    <ul>
        {% for video in videos %}
            <li>
                <a href="{{ url_for('static', filename='videos/' + video) }}">{{ video }}</a>
            </li>
        {% else %}
            <li>Keine Videos vorhanden</li>
        {% endfor %}
    </ul>
</body>
</html>
'''


