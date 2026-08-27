# updates.page CLI

Publish changelog posts to [updates.page](https://updates.page) from your terminal.

```bash
npm install -g @updatespage/cli
updates login
updates publish --title "Dark mode" --content "<p>It's here.</p>"
```

Needs Node `>=22.13 <23 || >=23.4`.

## Signing in

```bash
updates login
```

Opens your browser, you approve, done. Over SSH or anywhere without a browser
it shows a short code to enter from another device instead. `updates login
--device` forces that flow.

Tokens are stored in `~/.updatespage/credentials.json` with `0600`
permissions. `updates logout` revokes the token server-side, not just locally.

For CI, put the token in the environment — never in an argument, where it
lands in your shell history and the process list:

```bash
UPDATESPAGE_TOKEN=… updates list
updates login --token - < token.txt   # or read it from stdin
```

Juggling accounts? `--profile work` on any command keeps them separate.

API access is included for the first 14 days of a new account and continues on
the Pro plan. A failure that mentions API access rather than credentials means
the trial lapsed — [upgrade](https://app.updates.page/account/billing) rather
than signing in again.

## Writing a post

`--content` takes an **HTML fragment**, not Markdown. Markdown doesn't error —
it publishes literally, so your changelog shows `**Dark mode**` as raw
characters on a live page. Use `<p>`, `<ul>`/`<li>`, `<strong>`, `<a href>`,
`<code>`.

For anything longer than a sentence, write it to a file first — this also
avoids the shell expanding backticks and `$(…)` inside your release notes:

```bash
content=$(cat <<'HTML'
<p>Dark mode is in Settings, and follows your OS by default.</p>
HTML
)
updates draft --title "Dark mode" --content "$content"
```

## Commands

| Command | What it does |
|---|---|
| `updates publish [id]` | Create and publish a post, or publish an existing draft |
| `updates draft` | Create a post without publishing it |
| `updates update <id>` | Change fields on an existing post |
| `updates unpublish <id>` | Revert a published or scheduled post to a draft |
| `updates delete <id>` | Delete a post permanently |
| `updates list` | List your posts (`--status draft\|scheduled\|published`) |
| `updates get <id>` | Show one post in full |
| `updates categories` | List categories (`create`, `update`, `delete` too) |
| `updates upload <file>` | Upload an image and print its URL |
| `updates login` / `logout` / `whoami` | Sign in, out, and check who you are |
| `updates doctor` | Show the resolved setup and where each value came from |

`updates <command> --help` has the details of any one.

### Post fields

`publish`, `draft` and `update` share these:

| Flag | Effect |
|---|---|
| `--title <title>` | Post title |
| `--content <html>` | Post body |
| `--summary <text>` | One line, shown in feeds and embeds |
| `--category-id <id>` | File under a category (ids from `updates categories`) |
| `--url <url>` | Point the entry at an external page |
| `--private` / `--public` | Hide from / show on the public changelog |
| `--cover-image <path>` | Set the cover from a local png/jpg/gif/webp |

`--url` sends readers elsewhere instead of opening the post — it doesn't
replace the body. When you're **creating** a post, `--title` and `--content`
are still required alongside it. Changing the URL on a post that already
exists needs neither: `updates update <id> --url …` takes any subset of
fields.

### Scheduling

```bash
updates publish 123 --at 2026-09-01T09:00:00Z
```

ISO 8601. A value with no timezone is read as local time, so pass an explicit
`Z` or offset in CI. Impossible dates and daylight-saving gaps are rejected
rather than silently shifted.

## Scripting

Every command takes `--json`: structured data on stdout, nothing else.
Progress, warnings and prompts go to stderr.

```bash
updates list --status draft --json | jq -r '.posts[].id'
updates upload shot.png --json | jq -r .url
```

Failures are JSON too, so a script never has to parse English:

```json
{ "ok": false, "error": { "code": "auth.not_signed_in", "message": "…", "hint": "Run `updates login`." } }
```

And the exit code says which kind of failure it was:

| Code | Meaning |
|---:|---|
| `0` | Success |
| `1` | Anything without a more specific code |
| `2` | Usage — unknown flag, missing argument, bad value |
| `3` | Configuration problem |
| `4` | Not signed in, or the token was rejected |
| `5` | Network failure or server error |
| `6` | The thing you named does not exist |
| `7` | It exists, but is in the wrong state for this operation |
| `130` | Cancelled (Ctrl-C) |

Other global flags: `-q/--quiet`, `-y/--yes`, `--verbose`, `--color` /
`--no-color`, `--profile`, `--config`. Colour switches off automatically when
output isn't a terminal, and `NO_COLOR` is honoured.

## Something wrong?

```bash
updates doctor
```

Prints the endpoint, config file and credential store — and *where each value
came from*, which is usually the answer to "works on my machine, not in CI".
Exits non-zero if anything is broken, so it doubles as a health check.
`--offline` skips the checks that need the network.

## Development

```bash
npm install
npm run build      # tsc
npm run typecheck  # includes the tests
npm test           # node --test, in-process, no network
npm run updates -- list   # run from source
```

Built with [cli-starter](https://github.com/stratuslabs/cli-starter): `src/core/`
is that framework unmodified, `src/commands/` is this CLI.

## Links

[Documentation](https://docs.updates.page) ·
[Using updates.page from an agent](https://docs.updates.page/agents) ·
[Agent tooling](https://github.com/updates-page/agent-tooling)

## Licence

MIT.
