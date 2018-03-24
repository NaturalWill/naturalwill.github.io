---
title: install-kurento-server-in-ubuntu-16-04
date: 2018-03-07 12:00:55
tags:
---



    REPO="xenial"
    echo "deb http://ubuntu.kurento.org $REPO kms6" | sudo tee /etc/apt/sources.list.d/kurento.list
    wget http://ubuntu.kurento.org/kurento.gpg.key -O - | sudo apt-key add -
    apt-get update
    useradd -U -m kurento
    apt install kurento-media-server-6.0 openh264-gst-plugins-bad-1.5 -y
    nano /etc/kurento/modules/kurento/SdpEndpoint.conf.json
    systemctl start kurento-media-server-6.0
    systemctl enable kurento-media-server-6.0
