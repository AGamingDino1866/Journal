# Privacy Policy — Diary

_Last updated: 22 September 2026_

Diary is a local-first journal. It has no user accounts, no backend server, and
no analytics.

## What is collected

Nothing. There is no server to collect it with.

Everything you write — journal entries, photos, moods, ratings, reminders — is
stored in a database and file directory inside the app's private sandbox on your
own device. None of it is transmitted anywhere.

## Network requests

Diary makes outbound network requests in exactly two situations, both of which
you trigger deliberately:

1. **Searching for a film.** The text you typed is sent to
   [The Movie Database (TMDB)](https://www.themoviedb.org/) so it can return
   matching titles.
2. **Searching for a book.** The text you typed is sent to the
   [Google Books API](https://developers.google.com/books) for the same reason.

When you add a result to your shelf, its cover image is downloaded once and
cached on the device. After that, your library browses entirely offline.

These requests contain your search term and your own API key. They do not
contain your journal entries, your photos, your moods, or any identifier for
you or your device beyond what any HTTPS request necessarily reveals to the
server (such as your IP address). Those services have their own privacy
policies, which apply to the request once it reaches them.

If you never use film or book search, Diary makes no network requests at all.

## API keys

The TMDB and Google Books API keys you enter in Settings are stored locally on
your device and are sent only to the service they belong to.

## Your passcode

If you set a passcode, the passcode itself is never stored — not in the
database, not in preferences, not anywhere. Only a PBKDF2-SHA256 hash and a
random salt are kept, and those live in the Android Keystore rather than in the
app's files.

## Backups

Backups are encrypted files that you create and move yourself. They are not
uploaded anywhere by the app. Android's automatic cloud backup and
device-to-device transfer are both switched off for this app, so your journal is
not copied off the device by the operating system either.

## Deleting your data

Uninstalling the app removes the database, all attached photos, and the stored
passcode hash. There is nothing left anywhere else.

## Contact

Open an issue on this repository.
