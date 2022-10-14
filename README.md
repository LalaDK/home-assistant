# Home Assistant notes
## How to edit config.txt
```shell
// Mount SD-card
mkdir ~/mnt
sudo mount -t vfat /dev/sda1 ~/mnt
cd mnt
# Use your favorite editor (kak, nano, vi etc.)
kak config.txt
```

When done, remember to dismount the partition
```shell
sudo umount ~/mnt
```

### Increase GPU Memory for Raspberry Pi 4
In order to use hardware acceleration with Frigate AVR, the GPU memory needs to be increased to at least 256 MB. (I'm running HA on a RPi4)
```ini
// Use above method to edit config.txt
// Add the following to the bottom of the file, to increase to 512 MB
gpu_mem=512
```

## How to move data to SSD (External USB)
```
Settings > System > Storage > Three Dots Menu (upper right cornor) > Move datadisk
```

## Setup headless WiFi
```shell
# Mount SD-card
# Use the steps in step 1 to mount the first FAT-partition on the SD-card
sudo mkdir -p CONFIG/network
cd CONFIG/network
sudo touch my-network
```

Modify the following configuration and paste it into the previous created file: my-network
```ini
[connection]
id=my-network
uuid=a7cb6f7e-aea8-41ca-91e9-e3cc814248ea
type=802-11-wireless
 
[802-11-wireless]
mode=infrastructure
ssid=MY_SSID
# Uncomment below if your SSID is not broadcasted
#hidden=true
 
# Comment out this section if you have an open/public network that does not use a password
[802-11-wireless-security]
auth-alg=open
key-mgmt=wpa-psk
psk=MY_WLAN_SECRET_KEY
 
[ipv4]
method=auto
# Uncomment below if your using a static IP address and comment out the above method=auto
# You can set the IP address to what you like
#method=manual
#address=192.168.1.111/24;192.168.1.1
#dns=8.8.8.8;8.8.4.4;
 
[ipv6]
addr-gen-mode=stable-privacy
method=auto
```

## Zigbee2MQTT configuration for Conbee II
### MQTT
```yaml
server: mqtt://192.168.50.29:1883
user: mqtt
password: mqtt
```

### Serial
```yaml
port: >-
  /dev/serial/by-id/usb-dresden_elektronik_ingenieurtechnik_GmbH_ConBee_II_DE2497158-if00
adapter: deconz
```

>Use "ha hardware info" in terminal to find correct ID.

## frigate.yaml
```yaml
mqtt:
  host: 192.168.50.29
  user: mqtt
  password: mqtt

objects:
  track:
    - person
    - car
    - motorcycle
    - bird
    - cat
    - dog
    - horse
  filters:
    car:
      mask:
        - 484,134,453,228,310,204,315,130

detect:
  enabled: True
  width: 640
  height: 360
  fps: 5
record:
  enabled: True
  retain:
    days: 2
    mode: active_objects
  events:
    retain:
      default: 2
      mode: active_objects
snapshots:
  enabled: True
  timestamp: False
  bounding_box: True
  retain:
    default: 10
detectors:
  coral:
    type: edgetpu
    device: usb
    
cameras:
  garden:
    motion:
      mask:    
        - 540,101,640,92,640,0,0,0,0,115
    ffmpeg:
      inputs:
        - path: rtsp://USERNAME:PASSWORD@192.168.50.149:554/stream2
          roles:
            - detect
        - path: rtsp://USERNAME:PASSWORD@192.168.50.149:554/stream1
          roles:
            - record
            - rtmp
  garage:
    motion:
      mask:
        - 364,114,375,174,640,181,640,130,640,0,0,0,0,171
    ffmpeg:
      inputs:
        - path: rtsp://USERNAME:PASSWORD@192.168.50.208:554/stream2
          roles:
            - detect
        - path: rtsp://USERNAME:PASSWORD@192.168.50.208:554/stream1
          roles:
            - record
            - rtmp
```

## configuration.yaml
```yaml

# Loads default set of integrations. Do not remove.
default_config:

# Text to speech
tts:
  - platform: google_translate

automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml

lovelace:
  resources:
    - url: /hacsfiles/frigate-hass-card/frigate-hass-card.js
      type: module

sensor:
  - platform: integration
    source: sensor.ams_power
    name: energy_spent
    unit_prefix: k
    round: 2

homeassistant:
  customize_glob:
    sensor.ams_p:
      last_reset: '1970-01-01T00:00:00+00:00'
      device_class: energy
      state_class: measurement
    sensor.energy_spent:
      last_reset: '1970-01-01T00:00:00+00:00'
      device_class: energy
      state_class: total_increasing
      unit_of_measurement: kWh
```
