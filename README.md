Створення автономного DVR (відеореєстратора) на базі Raspberry Pi 5 із підключенням двох різних джерел — аналогового через USB-плату EasyCap MS2106


Запис через FFmpeg (Скріпт/Автоматизація)




start_dvr.sh
#!/bin/bash

# Шлях до вашої USB-флешки
TARGET_DIR="/media/prof/ESD-USB/dvr_records"
mkdir -p "$TARGET_DIR"

# Пауза для стабілізації USB при старті системи
sleep 5

# ЗАПИС З ANALOG FPV ТА ОДНОЧАСНИЙ ВИВІД НА ЕКРАН
# Налаштовано на утримання потоку при перешкодах (без синього екрана)
ffmpeg -f v4l2 -input_format mjpeg -video_size 640x480 -framerate 30 \
-fflags +genpts+discardcorrupt+nobuffer -max_delay 100000 -i /dev/video0 \
-vf "yadif=0:-1:0,format=yuv420p" -c:v libx264 -preset ultrafast -crf 26 -g 30 -sn -an \
-f segment -segment_time 600 -segment_format mp4 -strftime 1 -reset_timestamps 1 \
"$TARGET_DIR/fpv_%Y-%m-%d_%H-%M-%S.mp4" \
-c:v copy -f nut - | ffplay -loglevel warning -vf setpts=0 -window_title "FPV Live Preview" -alwaysontop -






ustreamer.service
[Unit]
Description=uStreamer FPV Live Stream Service
After=network.target

[Service]
Type=simple
User=prof
ExecStart=/usr/bin/ustreamer --device=/dev/video0 --format=mjpeg --resolution=640x480 --desired-fps=30 --host=0.0.0.0 --port=8554 --allow-origin=* --workers=3
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.targe
