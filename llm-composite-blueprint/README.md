# LLM Composite Script for Music Assistant

This blueprint creates a single script that combines search and play functionality for Music Assistant voice control. It replaces both Option 3 (play script) and Option 4 (search script) from the main voice-support repository.

## What This Blueprint Does

The LLM Composite Script provides a unified way to control Music Assistant playback through voice commands:

1. **Search First**: The script searches your Music Assistant library for matching content
2. **Smart Response Logic**:
   - **Auto-Play Mode**: When `media_type` is specified and results exist, the first matching result is played automatically
   - **Return Choices Mode**: When `media_type` is not specified or the search is ambiguous, formatted choices are returned showing available options

## Key Features

- Single script handles both direct playback and search results
- Supports all media types: tracks, albums, artists, playlists, and radio
- Configurable result limits and default player
- Radio mode support for continuous playback
- Shuffle control
- Area and media player targeting

## Configuration

### Prerequisites

1. Music Assistant integration installed and configured
2. LLM integration set up (OpenAI Conversation, Google Generative AI, etc.)
3. LLM integration configured with home control enabled (so it can use scripts as tools)

### Installation

1. Import the Blueprint using the button below:

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fmusic-assistant%2Fvoice-support%2Fblob%2Fmain%2Fllm-composite-blueprint%2Fllm_composite_script.yaml)

2. Create a script using the blueprint
3. Configure the required settings (see below)
4. Save the script
5. **Expose the script to Assist** (critical step)

### Required Configuration

#### Music Assistant Instance
Select your Music Assistant integration instance from the dropdown.

#### Default Player (Optional but Recommended)
The default Music Assistant player to use when no target is specified in the voice request. Leave empty if you want the script to prompt for a target when none is provided.

### Optional Configuration

#### Default Result Limit
Maximum number of results to return per media type when showing choices (default: 3, range: 1-20).

#### Radio Mode
Determines continuous playback behavior:
- **Use player settings**: Uses the "Don't stop the music" setting on the Music Assistant Player
- **Always**: Continuously adds new songs to the playlist
- **Never**: Stops after the selected songs are finished

#### Prompt Customization
Customize the prompts for your specific LLM model if needed. Each parameter (media_id, media_type, artist, album, area, media_player, shuffle, limit) can have its prompt customized.

### Script Description

When creating the script, provide a clear description that the LLM will use to understand when to call it. Example:

> This script searches Music Assistant and either auto-plays results or returns available choices. The tool takes the following arguments: media_id, media_type, artist, album, area, media_player, shuffle, limit. media_id is always required. Use this tool whenever the user wants to play music.

## Usage

### How It Works

1. **Voice Command Processing**: The LLM parses your natural language request and extracts parameters
2. **Search Execution**: The script searches Music Assistant using the provided `media_id` (search query)
3. **Result Handling**:
   - If `media_type` is specified (track, album, artist, playlist, radio) AND results exist for that type: **auto-plays the first result**
   - If `media_type` is not specified OR no results match the specified type: **returns formatted choices**

### Auto-Play Mode

Use when you know exactly what you want:

- "Play the album Dark Side of the Moon by Pink Floyd in the kitchen"
- "Shuffle tracks by Beyonce in the living room"
- "Play the playlist Chill Vibes"

When `media_type` is specified (album, track, artist, playlist, radio), the script automatically plays the first matching result.

### Return Choices Mode

Use when you want to browse options:

- "Find music by The Beatles"
- "Search for jazz albums"
- "Show me rock playlists"

When `media_type` is omitted or the search is ambiguous, the script returns a formatted list of available options across all media types (artists, albums, tracks, playlists, radio).

### Target Determination

The script determines where to play using this priority:

1. **Area or player mentioned in request**: Plays on specified target(s)
2. **Request origin area**: If no target specified, plays on Music Assistant player in the same area as the voice satellite
3. **Default player**: If no target specified and origin cannot be determined, uses the configured default player
4. **Error**: If no target can be determined, returns an error asking for target specification

### Examples

#### Auto-Play Examples

| Command | Behavior |
|---------|----------|
| "Play the track Bohemian Rhapsody by Queen in the bedroom" | Searches for track, plays first result |
| "Shuffle music by Taylor Swift" | Plays artist with shuffle enabled |
| "Play the album Rumours by Fleetwood Mac" | Searches and plays album |
| "Play BBC Radio 1" | Plays radio station |

#### Return Choices Examples

| Command | Behavior |
|---------|----------|
| "Find Pink Floyd music" | Returns artists, albums, tracks, playlists matching "Pink Floyd" |
| "Show me jazz albums" | Returns albums matching "jazz" |
| "What rock playlists do I have?" | Returns playlists matching "rock" |

#### Sample Choice Response

When returning choices, the response format is:

```
Found results for "jazz":

ARTISTS (3):
- Miles Davis
- John Coltrane
- Thelonious Monk

ALBUMS (3):
- Kind of Blue by Miles Davis
- A Love Supreme by John Coltrane
- Time Out by Dave Brubeck

TRACKS (3):
- Take Five by Dave Brubeck (from Time Out)
- So What by Miles Davis (from Kind of Blue)
- My Favorite Things by John Coltrane

PLAYLISTS (2):
- Evening Jazz
- Smooth Jazz Vibes

To play, call this script again with:
- media_id: the name of your chosen item
- media_type: the type (track/album/artist/playlist/radio)
- artist: the artist name (for tracks/albums, if shown)
- album: the album name (for tracks, if shown)
```

## Script Parameters

The script accepts the following parameters:

| Parameter | Required | Description |
|-----------|----------|-------------|
| `media_id` | Yes | The search query (song name, artist, album, playlist, or general terms) |
| `media_type` | No | Filter by type (track, album, artist, playlist, radio). Enables auto-play when specified. |
| `artist` | No | Artist name to refine search |
| `album` | No | Album name to refine search |
| `area` | No | Target area(s) for playback |
| `media_player` | No | Specific Music Assistant media player(s) |
| `shuffle` | No | Set to true to enable shuffle |
| `limit` | No | Maximum results per media type when returning choices |

## Troubleshooting

### "No Music Assistant instance selected"
Configure the Music Assistant Instance setting in the script configuration.

### "No results found"
The search query didn't match anything in your Music Assistant library. Try different terms or check that the music is properly imported.

### "Unable to find valid target"
No area or media player was specified and no default player is configured. Either specify a target in your voice command or set a default player.

### Script Not Available to LLM
Make sure to **expose the script to Assist** after creating it. The script won't be available as a tool until exposed.

## Differences from Separate Scripts

This composite script replaces both the play script (Option 3) and search script (Option 4):

| Feature | Old Play Script | Old Search Script | Composite Script |
|---------|----------------|-------------------|------------------|
| Direct playback | Yes | No | Yes (with media_type) |
| Return choices | No | Yes | Yes (without media_type) |
| Number of scripts needed | 1 for play, 1 for search | 1 for play, 1 for search | 1 total |
| LLM tool calls | 1-2 | 1-2 | 1-2 |

The composite script simplifies setup by requiring only one script instead of two.
