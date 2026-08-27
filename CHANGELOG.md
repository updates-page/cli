# Changelog

## 1.4.1

- `repository` and `bugs` in `package.json` pointed at the old repo, so every
  link on the npm page 404'd. Now `updates-page/cli`.
- `--help` advertised `https://updates.page/docs`, which 404s. The docs are at
  `https://docs.updates.page`.
- The README claimed API access required the Pro plan. It is included for the
  first 14 days of a new account and continues on Pro.
- The exit-code table was missing `1` and `7`.

## 1.4.0

Internal rewrite. Every command that publishes or reads content is unchanged —
same names, same flags, same arguments, and `updates categories` with no
subcommand still lists.

Two things did change:

- **`updates config --api-key` is gone.** Use `updates login`, or
  `updates login --token -` / `$UPDATESPAGE_TOKEN` unattended. A key passed as
  a command-line argument is saved in your shell history and readable from the
  process list, which is the whole reason the browser flow exists. A key saved
  by 1.3 in `~/.updatespage/config.json` is **not** read — sign in again.
- **`list` and `get` render differently.** `list` is a table; `get` is a field
  list followed by the content. If you were scraping either, switch to
  `--json`, which is the interface that stays stable.

Removing a command is a breaking change and would ordinarily mean 2.0. It was
a minor deliberately: 1.0 shipped days earlier and the flag it replaced is one
nobody should have been relying on, so a major would have announced a migration
that does not exist and spent the number this CLI's first real stability
commitment should get.

Node `>=22.13 <23 || >=23.4` is now required (1.3 needed 18+). The gap is real,
not a typo: 23.0–23.3 are newer than the 22.13 floor and still lack what it
provides.
