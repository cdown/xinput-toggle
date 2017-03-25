xinput-toggle is a simple script to toggle devices on and off for X11 systems.
It was created to toggle Yubikey state, but it can deal with any X input
device.

## Usage

`xinput-toggle -h` contains detailed usage information.

I suggest you bind xinput-toggle to a keybinding in your window manager if you
will use it a lot.

For these examples I will show this being used to enable/disable a Yubikey
(`-r` is a case-insensitive regex used to identify devices by names, as shown
by `xinput list`).

```
# Toggle device status (on -> off, off -> on)
xinput-toggle -r yubikey

# Show notification indicating actions performed
xinput-toggle -r yubikey -n

# Disable xinput (no toggle)
xinput-toggle -r yubikey -d

# Enable xinput (no toggle)
xinput-toggle -r yubikey -e

# Enable for 5 seconds, then disable again
xinput-toggle -r yubikey -e -t 5
```

## Requirements

- xinput
- bash 4+
- notify-send (optional, for `-n`, aka libnotify)
