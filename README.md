# LP10 Force Direct Audio

## Effort

This effort provides a small standalone LP10 helper script that forces a
comparison unit into a direct hardware audio path. The goal is to make the LP10
usable as a clean comparison device where audio is sent directly to the ALSA
hardware endpoint without vendor playback pipelines, ALSA software mixing,
resampling, DSP, software volume, gain, or format conversion.

## What The Script Changes

The script `lp10-force-direct-audio.sh` runs in `force` mode by default.

It performs these actions on the LP10:

- Tries to remount `/`, `/etc`, `/root`, and `/opt` read-write.
- Stops known vendor audio init scripts such as AirPlay and LP10 NetAudio hooks.
- Terminates known audio processes such as AirPlay, Cast, Roon, Tidal, Qobuz,
  Bluetooth ALSA, GStreamer, `aplay`, and `lp10-netaudio`.
- Sends `kill -9` to remaining matching audio processes in force mode.
- Backs up existing ALSA configuration files:
  - `/etc/asound.conf`
  - `/etc/asoundrc`
  - `/root/.asoundrc`
- Writes a strict direct ALSA configuration to `/etc/asound.conf`.
- Removes `/etc/asoundrc` and `/root/.asoundrc` so they cannot override the
  forced direct path.
- Verifies that `/etc/asound.conf` targets the selected hardware card/device.
- Fails if active ALSA config contains known software-processing markers such
  as `plug`, `dmix`, `softvol`, `rate`, `resample`, `volume`, or `gain`.

## Default Direct Path

By default, the script targets:

```sh
hw:0,1
S16_LE
48000 Hz
2 channels
```

The generated ALSA default path is hardware-only:

```text
pcm.!default -> type hw, card 0, device 1
ctl.!default -> type hw, card 0
pcm.lp10_direct -> type hw, card 0, device 1
ctl.lp10_direct -> type hw, card 0
```

No `plug`, `dmix`, `softvol`, `route`, `rate`, DSP, gain, or software-volume
PCM is defined by the generated config.

## Usage

Copy the script to the LP10:

```sh
scp lp10-force-direct-audio.sh root@LP10_IP:/opt/lp10-force-direct-audio.sh
ssh root@LP10_IP chmod 700 /opt/lp10-force-direct-audio.sh
```

Force direct mode:

```sh
ssh root@LP10_IP sh /opt/lp10-force-direct-audio.sh
```

This is the same as:

```sh
ssh root@LP10_IP sh /opt/lp10-force-direct-audio.sh force
```

Check status:

```sh
ssh root@LP10_IP sh /opt/lp10-force-direct-audio.sh status
```

Restore the last backed-up ALSA configuration:

```sh
ssh root@LP10_IP sh /opt/lp10-force-direct-audio.sh restore
```

## Custom Target

To force a different exact ALSA hardware endpoint:

```sh
LP10_DIRECT_ALSA_DEVICE=hw:0,0 sh /opt/lp10-force-direct-audio.sh force
```

The script intentionally refuses non-hardware targets. Use exact `hw:X,Y`
devices only.

## Notes

This script configures the LP10 audio path for comparison. It does not by itself
prove bit-perfect playback. For proof, compare input and captured output with a
hash-based test for the exact source path under test.
