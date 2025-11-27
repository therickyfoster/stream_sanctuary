# 📹 Multi-Source Independent Viewing - Visual Architecture

## System Flow Diagram

```
╔═══════════════════════════════════════════════════════════════════════════╗
║                          STREAMER'S BROWSER                               ║
╠═══════════════════════════════════════════════════════════════════════════╣
║                                                                           ║
║  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐         ║
║  │  🖥️ Main Screen │  │  📹 Webcam      │  │  🖥️ Secondary   │         ║
║  │  1920x1080@60   │  │  1280x720@30    │  │  1920x1080@30   │         ║
║  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘         ║
║           │                    │                     │                   ║
║           │   MediaStream      │   MediaStream       │   MediaStream    ║
║           │                    │                     │                   ║
║  ┌────────▼────────────────────▼─────────────────────▼────────┐         ║
║  │         MultiSourceBroadcaster (JavaScript)                │         ║
║  │  • Manages multiple MediaStream objects                    │         ║
║  │  • Creates StreamingEngine per source                      │         ║
║  │  • Modifies stream keys: key_SourceName                    │         ║
║  └────────┬────────────────────┬─────────────────────┬────────┘         ║
║           │                    │                     │                   ║
╚═══════════╪════════════════════╪═════════════════════╪═══════════════════╝
            │                    │                     │
            │ RTMP/WHIP/WS       │ RTMP/WHIP/WS       │ RTMP/WHIP/WS
            │ key_Main_Screen    │ key_Webcam         │ key_Secondary
            │                    │                     │
            ▼                    ▼                     ▼
╔═══════════════════════════════════════════════════════════════════════════╗
║                        STREAMING SERVER / CDN                             ║
╠═══════════════════════════════════════════════════════════════════════════╣
║                                                                           ║
║  ┌─────────────────────────────────────────────────────────────────────┐ ║
║  │  Stream Router (Nginx RTMP / Node-Media-Server / WHIP)             │ ║
║  │                                                                     │ ║
║  │  Incoming:                     Routing:                            │ ║
║  │  • key_Main_Screen      →      /hls/key/Main_Screen/index.m3u8   │ ║
║  │  • key_Webcam          →      /hls/key/Webcam/index.m3u8         │ ║
║  │  • key_Secondary       →      /hls/key/Secondary/index.m3u8      │ ║
║  └─────────────────────────────────────────────────────────────────────┘ ║
║                                                                           ║
║  ┌─────────────────────────────────────────────────────────────────────┐ ║
║  │  Source Metadata API                                                │ ║
║  │  GET /api/sources/streamer123                                       │ ║
║  │  Returns: [{id, label, url, resolution, fps}, ...]                 │ ║
║  └─────────────────────────────────────────────────────────────────────┘ ║
║                                                                           ║
╚═══════════╤════════════════════╤═════════════════════╤═══════════════════╝
            │                    │                     │
            │ HLS/DASH          │ HLS/DASH           │ HLS/DASH
            │                    │                     │
            ▼                    ▼                     ▼
╔═══════════════════════════════════════════════════════════════════════════╗
║                          VIEWER'S BROWSER                                 ║
╠═══════════════════════════════════════════════════════════════════════════╣
║                                                                           ║
║  ┌─────────────────────────────────────────────────────────────────────┐ ║
║  │  ViewerSourceSelector (JavaScript)                                  │ ║
║  │  • Fetches available sources from API                               │ ║
║  │  • Displays source selection buttons                                │ ║
║  │  • Manages video element srcObject switching                        │ ║
║  └───────────────────────────┬─────────────────────────────────────────┘ ║
║                              │                                           ║
║  ┌───────────────────────────▼─────────────────────────────────────────┐ ║
║  │  Source Selector UI                                                 │ ║
║  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐               │ ║
║  │  │ 🖥️ Main     │ │ 📹 Webcam    │ │ 🖥️ Secondary │               │ ║
║  │  │   Screen    │ │    (Active)   │ │    Screen    │               │ ║
║  │  │  [SELECT]   │ │   [▶LIVE]    │ │   [SELECT]   │               │ ║
║  │  └──────────────┘ └──────────────┘ └──────────────┘               │ ║
║  └───────────────────────────┬─────────────────────────────────────────┘ ║
║                              │                                           ║
║                              │ User clicks "Main Screen"                 ║
║                              │                                           ║
║  ┌───────────────────────────▼─────────────────────────────────────────┐ ║
║  │  <video> Element                                                    │ ║
║  │  ┌───────────────────────────────────────────────────────────────┐ │ ║
║  │  │                                                               │ │ ║
║  │  │                   🖥️ Main Screen Feed                        │ │ ║
║  │  │              (Now showing gameplay/desktop)                   │ │ ║
║  │  │                                                               │ │ ║
║  │  │                  [Fullscreen] [⚙️Settings]                    │ │ ║
║  │  └───────────────────────────────────────────────────────────────┘ │ ║
║  └─────────────────────────────────────────────────────────────────────┘ ║
║                                                                           ║
║  Note: Switching sources is INSTANT - new stream loads immediately       ║
║        Viewer's bandwidth usage = 1 stream at a time (same as normal)    ║
║                                                                           ║
╚═══════════════════════════════════════════════════════════════════════════╝
```

---

## Sequence Diagram: User Switches Source

```
Viewer          Browser UI       JavaScript          Streaming Server
  │                 │                │                      │
  │   Click         │                │                      │
  │ "Face Cam"      │                │                      │
  │─────────────────>                │                      │
  │                 │   switchToSource('webcam')            │
  │                 │───────────────>│                      │
  │                 │                │  GET /hls/.../webcam/index.m3u8
  │                 │                │─────────────────────>│
  │                 │                │                      │
  │                 │                │   200 OK (playlist)  │
  │                 │                │<─────────────────────│
  │                 │                │                      │
  │                 │                │  GET segment_001.ts  │
  │                 │                │─────────────────────>│
  │                 │                │                      │
  │                 │                │   200 OK (video)     │
  │                 │                │<─────────────────────│
  │                 │                │                      │
  │                 │   Update video.srcObject              │
  │                 │<───────────────│                      │
  │                 │                │                      │
  │   Video shows   │                │                      │
  │   Face Cam      │                │                      │
  │<─────────────────                │                      │
  │                 │                │                      │
  │  (Instant switch - typically < 500ms)                   │
  │                 │                │                      │
```

---

## Data Flow: Multi-Source Broadcast

```
TIME: T+0 seconds (Stream Start)
════════════════════════════════════════════

Streamer initiates broadcast:
├─ Source 1: Main Screen → key_Main_Screen → Server → CDN → Available
├─ Source 2: Webcam      → key_Webcam      → Server → CDN → Available
└─ Source 3: Secondary   → key_Secondary   → Server → CDN → Available

All three streams are LIVE simultaneously


TIME: T+10 seconds (Viewer Joins)
════════════════════════════════════════════

Viewer opens stream:
1. Browser fetches: GET /api/sources/streamer123
2. Server responds: [
     {id: 'main_screen', label: '🖥️ Main Screen', url: '...'},
     {id: 'webcam', label: '📹 Webcam', url: '...'},
     {id: 'secondary', label: '🖥️ Secondary', url: '...'}
   ]
3. UI renders 3 buttons
4. Default source (main_screen) loads automatically
5. Viewer sees: Main Screen (gameplay)


TIME: T+120 seconds (Viewer Switches)
════════════════════════════════════════════

Viewer clicks "📹 Webcam" button:
1. JavaScript calls: switchToSource('webcam')
2. Stop current stream: main_screen
3. Load new stream: webcam
4. Update video element srcObject
5. Highlight active button
6. Viewer now sees: Webcam (face cam)

Switch latency: ~200-500ms (instant for user)


TIME: T+180 seconds (Another Switch)
════════════════════════════════════════════

Viewer clicks "🖥️ Secondary" button:
1. Same process as above
2. Now viewing: Secondary Screen
3. Total switches: 2
4. Bandwidth used: Same as single stream
5. User experience: Seamless


TIME: T+300 seconds (Stream End)
════════════════════════════════════════════

Streamer ends broadcast:
├─ All 3 sources stop simultaneously
├─ Viewers see "Stream Ended" message
└─ Analytics logged: 
    • Main Screen: 150 seconds watched
    • Webcam: 60 seconds watched
    • Secondary: 90 seconds watched
```

---

## Component Hierarchy

```
Stream Sanctuary Application
│
├─ MultiSourceBroadcaster (Streamer Side)
│  │
│  ├─ Source Management
│  │  ├─ addSource(config) → MediaStream
│  │  ├─ removeSource(sourceId)
│  │  └─ getSources() → Array<SourceConfig>
│  │
│  ├─ Broadcast Management
│  │  ├─ startBroadcast(sourceId, destination)
│  │  ├─ stopBroadcast(sourceId)
│  │  └─ stopAllBroadcasts()
│  │
│  └─ StreamingEngine (per source)
│     ├─ WebRTC-WHIP Connection
│     ├─ RTMP Connection
│     ├─ WebSocket Connection
│     └─ HLS Upload
│
├─ ViewerSourceSelector (Viewer Side)
│  │
│  ├─ Source Discovery
│  │  └─ fetchAvailableSources(streamerId)
│  │
│  ├─ Source Switching
│  │  ├─ switchToSource(sourceId, videoElement)
│  │  ├─ connectToSource(url, source)
│  │  └─ stopCurrentSource()
│  │
│  └─ UI Management
│     ├─ renderSourceSelector(sources)
│     ├─ updateActiveButton(sourceId)
│     └─ trackSourceSwitch(sourceId)
│
└─ UI Components
   │
   ├─ Source Manager (Streamer)
   │  ├─ Add Source Button
   │  ├─ Source Cards (for each source)
   │  │  ├─ Source Info Display
   │  │  ├─ Start/Stop Broadcast Buttons
   │  │  ├─ Preview Button
   │  │  └─ Remove Button
   │  └─ Batch Controls (Start All / Stop All)
   │
   └─ Source Selector (Viewer)
      ├─ Source Buttons (dynamically generated)
      ├─ Active Indicator (highlights current source)
      └─ Video Player (displays selected source)
```

---

## State Management

```javascript
// Application State Structure

APP_STATE = {
    // ... existing properties ...
    
    sources: {
        // Available media sources
        available: [
            {
                id: 'source_abc123',
                type: 'screen',
                label: '🖥️ Main Screen',
                stream: MediaStream,
                broadcasting: true,
                config: {
                    resolution: { width: 1920, height: 1080 },
                    framerate: 60,
                    captureAudio: true
                }
            },
            {
                id: 'source_def456',
                type: 'webcam',
                label: '📹 Face Cam',
                stream: MediaStream,
                broadcasting: true,
                config: { ... }
            }
        ],
        
        // Currently broadcasting sources
        active: ['source_abc123', 'source_def456'],
        
        // Viewer's selected source
        viewerSelected: 'source_abc123'
    }
}
```

---

## Network Topology

```
Internet Backbone
       │
       │
       ▼
┌─────────────────┐
│   CDN Edge      │◄──── Low latency distribution
│   (Cloudflare)  │      Cached HLS segments
└────────┬────────┘      Geographically distributed
         │
         │
         ▼
┌─────────────────┐
│  Origin Server  │◄──── Transcoding & packaging
│  (Your Server)  │      Multiple source routing
└────────┬────────┘      Authentication & analytics
         │
         │
         ▼
┌─────────────────┐
│ RTMP/WHIP Ingest│◄──── Receives multiple streams
│                 │      Routes by stream key suffix
└─────────────────┘
         ▲
         │
         │ Upload: 3 simultaneous streams
         │ Total: ~12 Mbps
         │
┌────────┴────────┐
│   Streamer's    │
│     Browser     │
└─────────────────┘
```

---

## Example Production Setup

```
Company: GameStream Pro
Streamer: ProGamer123
Stream: Friday Night Fortnite

Sources Configured:
├─ 🖥️ Main Screen (Fortnite gameplay, 1080p60, 6 Mbps)
├─ 📹 Face Cam (Reaction, 720p30, 2 Mbps)
├─ 🖥️ Secondary (Discord/Chat, 1080p30, 3 Mbps)
└─ 🎤 Microphone (Audio only, 128 kbps)

Viewer Statistics (1000 concurrent):
├─ Main Screen: 800 viewers (80%)
├─ Face Cam: 150 viewers (15%)
├─ Secondary: 50 viewers (5%)

Server Load:
├─ Ingress: 12 Mbps × 1 streamer = 12 Mbps
├─ Egress: 6 Mbps × 800 + 2 Mbps × 150 + 3 Mbps × 50
│           = 4800 + 300 + 150 = 5250 Mbps (5.25 Gbps)
└─ CDN handles majority of egress bandwidth

Cost Estimate:
├─ Origin Server: $50/month (Digital Ocean)
├─ CDN Bandwidth: $0.05/GB × 2TB = $100/month
└─ Total: ~$150/month for 1000 concurrent viewers
```

This visual architecture makes it crystal clear how the system works! 🎨📊