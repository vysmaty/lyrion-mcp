# Lyrion MCP Server

An [MCP](https://modelcontextprotocol.io/) server that lets LLMs control a
[Lyrion Music Server](https://lyrion.org/) (LMS / Squeezebox) instance.

## Features

- **12 MCP tools** with explicit search tools for tracks, albums, and artists
- Play by URL, track ID, search, collection ID, TIDAL app reference, or Spotify Artist Radio
- Search across local library + Spotify (via Spotty) + TIDAL with explicit track/album/artist filters
- Full playback control: pause, stop, seek, power on/off
- Playlist management: add, insert, delete, clear, move, jump, save
- Player settings: volume, shuffle, repeat, mute
- Player sync/unsync
- Library browsing: genres, artists, albums, titles, years, playlists
- Raw LMS CLI passthrough for advanced commands

## Quick Start

```bash
pip install -r requirements.txt
LMS_HOST=your-lms-host python main.py
```

## Configuration

Set environment variables (see `.env.example`):

| Variable | Default | Description |
|----------|---------|-------------|
| `LMS_HOST` | `localhost` | Lyrion server host |
| `LMS_PORT` | `9000` | Lyrion server port |
| `LMS_USERNAME` | (none) | LMS username (if auth enabled) |
| `LMS_PASSWORD` | (none) | LMS password (if auth enabled) |
| `LMS_HTTPS` | off | Set `1` if LMS uses HTTPS |
| `MCP_TRANSPORT` | `stdio` | `stdio`, `sse`, or `streamable-http` |
| `MCP_HOST` | `0.0.0.0` | Bind address (HTTP transports) |
| `MCP_PORT` | `8000` | Port (HTTP transports) |

## Docker

```bash
docker build -t lyrion-mcp .
docker run -d -p 8000:8000 -e LMS_HOST=your-lms-host lyrion-mcp
```

The server runs on `http://localhost:8000` using the `streamable-http`
transport. Point your MCP client at that URL.

### Docker Compose

```yaml
services:
  lyrion-mcp:
    image: ghcr.io/vysmaty/lyrion-mcp:latest
    restart: unless-stopped
    ports:
      - "8000:8000"
    environment:
      LMS_HOST: your-lms-host
      LMS_PORT: "9000"
      LMS_HTTPS: "0"
      # LMS_USERNAME: your-username
      # LMS_PASSWORD: your-password
```

For a local checkout, replace the `image` line with:

```yaml
    build: .
```

### GitHub Container Registry

Publishing a GitHub release builds and pushes a multi-architecture image to
GitHub Container Registry:

```bash
docker pull ghcr.io/vysmaty/lyrion-mcp:latest
docker run -d -p 8000:8000 -e LMS_HOST=your-lms-host ghcr.io/vysmaty/lyrion-mcp:latest
```

Release images are tagged as `latest`, the release version (for example
`v1.0.0`), the major/minor version, and the commit SHA. The workflow can also
be started manually from GitHub Actions, where it publishes an `edge` image and
an optional custom tag. The workflow must be present on the repository's
default branch before GitHub can run it for new releases.

## Tools

| Tool | Description |
|------|-------------|
| `get_status` | System topology (all players) or now-playing (with player_id) |
| `play_media` | Play by URL, track_id, search (defaults to Artist Radio), collection ID, or TIDAL app reference |
| `search_media` | Search local library + Spotify + TIDAL with `media_type` filter (`any`, `track`, `album`, `artist`) |
| `search_tracks` | Search only playable tracks/songs; use before adding individual songs to a playlist |
| `search_albums` | Search only albums; TIDAL albums return playable `lms://tidal/...` references |
| `search_artists` | Search only artists/bands; TIDAL artists return playable `lms://tidal/...` references |
| `control_playback` | Pause, stop, play, seek, power on/off |
| `manage_playlist` | Add, insert, delete, clear, move, jump, save |
| `set_player` | Volume, shuffle, repeat, mute |
| `sync_players` | Sync or unsync players |
| `browse_library` | Browse genres/artists/albums/titles/years/playlists; searched artists/albums include TIDAL matches |
| `query_lms` | Raw LMS CLI passthrough |

## Development

```bash
pip install -r requirements.txt
python -m pytest -q          # 110 tests
python -m mypy client.py main.py
python -m pyflakes client.py main.py tests/
```

## Requirements

- Python 3.12+
- A running Lyrion Music Server (9.x)
- [Spotty plugin](https://github.com/michaelherger/Spotty-Plugin) (optional, for Spotify)
- [TIDAL plugin](https://github.com/michaelherger/lms-plugin-tidal) (optional, for TIDAL)
