# Fix for FiiO K3 auto power-off on Arch Linux

The FiiO K3 has an auto power-off "feature" or low power mode which disables it when there is no audio signal keeping it alive.
It will make an audible click every time a new sound is played, like a notification.
The sound gets cut off because of the startup delay which can be annoying.

This workaround fixes the auto power off "feature" by constantly playing null audio via `ffplay`.


## How it works

1. The udev rule triggers if the FiiO K3 is added/removed from the system and executes the script `fiio-k3-fix`.
2. The script will try to get the current user running `pulseaudio` and start or stop the service accordingly.
3. The service runs `ffplay` to constantly play null audio which keeps the FiiO K3 alive.

## Install

Disconnect the FiiO K3 and run the following commands to install the fix.

```bash
# add service
cp fiio-k3-fix.service ~/.config/systemd/user

# enable service
systemctl --user enable fiio-k3-fix.service

# udev script
sudo cp fiio-k3-fix /usr/local/bin
sudo chmod +x /usr/local/bin/fiio-k3-fix

# add udev rule
sudo cp 91-fiio-k3-added.rules /etc/udev/rules.d

# reload rules
sudo udevadm control --reload
```

## Testing

Connect the FiiO K3 again and turn it on. The service status should now show `ffplay` running.

```bash
systemctl --user status fiio-k3-fix.service
```

```bash
$ systemctl --user status fiio-k3-fix.service
● fiio-k3-fix.service - Fiio K3 Fix
     Loaded: loaded (/home/r15ch13/.config/systemd/user/fiio-k3-fix.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2021-07-17 22:50:16 CEST; 3min 30s ago
   Main PID: 84870 (ffplay)
      Tasks: 38 (limit: 38451)
     Memory: 8.7M
        CPU: 1.379s
     CGroup: /user.slice/user-1000.slice/user@1000.service/app.slice/fiio-k3-fix.service
             └─84870 /usr/bin/ffplay -f lavfi -i anullsrc -nodisp

Jul 17 22:50:17 Bluescreen ffplay[84870]:   Stream #0:0: Audio: pcm_u8, 44100 Hz, stereo, u8, 705 kb/s
Jul 17 22:50:38 Bluescreen ffplay[84870]: [47.9K blob data]
```

