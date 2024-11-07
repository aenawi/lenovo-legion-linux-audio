# Fixing Audio Issues on Lenovo Legion Pro 7 for Ubuntu 24.04 LTS Desktop

⚠️ **IMPORTANT NOTICE** ⚠️
> This guide has NOT been tested on Ubuntu yet. It's adapted from the working Fedora solution and needs community testing and validation. If you test this solution, please share your results by creating an issue or pull request.

For Fedora users, please check the [Fedora Instructions](README.md) (tested and working).

## Testing Status
- ❓ Ubuntu 24.04 LTS Desktop: Needs testing
- ❓ Ubuntu 22.04 LTS Desktop: Needs testing
- ✅ Fedora 41 Workstation: Working (see [Fedora guide](README.md))

## Call for Testing
We need Ubuntu users with Lenovo Legion Pro 7 laptops to:
1. Test these instructions
2. Report success/failure
3. Suggest modifications if needed
4. Share their experiences

Please create an issue or pull request to share your results!

## Problem Description
- No audio devices detected in GNOME Settings
- No sound output from speakers
- PipeWire/WirePlumber connection issues
- Affected System: Ubuntu 24.04 LTS Desktop on Lenovo Legion Pro 7
- Hardware: Intel Corporation Raptor Lake High Definition Audio Controller and NVIDIA Audio

## Solution Steps

### 1. Install Required Packages
```bash
# Install PipeWire and required packages
sudo apt update
sudo apt install pipewire pipewire-audio-client-libraries \
    pipewire-pulse wireplumber \
    pipewire-media-session- pulseaudio- \
    alsa-utils pavucontrol
```

### 2. Clean Existing Audio Configuration
```bash
# Stop all audio services
systemctl --user stop pipewire pipewire-pulse wireplumber

# Remove existing configurations
rm -rf ~/.config/pipewire
rm -rf ~/.local/state/pipewire
rm -rf /run/user/$UID/pipewire
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

### 6. Enable PipeWire Service
```bash
# Enable PipeWire instead of PulseAudio
systemctl --user --now enable pipewire pipewire-pulse
systemctl --user --now enable wireplumber
systemctl --user mask pulseaudio
systemctl --user --now disable pulseaudio.service pulseaudio.socket
```

### 7. Configure Audio Settings
```bash
# Enable both speakers and headphones
amixer -D hw:0 sset Headphone on
amixer -D hw:0 sset Speaker on

# Enable Auto-Mute Mode
amixer -D hw:0 sset "Auto-Mute Mode" Enabled
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

I'll combine and organize the "Additional Notes" section to be more comprehensive:

## Additional Notes
- ⚠️ WARNING: This solution has NOT been tested on Ubuntu yet - community testing and feedback needed
- 🔧 This solution is adapted from the working Fedora 41 configuration
- 📝 Key Information:
  - The configuration is theoretically compatible with Lenovo Legion Pro 7 hardware
  - Should fix issues with both system audio and browser audio playback
  - Addresses both Intel and NVIDIA audio controllers

- 📌 Ubuntu Version-Specific Notes:
  - For Ubuntu 22.04 LTS users:
    ```bash
    # Add PipeWire PPA before following the main instructions
    sudo add-apt-repository ppa:pipewire-debian/pipewire-upstream
    sudo apt update
    ```
  - For Ubuntu 24.04 LTS users:
    - PipeWire is included by default
    - No additional PPA should be needed

- 🔍 After Installation:
  - Monitor system logs for any PipeWire or WirePlumber errors
  - Test all audio outputs (speakers, headphones, HDMI)
  - Check browser audio functionality
  - Verify microphone inputs if needed

- 🤝 Community Help:
  - If you test this solution, please share your results
  - Document any modifications needed for your setup
  - Report both successes and failures
  - Help improve these instructions for other Ubuntu users

## Share & Support
If you found this solution helpful:
- ⭐ Star this repository to help others find it
- 🔄 Share it with other Linux users, particularly those with Lenovo Legion laptops
- 🐛 If you find any issues or have improvements, feel free to open an issue or submit a pull request
- 📢 Spread the word on Linux forums and social media to help others facing similar audio issues

Your support helps make the Linux community stronger! 💪

## Credits
This solution was developed with assistance from Claude AI [Claude 3.5 Sonnet - new] by [Anthropic](https://www.anthropic.com) as of today 7th November 2024 **but has not been tested on Ubuntu 24.04 LTS Desktop running on Lenovo Legion Pro 7**.

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
