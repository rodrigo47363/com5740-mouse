# com5740-mouse

A small command-line helper to **connect, disconnect, pair and inspect a
Bluetooth mouse on Linux**, built on top of BlueZ (`bluetoothctl`).

It was written for a COM-5740 mouse, but it works with any Bluetooth mouse: the
target is either provided explicitly or auto-detected from devices that advertise
the `input-mouse` profile.

## Features

- Connect / disconnect / toggle, with bounded retries.
- `reconnect` that **does not disconnect first** (many mice sleep when the link
  drops and need to be touched to wake up).
- Pairing workflow: scan, pair, trust and connect.
- `scan` that lists only devices advertising a mouse profile.
- `status` showing the Bluetooth service, `rfkill` state, BlueZ properties and
  whether X11 sees the device.
- Every `bluetoothctl` call is bounded by a timeout, so the script never hangs.
- Optional X11 verification via `xinput`.

## Requirements

- Linux with BlueZ (`bluetoothctl`).
- `systemd` (the script uses `systemctl` for the Bluetooth service).
- `timeout` from coreutils (usually preinstalled).
- Optional: `xinput` (X11 detection), `rfkill` (adapter state).

Enable the Bluetooth service at boot:

```bash
sudo systemctl enable --now bluetooth
```

## Install

```bash
git clone https://github.com/rodrigo47363/com5740-mouse.git
cd com5740-mouse
chmod +x com5740-mouse
ln -sf "$PWD/com5740-mouse" "$HOME/.local/bin/com5740-mouse"
```

`~/.local/bin` is normally already in `PATH`. If it is not, add it to your shell
config or copy the script to a directory that is.

## Usage

```bash
com5740-mouse                # connect (and verify X11 detection)
com5740-mouse status         # detailed status
com5740-mouse reconnect      # recover the connection if it dropped
com5740-mouse disconnect
com5740-mouse toggle
com5740-mouse pair [MAC]     # pair a new mouse (scan + pair + connect)
com5740-mouse scan           # scan for 20 s and list detected mice
com5740-mouse --help
```

### First time

1. Put the mouse in pairing mode (usually hold its pairing button/wheel for
   3-5 seconds).
2. Find its address:

   ```bash
   com5740-mouse scan
   ```

3. Pair and connect:

   ```bash
   com5740-mouse pair AA:BB:CC:DD:EE:FF
   ```

After that, `com5740-mouse` (no arguments) connects it.

## Configuration

All values can be overridden through environment variables:

| Variable | Default | Description |
|---|---|---|
| `MOUSE_MAC` | auto-detect | Bluetooth address; skips auto-detection |
| `MOUSE_NAME` | from BlueZ | Name used in messages |
| `MOUSE_XNAME` | `<NAME> Mouse` | X11 input device name |
| `MOUSE_RETRIES` | `4` | Connection attempts |
| `MOUSE_DELAY` | `2` | Seconds between attempts |
| `MOUSE_BT_TIMEOUT` | `10` | Timeout per `bluetoothctl` call |

Examples:

```bash
MOUSE_MAC=AA:BB:CC:DD:EE:FF com5740-mouse status
MOUSE_RETRIES=8 MOUSE_DELAY=1 com5740-mouse
```

If you have more than one Bluetooth mouse and auto-detection picks the wrong
one, set `MOUSE_MAC` explicitly (for example in your shell profile).

## Notes

- `disconnect` and `toggle` put the mouse to sleep. Move it or click to wake it
  up before connecting again.
- `reconnect` only re-establishes a dropped link; it does not disconnect first.
- If `bluetoothctl` cannot reach the service, the script asks you to run
  `sudo systemctl enable --now bluetooth`.

## License

MIT. See [LICENSE](LICENSE).
