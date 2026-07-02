# Mineradio Function Areas

This note maps the current app by runtime responsibility. It is meant to help future fixes land in the shared path instead of patching one screen at a time.

## Desktop Shell

- `desktop/main.js` starts the Electron window, owns native window behavior, and launches the local app service.
- `desktop/preload.js` exposes the narrow desktop bridge used by the front end.
- The packaged Windows app runs from `resources/app`, so changes must be made in the app directory that the opened `Mineradio.exe` actually loads.

## Local API

- `server.js` is the local HTTP API and the main integration point for music providers.
- It owns NetEase QR/cookie login, QQ cookie login, search, home discovery, song URL resolution, playlists, podcast helpers, weather radio, update checks, and patch/download helpers.
- NetEase auth state is held in `.cookie`; QQ auth state is held in `.qq-cookie`.

## Main Front End

- `public/index.html` contains the primary UI, CSS, visual stages, search, home discovery, queue, playback controls, login modals, playlist shelf, lyrics, and playback failure handling.
- Search results and home recommendations both enter the same queue/playback path before requesting a source URL from the local API.
- Playback failure categories from `server.js` drive front-end notices, fallback behavior, and whether a login modal is opened.

## Stored User State

- Provider cookies live beside `server.js` as `.cookie` and `.qq-cookie`; they are local runtime secrets and must not be committed.
- Visual preferences, user archives, search history, and layout choices are mostly browser `localStorage`.
- Packaged defaults such as `public/default-user-fx-archive.json` seed first-run visual state without overwriting existing user choices.

## Playback Login Flow

Search result or home recommendation -> front-end queue -> `/api/song/url` -> `getLoginInfo()` -> NetEase `song_url_v1` or `song_url` with `userCookie`.

`getLoginInfo()` should distinguish a missing/invalid cookie from a temporary profile refresh failure. Playback can still use a valid cookie without a fresh profile, while account features that need `userId` must keep requiring a full login profile.
