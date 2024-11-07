# Fixing Audio Issues on Lenovo Legion Pro 7 for Fedora 41 (Workstation Edition)

For Ubuntu 24.04 LTS Desktop users, please check [Ubuntu Instructions](README-Ubuntu.md)

## Problem Description
- No audio devices detected in GNOME Settings
- No sound output from speakers
- PipeWire/WirePlumber connection issues
- Affected System: Fedora 41 Workstation Edition on Lenovo Legion Pro 7
- Hardware: Intel Corporation Raptor Lake High Definition Audio Controller and NVIDIA Audio

## Solution Steps

### 1. Clean Existing Audio Configuration
```bash
# Stop all audio services
systemctl --user stop pipewire pipewire-pulse wireplumber

# Remove existing configurations
rm -rf ~/.config/pipewire
rm -rf ~/.local/state/pipewire
rm -rf /run/user/$UID/pipewire
```

### 2. Reinstall Audio Packages
```bash
# Reinstall core audio packages
sudo dnf reinstall pipewire
sudo dnf reinstall pipewire-pulseaudio
sudo dnf reinstall wireplumber
sudo dnf reinstall pipewire-utils
sudo dnf reinstall pipewire-libs
sudo dnf reinstall pipewire-alsa
```

### 3. Create Fresh Configuration Directories
```bash
# Create necessary directories
mkdir -p ~/.config/pipewire/media-session.d
mkdir -p ~/.config/wireplumber

# Copy default configurations
cp -R /usr/share/pipewire/* ~/.config/pipewire/
cp -R /usr/share/wireplumber/* ~/.config/wireplumber/
```

### 4. Configure ALSA and Intel HDA
```bash
# Create ALSA base configuration
sudo bash -c 'cat > /etc/modprobe.d/alsa-base.conf << EOL
options snd-hda-intel model=laptop-amic power_save=0 power_save_controller=N
options snd-hda-intel probe_mask=1
options snd-hda-intel position_fix=1
EOL'

# Set default sound card
sudo bash -c 'cat > /etc/modprobe.d/default.conf << EOL
options snd_hda_intel index=0
EOL'
```

### 5. Add User to Audio Group
```bash
sudo usermod -a -G audio $USER
```

### 6. Configure Audio Settings
```bash
# Enable both speakers and headphones
amixer -D hw:0 sset Headphone on
amixer -D hw:0 sset Speaker on

# Enable Auto-Mute Mode
amixer -D hw:0 sset "Auto-Mute Mode" Enabled
```

### 7. Start Audio Services
```bash
# Unmask and enable audio services
systemctl --user unmask pipewire pipewire-pulse wireplumber
systemctl --user enable --now pipewire.socket pipewire.service
systemctl --user enable --now pipewire-pulse.socket pipewire-pulse.service
systemctl --user enable --now wireplumber.service
```

### 8. Final Steps
1. Reboot the system
2. After logging in, verify audio is working:
```bash
# Set volume and unmute
pactl set-sink-volume alsa_output.pci-0000_00_1f.3.analog-stereo 100%
pactl set-sink-mute alsa_output.pci-0000_00_1f.3.analog-stereo 0

# Test audio
speaker-test -t wav -c 2

# You need to press Ctrl+C to stop the speaker-test!
```

### Verification
- Check if audio devices appear in GNOME Settings
- Test system sounds
- Test browser audio (YouTube, etc.)
- Test both speakers and headphones

## Additional Notes
- This solution fixes issues with both system audio and browser audio playback
- The configuration is specifically tested on Lenovo Legion Pro 7 with Fedora 41 (Workstation Edition), and might work with Fedora 40 (Workstation Edition) as well (but not tested!)
- The solution addresses both Intel and NVIDIA audio controllers

## Share & Support
If you found this solution helpful:
- ⭐ Star this repository to help others find it
- 🔄 Share it with other Linux users, particularly those with Lenovo Legion laptops
- 🐛 If you find any issues or have improvements, feel free to open an issue or submit a pull request
- 📢 Spread the word on Linux forums and social media to help others facing similar audio issues

Your support helps make the Linux community stronger! 💪

## Credits
This solution was developed with assistance from Claude AI [Claude 3.5 Sonnet - new] by [Anthropic](https://www.anthropic.com) as of today 7th November 2024 and has been tested on Fedora 41 (Workstation Edition) running on Lenovo Legion Pro 7.

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
