StreamShuttle - A Privacy-First Approach to Home Security and Automation.

Modern NVR with object/motion/audio detection, push notifications, multi-location, and encrypted local and cloud-based storage support built in. Checkout https://streamshuttle.com to learn more.

# Before starting

Make sure the `LICENSE` environment variable is set correctly before continuing. You can find your license key under the "Manage Subscriptions" section in the "My Account" area @ https://streamshuttle.com . (*a yearly subscription or one time purchase is required to utilize the docker based installation*)
 - To add your license simply rename or copy the example file `env.example` to `env` inside the `conf` directory. Then just update the `LICENSE` variable with your license key.

Make sure you update the `shuttle-conf` service volume entry (in the `compose.yml` file) so that it points to the correct AppImage.
 - Head over to the <a href="https://streamshuttle.com/shop/release/release/index">Downloads</a> area @ https://streamshuttle.com .
 - Grab the latest Linux release and move it into the `./app_image/` directory relative to this file.
 - Then simply modify the following line to reflect the correct version.
    - `./app_image/StreamShuttle-0.0.7.AppImage:/app/app_image/StreamShuttle.AppImage`
      - Change `0.0.7` to reflect the version you have download.

If you intend to use push notifications you can set the `PUSH_NOTIFICATION_KEY` environment variable. You can find the push notification key under in the "My Account" area @ https://streamshuttle.com .
 - You can also add this later via the WebUI instead.

## Start StreamShuttle (using the regular hub)
`docker compose up -d`

## Start StreamShuttle (using the cuda enabled hub)
`docker compose -f compose.yml -f compose.cuda.yml up -d`
 - alternatively you could just replace the "shuttle-hub" config from the compose.cuda.yml into the primary compose.yml
and use without specifying the compose configuration files.

# Common configuration changes

## Utilize a specific or external disk for all streamshuttle_data
Simply swap out the volume config in the compose.yml file with the following. Where `/disk1/streamshuttle_data` is the path to where you want the data to be stored.
 - make sure the full path exists before starting
```
volumes:
  streamshuttle_data:
    driver: local
    driver_opts:
      o: bind
      type: none
      device: /disk1/streamshuttle_data
```

## Changing a specific hubs storage directory (*Before running for the first time*)
Add the desired volume to the shuttle-hub service in the compose.yml (or compose.cuda.yml) file

ex. under the "volumes" key add a new entry
```
    volumes:
      - streamshuttle_data:/data
      - /mnt/disk1/hub-1-data:/mnt/hub-1-data
```
- open the configuration file that is in this repository.
 - `/hub/config/hub-1.json`
 - and replace the `defaultStorageRoot` option with the new path.
 - if following the example, we simply change  `/data/shuttle_storage` to `/mnt/hub-1-data`


## Changing a specific hubs storage directory (*After having previously started StreamShuttle*)
Make sure all containers are stopped
`docker compose stop`

Add the desired volume to the shuttle-hub service in the compose.yml (or compose.cuda.yml) file

ex. under the "volumes" key add a new entry
```
    volumes:
      - streamshuttle_data:/data
      - /mnt/disk1/hub-1-data:/mnt/hub-1-data
```

Now modify the hub configuration file so that the storage directory
points to the desired path.

- open the configuration file that is in this repository.
 - `nano /hub/config/hub-1.json`
 - and replace the `defaultStorageRoot` option with the new path.
 - if following the example, we simply change  `/data/shuttle_storage` to `/mnt/hub-1-data`

If hub data already existed make sure it is copied into the correct location
 - run the shuttle-conf container with the new volume mount
  - `docker compose run --rm -v /mnt/disk1/hub-1-data:/mnt/hub-1-data shuttle-conf bash`
  - then run the following to move the data
  - `mv /data/shuttle_storage/* /mnt/hub-1-data/`

## How to upgrade
First grab the latest release from https://streamshuttle.com and add it to the `app_image` directory.

Make sure all the containers are stopped.
`docker compose stop`

Modify the app image volume mount version for `shuttle-conf` service.

- `./app_image/StreamShuttle-0.0.7.AppImage:/app/app_image/StreamShuttle.AppImage`
  - Change `0.0.7` to reflect the version you have download.

Execute the following command which ensures the latest version will be installed on the next start.
```
docker compose run --rm --entrypoint='' shuttle-conf bash -c 'rm -rf /data/app_root/squashfs-root'
```
