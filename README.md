# caldav-purge

Delete all events from a CalDAV calendar.

Works with any CalDAV server (Mailbox.org, Nextcloud, Fastmail, iCloud, etc.).

## Setup

```sh
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Usage

```
python caldav_purge.py [--url URL] [--username USERNAME] [--principal-path PATH] [--calendar CALENDAR] [--dry-run] [--yes] [--debug]
```

Any missing credentials are prompted interactively:

```
$ python caldav_purge.py
CalDAV server URL: https://dav.mailbox.org:443
Username: user@mailbox.org
Password:
Connecting to https://dav.mailbox.org:443 ...
Connected.
Available calendars:
  1. Personal
  2. Work
Select calendar [1-2]: 1
Selected calendar: Personal
Fetching events ...
Found 42 event(s).

This will permanently delete 42 event(s) from 'Personal'.
Type "yes" to confirm: yes

Deleting 42 event(s) ...
  Progress: 10/42
  Progress: 20/42
  Progress: 30/42
  Progress: 40/42
  Progress: 42/42

Done. Deleted: 42, Errors: 0
```

### Options

| Flag | Description |
|------|-------------|
| `--url` | CalDAV server URL (fallback: `CALDAV_URL` env var, then prompt) |
| `--username` | Username (fallback: `CALDAV_USERNAME` env var, then prompt) |
| `--principal-path` | Principal path to skip auto-discovery (e.g. `/principals/users/8/`) |
| `--calendar` | Calendar name — case-insensitive (fallback: interactive picker) |
| `--dry-run` | List events without deleting |
| `--yes`, `-y` | Skip confirmation prompt |
| `--debug` | Enable debug logging (shows HTTP requests) |

The password is read from the `CALDAV_PASSWORD` env var or prompted via `getpass` (never a CLI argument).

### Environment variables

For non-interactive use (scripts, cron):

```sh
export CALDAV_URL=https://dav.mailbox.org:443
export CALDAV_USERNAME=user@mailbox.org
export CALDAV_PASSWORD=secret
python caldav_purge.py --calendar Personal --yes
```

### Dry run

Preview which events would be deleted:

```
$ python caldav_purge.py --calendar Personal --dry-run
Fetching events ...
Found 3 event(s).

Dry run — listing events:
  - Team standup
  - Dentist appointment
  - Flight to Berlin

3 event(s) would be deleted.
```

### Troubleshooting

If the script hangs at "Connecting to …", the server may not support principal auto-discovery on the root URL. Use `--principal-path` to skip discovery:

```sh
python caldav_purge.py --url https://dav.mailbox.org:443 --username user@mailbox.org --principal-path /principals/users/8/ --dry-run
```

You can find your principal path in your CalDAV client settings or by checking the server's `.well-known/caldav` response.

To inspect the HTTP requests being made, use `--debug`:

```sh
python caldav_purge.py --url https://dav.mailbox.org:443 --username user@mailbox.org --debug --dry-run
```

## Common CalDAV URLs

| Provider | URL |
|----------|-----|
| Mailbox.org | `https://dav.mailbox.org:443` |
| Nextcloud | `https://your-server.com/remote.php/dav` |
| Fastmail | `https://caldav.fastmail.com` |
| iCloud | `https://caldav.icloud.com` |
