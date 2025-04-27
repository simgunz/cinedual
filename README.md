# CineDual

These scripts allow playing a single video file with two different audio tracks simultaneously outputting to two separate audio devices (sinks), keeping playback synchronized.

## Purpose

This is useful for scenarios like:

*   Watching a movie with one person using speakers (e.g., original language) and another using headphones (e.g., dubbed language or audio description track).
*   Any situation requiring two different audio streams from the same video file on separate outputs.

## Scripts

1. **`dualplay.sh`** (Recommended):
   * A unified script combining the functionality of both playsync.sh and controlsync.sh
   * Uses subcommands for better organization:
     * `play` - Start playback with two audio tracks
     * `control` - Control an existing playback session
   * Features interactive sink selection using `fzf`
   * Includes audio synchronization tools

2.  **`playsync.sh`**:
    *   Starts two `mpv` instances.
    *   Instance 1: Plays video + specified audio track on sink 1.
    *   Instance 2: Plays specified audio track *only* (no video) on sink 2.
    *   Uses IPC sockets (`/tmp/mpv_sync_*`) to allow control.
    *   Features interactive sink selection using `fzf`.

3.  **`controlsync.sh`**:
    *   Connects to the IPC sockets created by a running `playsync.sh` instance.
    *   Allows controlling playback (play/pause/seek/quit) from a separate terminal.

## Requirements

*   `mpv`: A versatile media player.
*   `socat`: Utility for data transfer between channels, used here for IPC communication with `mpv`.
*   `fzf`: Interactive fuzzy finder for sink selection.
*   `pactl`: PulseAudio command-line utility (included with PulseAudio or PipeWire).

Install them on Debian/Ubuntu derivatives:
```bash
sudo apt update
sudo apt install mpv socat fzf pulseaudio-utils
```
Or on Fedora:
```bash
sudo dnf install mpv socat fzf pipewire-utils
```

## Usage

### Using the unified script (recommended)

1.  **Find your audio track IDs (AID):**
    Run `mpv --list-tracks "/path/to/your/movie.mkv"`.
    
    Look for the `id` field for audio tracks (e.g., `1`, `2`). Only numeric track IDs are supported.

2.  **Start Playback:**
    ```bash
    ./dualplay.sh play "/path/to/your/movie.mkv" <video_aid> <audio_only_aid>
    ```
    
    Example:
    ```bash
    ./dualplay.sh play "MyMovie.mkv" 1 2
    ```

3.  **Control Playback:**
    Open a new terminal and run:
    ```bash
    ./dualplay.sh control "/path/to/your/movie.mkv"
    ```
    
    **Available commands:**
    - `play` - Resume playback
    - `pause` - Pause playback
    - `seek <seconds>` - Seek to a specific position in seconds
    - `delay <seconds>` - Set audio synchronization delay directly
    - `+` / `++` / `+++` - Increase audio delay by small/medium/large steps
    - `-` / `--` / `---` - Decrease audio delay by small/medium/large steps
    - `quit` - Exit MPV instances

4.  **Get Help:**
    ```bash
    ./dualplay.sh help
    ```

### Using the separate scripts (legacy)

1.  **Find your audio track IDs (AID):**
    Run `mpv --list-tracks "/path/to/your/movie.mkv"`.
    
    Look for the `id` field for audio tracks (e.g., `1`, `2`). Only numeric track IDs are supported.

2.  **Start Playback:**
    Run [`playsync.sh`](playsync.sh) with the movie file and audio track IDs:
    ```bash
    ./playsync.sh "/path/to/your/movie.mkv" <video_aid> <audio_only_aid>
    ```
    
    You will be prompted to interactively select audio sinks using `fzf`.
    
    *Examples:*
    ```bash
    ./playsync.sh "MyMovie.mkv" 1 2
    ```
    *(Where 1 is the first audio track for the video player, 2 is the second audio track for the audio-only player)*

3.  **Control from another terminal:**
    Open a new terminal and run [`controlsync.sh`](controlsync.sh) with the *exact same* movie file path:
    ```bash
    ./controlsync.sh "/path/to/your/movie.mkv"
    ```
    This script will find the running `mpv` instances and allow you to control them.

    **Available commands in controlsync.sh:**
    - `play` - Resume playback
    - `pause` - Pause playback
    - `seek <seconds>` - Seek to a specific position in seconds
    - `delay <seconds>` - Set audio synchronization delay directly
    - `+` - Increase audio delay by 0.1s (small adjustment)
    - `++` - Increase audio delay by 0.2s (medium adjustment)
    - `+++` - Increase audio delay by 0.4s (large adjustment)
    - `-` - Decrease audio delay by 0.1s (small adjustment)
    - `--` - Decrease audio delay by 0.2s (medium adjustment)
    - `---` - Decrease audio delay by 0.4s (large adjustment)
    - `quit` - Exit MPV instances

    **Note:** The `quit` command only tells the `mpv` instances to exit. It does *not* stop the original [`playsync.sh`](playsync.sh) script. It's usually better to press `Ctrl+C` in the terminal running [`playsync.sh`](playsync.sh) to stop everything and clean up.

## Audio Synchronization

After seeking to a different position in the video, you may notice that the audio tracks are not perfectly synchronized. The `controlsync.sh` script provides a delay compensation system to fix this:

- A positive delay value means the audio-only player is delayed relative to the video player
- A negative delay value means the audio-only player starts earlier than the video player

**Methods to adjust synchronization:**
1. Use `delay <seconds>` to set a specific delay value (e.g., `delay 0.3` or `delay -0.1`)
2. Use dedicated commands for incremental adjustment:
   - `+`, `++`, or `+++` to increase delay by small, medium, or large steps
   - `-`, `--`, or `---` to decrease delay by small, medium, or large steps

The delay settings are automatically saved to `~/.config/mpv_dual_audio/delay_settings.conf` and will be remembered between sessions.

## How it Works

The scripts launch two `mpv` processes: one for video and the specified video audio track, another for only the specified audio-only track. Both are started paused. It uses `--input-ipc-server` to create Unix domain sockets for each instance. After both sockets are ready, it sends commands via `socat` to unpause both instances with appropriate timing based on the delay compensation value.

The control functionality connects to these sockets to control both players synchronously, and provides tools to adjust the synchronization delay in real-time.

## Customization

*   **Audio Tracks:** Audio tracks are specified as command-line arguments using numeric track IDs.
*   **Audio Sinks:** Sinks are selected interactively using `fzf` when starting playback.
*   **Delay Compensation:** The default delay is 0.2 seconds but can be adjusted as needed.