xinput-toggle is a simple script to toggle devices on and off for X11 systems.
It was created to toggle Yubikey state, but it can deal with any X input
device.

## Usage

I suggest you bind xinput-toggle to a keybinding in your window manager if you
will use it a lot.

For these examples I will show this being used to enable/disable a Yubikey.

```
# Toggle device status (on -> off, off -> on)
xinput-toggle -r '[Yy]ubikey'

# Show notification indicating actions performed
xinput-toggle -r '[Yy]ubikey' -n

# Disable xinput (no toggle)
xinput-toggle -r '[Yy]ubikey' -d

# Enable xinput (no toggle)
xinput-toggle -r '[Yy]ubikey' -e

# Enable for 5 seconds, then disable again
xinput-toggle -r '[Yy]ubikey' -e -t 5
```

## Requirements

- xinput
- bash 4+
- notify-send (optional, for `-n`)
