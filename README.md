# gitdna

Point it at a directory of git repos, get back a single self-contained HTML
dashboard of _your_ commit history across all of them.

- **When you code** — hour-of-day, day-of-week, and a 24×7 heatmap
- **Momentum** — longest streak, biggest 1-hour burst, longest silence, most
  prolific single day
- **Timeline** — commits per month, last-365-days bars, per-year totals
- **Lines of code** — added / removed / net, with an opt-in "raw" number that
  includes lockfiles and vendor dirs
- **LOC by month / hour / weekday / year** stacked bars
- **Top repos, top languages, biggest single commit, favorite commit verbs**

No Python dependencies. Charts are rendered client-side by
[Chart.js](https://www.chartjs.org/) loaded from jsDelivr.

## Install

```sh
git clone https://github.com/tuncenator/gitdna.git
cd gitdna
chmod +x gitdna
ln -s "$PWD/gitdna" ~/.local/bin/gitdna   # optional
```

Requires **Python 3.9+** and **git** on `PATH`. That's it.

## Usage

```sh
gitdna                                  # scan . using your local git identity
gitdna ~/src                            # scan another root
gitdna ~/src --open                     # scan and open the report
gitdna ~/src -a 'alice\|alice@co.com'   # explicit author pattern
gitdna ~/src -o ~/dna.html -t "My DNA"  # custom output path and title
```

Output defaults to `<root>/gitdna.html`.

### Author pattern

`--author` is passed straight to `git log --author`, so it is a regex in
escaped-alternation form: `alice\|bob@co.com`. When you omit it, gitdna infers
a pattern from your local `git config user.name` and `user.email`.

### All flags

| flag | default | notes |
|---|---|---|
| `root` | `.` | directory to recursively scan for `.git` dirs |
| `-a`, `--author` | `git config` | regex author filter (`A\|B\|C`) |
| `-o`, `--output` | `<root>/gitdna.html` | output HTML path |
| `-t`, `--title` | auto | dashboard title |
| `-d`, `--max-depth` | `6` | how deep to walk for nested repos |
| `-j`, `--jobs` | `min(16, 2×CPU)` | parallel workers |
| `--timeout` | `60` | per-repo git timeout, seconds |
| `--json PATH` | — | also dump the raw analysis JSON |
| `--open` | off | open the output in the default browser |

## How it works

Two `git log` calls per repo, run in a thread pool:

1. `git log --all --author=... --pretty=...` — commit list
2. `git log --all --no-merges --author=... --numstat` — per-file additions /
   deletions

Duplicate commits (e.g. same SHA found in multiple clones) are deduplicated by
SHA. Timestamps are taken from the author date so the hour-of-day reflects
_when you wrote it_, not when it eventually landed on `main`.

For line counts, paths matching common noise patterns are excluded from the
"filtered" LOC totals (you still see the raw numbers alongside):

- lockfiles: `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `Cargo.lock`,
  `poetry.lock`, `composer.lock`, `Gemfile.lock`, `go.sum`
- generated / minified: `*.min.js`, `*.min.css`, `*.map`, `*.snap`
- vendored trees: `vendor/`, `node_modules/`, `dist/`, `build/`
- binaries: `.svg`, `.png`, `.jpg`, `.jpeg`, `.gif`, `.ico`, `.pdf`, `.mp4`,
  `.zip`, `.tar`, `.tar.gz`, `.lockb`

The list lives at the top of [`gitdna`](./gitdna); edit to taste.

## Output

A single HTML file. All data is embedded as inline JSON; Chart.js loads from
CDN. Send it, commit it, host it — it is one file.

## Privacy

Everything runs locally. No network calls are made during analysis. The
generated HTML pulls Chart.js from jsDelivr when opened in a browser; if you
prefer fully offline output, vendor Chart.js and replace the `<script src>` in
the template.

## License

MIT — see [LICENSE](./LICENSE).
