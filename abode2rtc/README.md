# Home Assistant Add-on: Abode Camera Streaming

_Live video from Abode cameras, including the Abode Cam 2, in Home Assistant._

![Supports aarch64 Architecture][aarch64-shield]
![Supports amd64 Architecture][amd64-shield]
![Does not support armhf Architecture][armhf-shield]
![Supports armv7 Architecture][armv7-shield]
![Supports i386 Architecture][i386-shield]

![Sample Lovelace dashboard with camera live view](assets/lovelace.png)

## Introduction

The core [Abode integration][abode-int] only exposes the last snapshot for Abode
cameras, and the Abode Cam 2 has no officially supported local stream (RTSP or
ONVIF). This add-on fills that gap: it requests each camera's Live View stream
from Abode and serves it through [go2rtc], so you can watch it live on a Home
Assistant dashboard with the WebRTC Camera card.

### Credits

This add-on was written by Todd Radel ([@tradel](https://github.com/tradel)) in
[tradel/hassio-addons](https://github.com/tradel/hassio-addons). Streaming is
handled by [go2rtc] by Alex X ([@AlexxIT](https://github.com/AlexxIT)).

### History

The add-on was developed in the open in the Home Assistant Community thread
[Abode security cameras][thread], which Todd started on 2023-10-27 with the
first release. Highlights from that thread:

- **v1.2.2 (2023-11):** streams stopped timing out after a few minutes once the
  add-on fetched the KVS endpoint from Abode right before each stream started.
  This fork keeps that design.
- **Abode Cam 2:** users reported working live view on Abode Cam 2 cameras
  (2024-01, 2024-03). The camera built into the Abode iota base was reported
  not to work.
- **Port conflicts:** the add-on embeds its own go2rtc, so it can't run
  alongside the go2rtc add-on or anything else on ports 1984/8554.
- **2FA:** accounts with two-factor authentication weren't supported.

The per-stream sign-in emails fixed here were not raised in that thread. They
are tracked in [tradel/hassio-addons#16][issue16].

### What this fork changes

- **One Abode login per add-on start.** The original signed in to Abode on every
  stream start, as a new device each time. Abode sent a sign-in email for each
  one and eventually rate-limited the account. This version keeps one session
  alive, reuses it for every stream, and logs in from a persistent device ID.
  See [tradel/hassio-addons#16][issue16].
- **Built on your device.** Supervisor builds the add-on from this repository
  instead of pulling a prebuilt image.

Tested with three Abode Cam 2 cameras on Home Assistant OS (aarch64) with
WebRTC Camera 3.6.1.

**Note:** Abode has not published an API for its cameras. This add-on uses the
same internal API as the Abode web app, so it may stop working if Abode changes
that API.

## Installation

### Prerequisites

1. Set up the [Abode integration][abode-int]. You should see a `camera.` entity
   for each Abode camera.
2. Install the [WebRTC Camera][webrtc-repo] custom integration (from [HACS] or
   Home Assistant's integration marketplace) and restart Home Assistant.

### Install the add-on

1. Go to **Settings** > **Apps** and open the app store.
2. Open the **⋮** menu in the top right and select **Repositories**.
3. Add `https://github.com/duvallg/ha-abode-cam2`.
4. Refresh the page, then find **Abode Camera Streaming** and install it.

If the original add-on from `tradel/hassio-addons` is installed, uninstall it
or at least stop it first. Both bind host ports 1984, 8554 and 8555, and running
the original brings back the per-stream sign-in emails.

## Configuration

Open the add-on's **Configuration** tab, enter your Abode username (the email
address you sign in to Abode with) and password, and click **Save**. Then start
the add-on from the **Info** tab.

![Add-on configuration screen](assets/config.png)

Expect one Abode sign-in email each time the add-on starts (and the first start
registers a new device). Viewing cameras doesn't sign in again.

Accounts with multi-factor authentication enabled have been reported not to
work (see [tradel/hassio-addons#12][issue12]). If Abode refuses the login, the
add-on log shows the HTTP status and Abode's response.

### Connect WebRTC Camera

Once the add-on is running, go to **Settings** > **Devices & services**, add
**WebRTC Camera**, and set the go2rtc URL to `http://localhost:1984/`. Setup
fails if the add-on isn't running yet.

## Adding a camera to a dashboard

The add-on creates one go2rtc stream per Abode camera, named after the camera's
entity ID without the `camera.` prefix. For example, `camera.garage_camera`
becomes `garage_camera`.

Add a manual card with:

```yaml
type: custom:webrtc-camera
url: garage_camera
muted: true
```

See the [WebRTC Camera documentation][webrtc] for more options.

## How it works

Abode cameras don't expose a supported local stream. They send video to
[Amazon Kinesis Video Streams][kvs]. When you open Live View, the
[Abode web app][webapp] calls `POST /integrations/v1/camera/{uuid}/kvs/stream`
to get a short-lived WebRTC signaling endpoint and ICE servers:

```json
{
    "channelEndpoint": "wss://v-50e9eb79.kinesisvideo.us-west-2.amazonaws.com/...",
    "iceServers": [
        {"urls": ["stun:stun.kinesisvideo.us-west-2.amazonaws.com:443"]},
        {"credential": "...", "username": "...", "urls": ["turns:...:443?transport=tcp"]}
    ],
    "type": "KVS"
}
```

[go2rtc] can play Kinesis WebRTC streams directly. On startup, the add-on:

1. Signs in to Abode once, saves the session, and refreshes its access token in
   the background.
2. Reads the Abode camera entities from Home Assistant.
3. Writes a `go2rtc.yaml` with one stream per camera using go2rtc's [echo]
   source, so a camera's endpoint is fetched only when its stream starts. The
   endpoints expire within minutes, so they can't be generated up front.
4. Starts go2rtc.

Each stream entry runs `stream.py` when a viewer connects:

```yaml
streams:
  garage_camera: 'echo:python3 /app/stream.py /tmp/abode.json XF:b0c5ca3e8a79'
```

`stream.py` requests a fresh endpoint with the saved session and returns it to
go2rtc. It signs in again only if Abode rejects the saved session, and it skips
offline cameras without signing in.

## Limitations

- Depends on Abode's cloud. If Abode or your internet connection is down, there
  is no stream.
- The AppArmor profile runs in complain mode.
- armhf isn't supported because go2rtc doesn't support it.

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[armhf-shield]: https://img.shields.io/badge/armhf-no-red.svg
[armv7-shield]: https://img.shields.io/badge/armv7-yes-green.svg
[i386-shield]: https://img.shields.io/badge/i386-yes-green.svg
[abode-int]: https://www.home-assistant.io/integrations/abode/
[hacs]: https://hacs.xyz/
[go2rtc]: https://github.com/AlexxIT/go2rtc
[webrtc-repo]: https://github.com/AlexxIT/WebRTC
[webrtc]: https://github.com/AlexxIT/WebRTC/blob/master/README.md#custom-card
[kvs]: https://aws.amazon.com/kinesis/video-streams/
[webapp]: https://my.goabode.com/#/app/live-video
[echo]: https://github.com/AlexxIT/go2rtc/wiki/Source-Echo-examples
[issue12]: https://github.com/tradel/hassio-addons/issues/12
[issue16]: https://github.com/tradel/hassio-addons/issues/16
[thread]: https://community.home-assistant.io/t/abode-security-cameras/632522
