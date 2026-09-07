# Paddock 🏍️

**A group-ride app for motorcyclists.** Plan the route, track the pack live, and keep everything about the ride — the chat, the photos, the people — in one place instead of scattered across three WhatsApp groups.

Built solo: Android app, backend, and infrastructure.

---

## Download

**[⬇️ Download the latest APK](https://drive.google.com/file/d/1XXdzmF9RnEqpRPn15KNdvo9_tSGsDJfP/view?usp=sharing)** · v1.1.0 · Android 7.0+

Android will warn you about installing an app from outside the Play Store — that's expected for a direct APK. Tap **More details → Install anyway**.

---

## What it does

**Ride coordination — the core of it**
- Plan a ride with a real route: start, waypoints, destination, round trip with a layover, and an expected duration pulled from the actual routing
- Public, private (host approves) or invite-only, with join requests, direct invites and a rider cap
- The host can re-pick the route, the start or the destination after publishing — everyone who joined is told what changed
- **Live pack tracking** — see where everyone is on the map while the ride is on, so nobody gets lost at a junction
- **Regroup calls** — any rider can drop an "everyone stop here" pin for fuel, food or a break
- **Rider statuses** — "I've stopped to refuel, keep going, I'll catch up", so the pack never has to guess why someone dropped off the back
- **SOS** — one button that puts your location in front of everyone on the ride
- **Offline maps** — download a ride's route tiles before you leave, for the stretches with no signal
- Solo rides, a recap with photos afterwards, and host ratings

**The map**
- Rides, and rider-submitted places by category — cafés, viewpoints, bike shops, rest stops
- **Petrol pumps**, from OpenStreetMap, filtered down to stations that actually sell petrol rather than CNG-only outlets. Coverage extends itself to new cities as riders reach them
- Your own upcoming rides, marked apart from everyone else's
- Places you have actually ridden to, marked as visited
- Highways labelled, and a road hierarchy you can read at speed

**Chat that belongs to the ride**
- A chat per ride and per club, with replies, @mentions, read receipts and per-message info
- Photos and videos (multi-select, with an HD toggle), voice notes, and one-tap location sharing
- A media gallery per chat, unsend, host-only mode, and pinned announcements
- Search your own messages on the device, the way WhatsApp does it
- Works offline from a local cache, and survives a dropped connection without duplicating what you sent

**The community around it**
- A home feed of the riders you follow, with suggestions filling in behind it while your follow list is still short
- A discovery tab: search for riders, and a grid of everything posted
- Posts with photo carousels and reels, likes, saves and hashtags
- **Comments**, one level of replies deep, where you can tag another rider and they get told
- Follow riders, add ride buddies, join clubs, RSVP to events
- Rider-submitted destinations — cafés, viewpoints and meet spots that end up as routes other people ride
- Achievements that track distance ridden, and a notifications history rather than a banner you can miss
- Reporting, blocking and an admin moderation queue

**For brands**
- Verified brand accounts, applied for through a partners page and approved by hand
- Brand-hosted events — track days, riding experiences — carried at the top of the discovery tab with RSVPs
- Paid-collab offers between brands and riders, with disclosure on any post that carries one

---

## Built with

**Android app** — Expo SDK 52 · React Native 0.76 · TypeScript · expo-router · TanStack Query · Zustand · react-native-maps · Reanimated + Gesture Handler · SQLite · STOMP over WebSocket

**Backend** — Spring Boot 3.5 · Java 23 · PostgreSQL · JWT auth · WebSocket/STOMP for live location and chat · Cloudinary for media · Expo Push

**Infrastructure** — Render (Singapore) · Neon Postgres (Singapore) · 740 backend tests

---

## A few engineering details

Things that took more thought than the feature list suggests:

**Chat works with no signal.** Messages are cached on-device in SQLite, so opening a chat paints instantly from disk while the network catches up. The schema is versioned with `PRAGMA user_version` and *rebuilt* rather than migrated on a version bump — it's a cache, so throwing it away is always correct and never leaves a half-migrated database on someone's phone.

**Sending a message is idempotent.** Every message carries a client-generated id, so a retry after a dropped connection can't produce a duplicate — the server recognises the id and returns the message it already stored.

**Latency was measured, not guessed.** The app has a built-in panel reporting socket transport, connect time, ping and cache-paint time, because a release build on a real phone is the only honest benchmark. Those numbers are what prompted moving the API and database from Oregon to Singapore — first byte went from ~307ms to ~123ms.

**The WebSocket knows when to sleep.** It disconnects when the app is backgrounded so the server stops treating the rider as present and resumes sending push notifications instead — *unless* a ride is live, when the map has to keep updating from the background.

**Location is one position, not a history.** A rider's position is broadcast to the pack live, and exactly one last-known position is stored per rider per ride — overwritten on every update, wiped when the ride ends. That's enough to answer "where were they last, and how long ago", which is what a safety feature needs; a trail of everywhere someone has been is a different product with different obligations, so the app doesn't keep one. Writes are also throttled well below the broadcast rate, because the map wants every update and the database doesn't.

**The database sleeps when nobody is riding.** Seven background jobs poll for rides that need attention — start reminders, auto-completion, pack checks. On serverless Postgres, billed per awake minute, that meant the database never slept and a month's quota went in under three weeks. All seven now sit behind one cached "is any ride actually pending?" check, invalidated by an entity listener whenever a ride is written, so the pollers can't drift out of step with it. Idle costs nothing; a live ride is unaffected.

**Telling a petrol pump from a CNG station is harder than it looks.** Fuel stops are imported from OpenStreetMap, where `amenity=fuel` covers petrol, CNG, LPG and EV charging alike. The obvious filter — trust the `fuel:petrol` tag — fails: only 10 of 464 stations around Delhi carried that tag at all, so its absence proves nothing, and filtering on it flagged an Indian Oil forecourt as gas-only. Brand and name turned out to be the reliable signal, with positive evidence of petrol always winning, because plenty of real stations sell both.

---

## If you're testing this

**Clearing app data logs you out and loses anything that hadn't sent yet.** Your messages themselves are safe — they live on the server, and reopening a chat pulls its history back down. But a message that *failed* to send is only ever on your phone, so clearing data throws it away for good. Worth knowing because clearing data is the first thing most people try when an app misbehaves.

**Message search only finds what your phone has already received.** Chats are cached on the device and searched there, the same way WhatsApp does it — so right after installing, or after clearing data, search comes up empty until you have opened a few conversations. It fills in as you use the app.

**The home feed is people you follow.** Follow nobody and it fills with suggested posts instead, marked as such — you are not missing anything, there just isn't a graph yet. The discovery tab is where you go to find riders to follow.

**Some screens will look empty at first, and that is the app working correctly.** Rides, posts and roads are all made by riders, so on day one there aren't any — they fill in as people use it. Events are hosted by brands, so that shelf stays quiet until one posts. Petrol pumps are already there and cover Delhi NCR, and they extend themselves to new cities automatically as riders reach them.

**Location is asked for in context, never on launch.** If you say no, the map still works — you just lose "near me" ordering and the nearest-fuel pins. You can turn it on later from the Fuel chip on the map.

---

## Notes

Source is private — happy to walk through the code or the architecture on request.

This is a personal project, still pre-release. If something breaks, I'd genuinely like to hear about it.
