# Apple Music ALAC Downloader
Original script by Sorrow. Modified by me to include some fixes and improvements.

## How to use (CLI Mode)
1. Create a virtual device on Android Studio with a image that doesn't have Google APIs.
2. Install this version of Apple Music: https://www.apkmirror.com/apk/apple/apple-music/apple-music-3-6-0-beta-release/apple-music-3-6-0-beta-4-android-apk-download/. You will also need SAI to install it: https://f-droid.org/pt_BR/packages/com.aefyr.sai.fdroid/.
3. Launch Apple Music and sign in to your account. Subscription required.
4. Port forward 10020 TCP: `adb forward tcp:10020 tcp:10020`.
5. Start frida server.
6. Start the frida agent: `frida -U -l agent.js -f com.apple.android.music`.
7. Start downloading some albums: `go run main.go https://music.apple.com/us/album/whenever-you-need-somebody-2022-remaster/1624945511`.

## API Mode

The application can also run as an HTTP API server to push Apple Music audio.

### Starting the API Server

```bash
go run main.go -api -port 8080
```

Options:
- `-api`: Run in API server mode
- `-port`: API server port (default: 8080)

### API Endpoints

#### Health Check
```
GET /health
```
Returns server health status.

Response:
```json
{"success": true, "data": "OK"}
```

#### Get Album Metadata
```
GET /api/album/{storefront}/{albumId}
```

Parameters:
- `storefront`: Country code (e.g., `us`, `gb`, `jp`)
- `albumId`: Apple Music album ID

Example:
```bash
curl http://localhost:8080/api/album/us/1624945511
```

Response:
```json
{
  "success": true,
  "data": {
    "data": [{
      "id": "1624945511",
      "attributes": {
        "artistName": "Rick Astley",
        "name": "Whenever You Need Somebody (2022 Remaster)",
        ...
      },
      "relationships": {
        "tracks": {...}
      }
    }]
  }
}
```

#### Get Track Info / Stream Track
```
GET /api/track/{storefront}/{trackId}
GET /api/track/{storefront}/{trackId}?info=true
```

Parameters:
- `storefront`: Country code (e.g., `us`, `gb`, `jp`)
- `trackId`: Apple Music track ID
- `info=true`: (Optional) Return track info as JSON instead of streaming audio

Example (get track info):
```bash
curl "http://localhost:8080/api/track/us/1624945518?info=true"
```

Response:
```json
{
  "success": true,
  "data": {
    "trackId": "1624945518",
    "trackName": "Never Gonna Give You Up",
    "artistName": "Rick Astley",
    "albumName": "Whenever You Need Somebody (2022 Remaster)",
    "durationMs": 213573,
    "bitDepth": "24",
    "sampleRate": "44100",
    "available": true
  }
}
```

Example (stream track):
```bash
curl -o track.m4a http://localhost:8080/api/track/us/1624945518
```

This will download the track as an M4A file with ALAC codec.

### Requirements for API Mode

The API server still requires the decryption service to be running for streaming tracks:
1. Complete steps 1-6 from the CLI Mode setup
2. The frida agent must be running on the Android emulator
3. Port 10020 must be forwarded (`adb forward tcp:10020 tcp:10020`)
