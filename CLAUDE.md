# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Neovim plugin that provides text-to-speech (TTS) functionality using Microsoft Edge's TTS engine (edge-tts). It allows users to select text in Neovim and have it read aloud or saved to an audio file.

## Key Commands

### Development Setup
```bash
# Install Python dependencies
pip install -r requirements.txt
# or
pip install edge-tts

# Test the Python TTS script directly
python tts.py "Hello world" "en-GB-SoniaNeural" 1.0
python tts.py "Hello world" "en-GB-SoniaNeural" 1.0 output.mp3  # Save to file

# List available voices
edge-tts --list-voices
```

### Neovim Plugin Testing
```vim
" Load the plugin in Neovim
:lua require('tts-nvim').setup({voice = 'en-GB-SoniaNeural', speed = 1.0})

" Test TTS on visual selection
:'<,'>TTS

" Test saving to file
:'<,'>TTSFile
```

## Architecture

The plugin consists of two main components:

### 1. Python Backend (`tts.py`)
- Handles the actual TTS functionality using the edge-tts library
- Takes command-line arguments: text, voice, speed rate, and optional output filename
- Can either stream audio to ffplay for immediate playback or save to an MP3 file
- Speed rate is converted from a float (e.g., 1.5) to edge-tts percentage format (e.g., "+50%")

### 2. Lua Plugin Layer
The Lua components work together to integrate TTS into Neovim:

- **plugin/tts-nvim.lua**: Registers the `:TTS` and `:TTSFile` user commands
- **lua/tts-nvim/init.lua**: Main module that:
  - Executes the Python script using plenary.nvim's Job API
  - Dynamically locates the Python script relative to the Lua module
  - Provides `tts()` for audio playback and `tts_to_file()` for saving to MP3
  - Exposes `setup()` for configuration

- **lua/tts-nvim/config.lua**: Configuration management
  - Default settings: `voice = "en-GB-SoniaNeural"`, `speed = 1.0`, `python_path = "python3"`
  - Merges user options with defaults during setup
  - `python_path` allows users to specify custom Python interpreter (e.g., for virtual environments)

- **lua/tts-nvim/util.lua**: Visual selection utilities
  - `getVisualSelection()`: Captures visual selection coordinates
  - `getTextFromSelection()`: Extracts text from visual selection, handling multi-line selections

## Plugin Dependencies

- **Neovim**: Plugin host
- **plenary.nvim**: Used for async job execution
- **edge-tts**: Python library for TTS functionality
- **ffplay** (part of ffmpeg): For audio playback

## Common Development Tasks

### Adding New Voices
Modify the default voice in `lua/tts-nvim/config.lua:4` or pass it during setup.

### Adjusting Speed Range
The speed conversion happens in `tts.py:10`. Current formula: `(speed - 1) * 100` converts user speed (e.g., 1.5) to edge-tts format (+50%).

### Changing Output Filename
The hardcoded filename "tts.mp3" is in `lua/tts-nvim/init.lua:32`. Consider making this configurable through options.

### Error Handling
Errors from the Python script are captured via stderr callback in `lua/tts-nvim/init.lua:16-20` and printed to Neovim's message area.

## Important Implementation Details

1. **Python Script Execution**:
   - The plugin dynamically determines the Python script path using `debug.getinfo()` relative to the Lua module location (`init.lua:11,29`)
   - Uses the configured `python_path` (default: "python3") to execute the script
   - The Python interpreter is called with the script path as the first argument, followed by TTS parameters

2. **Visual Selection Handling**: The plugin properly handles both single-line and multi-line visual selections, extracting the exact selected text including partial line selections.

3. **Async Execution**: TTS operations run asynchronously using plenary.nvim's Job API, preventing Neovim from blocking during audio playback.

4. **Speed Parameter**: The speed value is user-friendly (1.0 = normal, 1.5 = 50% faster) and gets converted to edge-tts's percentage format internally.

5. **Python Path Configuration**: Users can specify a custom Python interpreter path to support virtual environments, conda environments, or specific Python installations. This is crucial for environments where the default `python3` command may not have the required `edge-tts` package installed.