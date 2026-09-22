# Diary

A local-first journal for Android tablets and phones. Write a page a day, tag it
with a mood, attach photos, and keep a shelf of the films and books that went
with it — all stored on the device.

Built with Flutter. No account, no sync server, no analytics.

---

## What it does

**A page a day.** A month calendar is the home screen. Each day cell shows the
mood recorded for it and whether anything was written. Tap a day — past or
future — to open its page.

**Moods.** Ten hand-animated moods. One per day, changeable at any time, with a
year view showing which ones came up most.

**Photos.** Attached to a day rather than embedded in the prose, and shown as a
scrolling rail on both the page and the journal feed. Editing a paragraph can't
move a photo, and deleting a line can't orphan one.

**Full-text search.** SQLite FTS5 with a trigram tokenizer, so search works on
languages that don't put spaces between words.

**A media shelf.** Search films and books, rate them in half-stars, and tie them
to the day you watched or read them. Metadata and cover art are fetched once and
then cached locally — the shelf browses fully offline and nothing leaves the
device after that first search.

**A lock.** Optional passcode and biometric unlock. PBKDF2-SHA256 at 150,000
iterations, salt and hash in the Android Keystore, throttling persisted before
the comparison runs so force-quitting mid-lockout doesn't reset the counter. The
app-switcher thumbnail is masked with `FLAG_SECURE` while the lock is on.

**Export and backup.** Offline PDF export of a month or a year, and an encrypted
`.diarybak` archive (AES-256-GCM, PBKDF2 at 200,000 iterations) that the user
creates and moves themselves.

**Calendar app role.** Registers the Android calendar intent filters, so Diary
can be set as the default calendar handler and open the right day when another
app asks for one.

---

## Privacy

This is the part that drove most of the design, so it's worth being specific.

| Data | Where it lives | Leaves the device? |
|---|---|---|
| Journal text | SQLite, app sandbox | Never |
| Photos | App sandbox, relative paths | Never |
| Moods | SQLite | Never |
| Passcode | Never stored in any form; only a PBKDF2 hash + salt, in the Keystore | Never |
| Film/book **search terms** | Sent to TMDB / Google Books to perform the search | Yes, that request only |
| Film/book metadata + cover art | Fetched once, then cached locally | Inbound only |
| Analytics, crash reporting, telemetry | — | None. There is none. |

No journal entry, image, or mood is ever transmitted anywhere. The only outbound
network requests the app makes are the media searches the user explicitly
performs, and they carry nothing but the search string.

There is no backend. Backups are files the user makes and moves themselves.

---

## API keys

Film and book search need the user's own free API keys, entered in
**Settings → Media search** on the device. They are stored locally and are never
committed, bundled, or transmitted anywhere except to the API they belong to.

- **TMDB** — required for film search. Free at
  [themoviedb.org/settings/api](https://www.themoviedb.org/settings/api).
- **Google Books** — optional but recommended. The API answers unauthenticated
  requests on a shared anonymous quota that is usually exhausted, which shows up
  as an HTTP 429 and looks exactly like the search being broken. A free key from
  the Google Cloud console fixes it.

A key can also be baked in at build time for development:

```bash
flutter build apk --release \
  --dart-define=TMDB_API_KEY=… \
  --dart-define=GOOGLE_BOOKS_API_KEY=…
```

A key entered in Settings takes precedence over the compile-time one.

### Attribution

This product uses the TMDB API but is not endorsed or certified by TMDB.

Book data is provided by the Google Books API. This application is not
affiliated with or endorsed by Google.

---

## Architecture

Clean Architecture with MVVM, three layers, dependencies pointing inward.

```
lib/
  core/        theme, layout, routing, settings, time
  data/        Drift tables, repositories, file store, remote APIs
  features/    calendar, entry, mood, library, lock, backup, export, onboarding
```

- **State** — Riverpod. `Notifier` for synchronous state, `AsyncNotifier` where
  there's I/O, stream providers over Drift so a write anywhere repaints the
  calendar.
- **Database** — Drift over SQLite. Foreign keys are switched on explicitly in
  `beforeOpen`; SQLite leaves them off by default, which silently turns every
  `ON DELETE CASCADE` into a no-op.
- **Dates** — day-precision columns store **epoch days** (days since
  1970-01-01 in civil time), instants store epoch milliseconds UTC. One
  conversion seam, `core/time/epoch_day.dart`, and the Kotlin side mirrors the
  same arithmetic so a calendar intent for the 19th can't open the 18th across
  a timezone boundary.
- **Layout** — written width-first against Material 3 window size classes, so a
  tablet gets a two-pane layout and a phone gets pushed routes, rather than the
  tablet being treated as a large phone.

---

## Building

Requires the Flutter SDK and an Android SDK with platform 36.

```bash
flutter pub get
dart run build_runner build --delete-conflicting-outputs   # Drift codegen
flutter build apk --release
```

Minimum Android 7.0 (API 24), which is the floor across `local_auth`,
`flutter_secure_storage`, and `flutter_local_notifications`.

---

## Licence

MIT — see [LICENSE](LICENSE).

Bundled fonts (Pretendard, Literata, Quicksand, JetBrains Mono) are used under
the SIL Open Font License; their licence files ship with the app.
