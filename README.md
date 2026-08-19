# Paddock 🏍️

**A group-ride app for motorcyclists.** Plan the route, track the pack live, and keep everything about the ride — the chat, the photos, the people — in one place instead of scattered across three WhatsApp groups.

Built solo: Android app, backend, and infrastructure.

---

## Download

**[⬇️ Download the latest APK](https://drive.google.com/file/d/1z9PNCJUBZMWw_EwMV_dZFxtIx7xjHUqf/view?usp=sharing)** · v1.1.0 · Android 7.0+

Android will warn you about installing an app from outside the Play Store — that's expected for a direct APK. Tap **More details → Install anyway**.

---

## Screenshots

<table>
  <tr>
    <td align="center" width="33%">
      <img src="screenshots/for-you.png" width="240" alt="For You feed"><br>
      <sub><b>Discover</b><br>Rides, routes and riders near you</sub>
    </td>
    <td align="center" width="33%">
      <img src="screenshots/ride-details.png" width="240" alt="Ride details"><br>
      <sub><b>Plan</b><br>Route, waypoints, riders, expected duration</sub>
    </td>
    <td align="center" width="33%">
      <img src="screenshots/live-tracking.png" width="240" alt="Live pack tracking"><br>
      <sub><b>Ride</b><br>Live pack tracking, regroups, SOS</sub>
    </td>
  </tr>
  <tr>
    <td align="center" width="33%">
      <img src="screenshots/ride-chat.png" width="240" alt="Ride chat"><br>
      <sub><b>Talk</b><br>Per-ride chat with photos, voice notes, replies</sub>
    </td>
    <td align="center" width="33%">
      <img src="screenshots/groups.png" width="240" alt="Groups"><br>
      <sub><b>Clubs</b><br>Riding groups with their own chat</sub>
    </td>
    <td align="center" width="33%">
      <img src="screenshots/profile.png" width="240" alt="Rider profile"><br>
      <sub><b>Profile</b><br>Your garage, your rides, your posts</sub>
    </td>
  </tr>
</table>

---

## What it does

**Ride coordination — the core of it**
- Plan a ride with a real route: start, waypoints, destination, round trip with a layover, and an expected duration pulled from the actual routing
- Public, private (host approves) or invite-only, with join requests, direct invites and a rider cap
- **Live pack tracking** — see where everyone is on the map while the ride is on, so nobody gets lost at a junction
- **Regroup calls** — any rider can drop a "everyone stop here" pin for fuel, food or a break
- **Rider statuses** — "I've stopped to refuel, keep going, I'll catch up", so the pack never has to guess why someone dropped off the back
- **SOS** — one button that puts your location in front of everyone on the ride
- **Offline maps** — download a ride's route tiles before you leave, for the stretches with no signal
- Solo rides, a recap with photos afterwards, and host ratings

**Chat that belongs to the ride**
- A chat per ride and per club, with replies, @mentions, read receipts and per-message info
- Photos and videos (multi-select, with an HD toggle), voice notes, and one-tap location sharing
- A media gallery per chat, unsend, host-only mode, and pinned announcements
- Works offline from a local cache, and survives a dropped connection without duplicating what you sent

**The community around it**
- Posts with photo carousels and reels, comments, likes, saves and hashtags
- Follow riders, add crew, join clubs, RSVP to events
- Rider-submitted destinations — cafés, viewpoints and meet spots that end up as routes other people ride
- Brand partnerships and paid-collab disclosure for riders who create content
- Reporting, blocking and an admin moderation queue

---

## Built with

**Android app** — Expo SDK 52 · React Native 0.76 · TypeScript · expo-router · TanStack Query · Zustand · react-native-maps · Reanimated + Gesture Handler · SQLite · STOMP over WebSocket

**Backend** — Spring Boot 3.5 · Java 23 · PostgreSQL · JWT auth · WebSocket/STOMP for live location and chat · Cloudinary for media · Expo Push

**Infrastructure** — Render (Singapore) · Neon Postgres (Singapore) · 550+ backend tests

---

## A few engineering details

Things that took more thought than the feature list suggests:

**Chat works with no signal.** Messages are cached on-device in SQLite, so opening a chat paints instantly from disk while the network catches up. The schema is versioned with `PRAGMA user_version` and *rebuilt* rather than migrated on a version bump — it's a cache, so throwing it away is always correct and never leaves a half-migrated database on someone's phone.

**Sending a message is idempotent.** Every message carries a client-generated id, so a retry after a dropped connection can't produce a duplicate — the server recognises the id and returns the message it already stored.

**Latency was measured, not guessed.** The app has a built-in panel reporting socket transport, connect time, ping and cache-paint time, because a release build on a real phone is the only honest benchmark. Those numbers are what prompted moving the API and database from Oregon to Singapore — first byte went from ~307ms to ~123ms.

**The WebSocket knows when to sleep.** It disconnects when the app is backgrounded so the server stops treating the rider as present and resumes sending push notifications instead — *unless* a ride is live, when the map has to keep updating from the background.

**Location is one position, not a history.** A rider's position is broadcast to the pack live, and exactly one last-known position is stored per rider per ride — overwritten on every update, wiped when the ride ends. That's enough to answer "where were they last, and how long ago", which is what a safety feature needs; a trail of everywhere someone has been is a different product with different obligations, so the app doesn't keep one. Writes are also throttled well below the broadcast rate, because the map wants every update and the database doesn't.

---

## Notes

Source is private — happy to walk through the code or the architecture on request.

This is a personal project, still pre-release. If something breaks, I'd genuinely like to hear about it.
