# Spotifyd <!-- omit in toc -->


> An open source Spotify client running as a UNIX daemon.

[Project Website](https://spotifyd.rs)

Spotifyd streams music just like the official client, but is more lightweight and supports more platforms. Spotifyd also supports the Spotify Connect protocol, which makes it show up as a device that can be controlled from the official clients.

> __Note:__ Spotifyd requires a Spotify Premium account.

## Common issues

- Spotifyd will not work without Spotify Premium
- The device name cannot contain spaces



# Spotifyd Setup on aarch64

This guide covers the setup of `spotifyd` on a Linux server with an aarch64 architecture, including PulseAudio configuration and troubleshooting steps.

## Prerequisites

- A Linux server with aarch64 architecture
- A valid Spotify account

## Step 1: Install Dependencies

First, install the necessary dependencies:

```sh
sudo apt-get update
sudo apt-get install build-essential libasound2-dev libssl-dev alsa-utils liboss4-salsa-asound2 pulseaudio
```

## Step 2: Install Rust and Cargo

If you don't have Rust and Cargo installed, install them using the following command:

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

## Step 3: Clone and Build Spotifyd

Clone the `spotifyd` repository and build it:

```sh
git clone https://github.com/Spotifyd/spotifyd.git
cd spotifyd
cargo build --release --no-default-features --features pulseaudio_backend,dbus_keyring,dbus_mpris
```

Move the built binary to a directory in your PATH:

```sh
sudo mv target/release/spotifyd /usr/local/bin/
```

## Step 4: Create Configuration File

Create the configuration file for `spotifyd`:

```sh
mkdir -p ~/.config/spotifyd
nano ~/.config/spotifyd/spotifyd.conf
```

Add the following content to the configuration file:

```toml
[global]
username = "your_spotify_username"
password = "your_spotify_password"
backend = "pulseaudio"
device_name = "MySpotifydDevice"
bitrate = 160
cache_path = "/home/your_username/.cache/spotifyd"
```

Replace `your_spotify_username` and `your_spotify_password` with your Spotify credentials and `your_username` with your actual username.

## Step 5: Configure PulseAudio

Ensure PulseAudio is installed and configured correctly:

1. **Start PulseAudio**:

   ```sh
   pulseaudio --start
   ```

2. **Edit PulseAudio Configuration**:

   ```sh
   sudo nano /etc/pulse/default.pa
   ```

   Uncomment or add the following lines to enable TCP connections:

   ```sh
   load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1
   load-module module-native-protocol-tcp auth-anonymous=1
   ```

3. **Restart PulseAudio**:

   ```sh
   pulseaudio --kill
   pulseaudio --start
   ```

## Step 6: Create Systemd Service

Create a systemd service file for `spotifyd`:

```sh
sudo nano /etc/systemd/system/spotifyd.service
```

Add the following content:

```ini
[Unit]
Description=Spotifyd
After=network.target

[Service]
User=your_username
ExecStart=/usr/local/bin/spotifyd --no-daemon --config-path /home/your_username/.config/spotifyd/spotifyd.conf
Restart=on-failure
RestartSec=5
StartLimitIntervalSec=500
StartLimitBurst=5

[Install]
WantedBy=multi-user.target
```

Replace `your_username` with your actual username.

## Step 7: Enable and Start the Service

Reload the systemd daemon, enable, and start the `spotifyd` service:

```sh
sudo systemctl daemon-reload
sudo systemctl enable spotifyd
sudo systemctl start spotifyd
```

## Step 8: Verify the Service Status

Check the status of the `spotifyd` service:

```sh
sudo systemctl status spotifyd
```

## Troubleshooting

### DNS Resolution Issues

If `spotifyd` fails with DNS resolution errors:

1. Ensure network connectivity:

   ```sh
   ping google.com
   ```

2. Manually test DNS resolution for Spotify’s access point:

   ```sh
   nslookup ap.spotify.com
   ```

### PulseAudio Connection Refused

If `spotifyd` fails to connect to PulseAudio:

1. Ensure PulseAudio is running:

   ```sh
   pulseaudio --start
   ```

2. Verify PulseAudio configuration allows network access.

3. Check available sinks:

   ```sh
   pactl list sinks
   ```

4. Ensure correct permissions for PulseAudio and `spotifyd` configuration directories:

   ```sh
   mkdir -p /home/your_username/.cache/spotifyd
   chown -R your_username:your_username /home/your_username/.cache/spotifyd
   chmod -R 755 /home/your_username/.cache/spotifyd
   ```

## Manual Test

Run `spotifyd` manually with verbose logging to catch any immediate errors:

```sh
/usr/local/bin/spotifyd --no-daemon --config-path /home/your_username/.config/spotifyd/spotifyd.conf --verbose
```

## Playing Music

To play music, use any Spotify client (phone, desktop, or web) to connect to the `spotifyd` device:

1. Open Spotify on your device.
2. Go to "Devices Available" and select the `spotifyd` device.
3. Play your music.

## Conclusion

By following these steps, you should be able to set up `spotifyd` on your aarch64 Linux server, configure PulseAudio, and troubleshoot common issues. Enjoy streaming your music with `spotifyd`!
