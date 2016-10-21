yubikey-toggle is a simple script to toggle your Yubikey on and off for X11
systems.

## Usage

I suggest you bind yubikey-toggle to a keybinding in your window manager if you
will use it a lot.

```
# Toggle yubikey status
yubikey-toggle

# Show notification indicating actions performed
yubikey-toggle --notify
```

## Requirements

- xinput
- bash 4+
- notify-send (optional, for `--notify`)
