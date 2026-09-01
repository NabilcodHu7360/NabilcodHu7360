# Publish and maintain this profile

The profile is ready for `github.com/NabilcodHu7360`. Its core artwork, charts, and
project cards are files in this repository; GitHub Actions refresh the generated files
on a schedule.

## Publish it

1. On GitHub, create a new **public** repository named `NabilcodHu7360` exactly.
2. Do not add a README, license, or `.gitignore` on the creation screen.
3. From this folder, run:

```bash
git init
git add .
git commit -m "feat: publish profile README"
git branch -M main
git remote add origin https://github.com/NabilcodHu7360/NabilcodHu7360.git
git push -u origin main
```

4. Open the profile Overview page and confirm the README appears.
5. Apply the sidebar copy and pins in `PROFILE-FIELDS.md`.
6. In **Settings → Actions → General**, set Workflow permissions to **Read and write**.
7. Run **Charts and cards** once from the Actions tab. The included cards already work;
   this first run confirms scheduled refreshes can commit updates.

The optional Metrics and Snake workflows add extra generated files. The finished README
does not depend on them, so the profile remains clean even before those workflows run.

---

## The files you edit

| file | what it controls |
|---|---|
| `README.md` | the layout, all the words, every link |
| `assets/projects.json` | which four repos get cards, and their one-line pitches |
| `PROFILE-FIELDS.md` | recommended GitHub sidebar text and pinned repositories |
| `.github/workflows/*.yml` | automated refresh schedules and timezone |

## The files you don't

`scripts/*.py` are the generators. `assets/*.svg` is all generated output — editing an
SVG by hand means the next scheduled run overwrites you.

`.gitattributes` looks boring and is load-bearing. It forces LF line endings on
everything the Linux runners execute. A CRLF ending inside a `run: |` block makes bash
fail with `command not found` on a script that is otherwise perfectly correct.

---

## Regenerating things by hand

The optional image utility uses Pillow. Everything else is Python standard library,
deliberately — the runners can then execute the scripts without an install step,
which is one fewer thing to break at 3am.

```bash
pip install pillow
```

### The radars

```bash
# self-rated — edit assets/skills.json first
python scripts/radar.py --data assets/skills.json -o assets/radar
```

The numbers in `skills.json` are a starting point somebody else picked. Change them to
what you actually think. Five to eight axes reads best; past eight the labels collide.
Keep the labels short for the same reason — that's why they're `Modeling` and `Evals`
rather than `Stats & Modeling` and `LLM Evaluation`. And don't put everything at 90: a
radar where every axis is maxed is a circle, and a circle communicates nothing.

The second radar needs the GitHub API and so only runs on the Actions runner. `--curve
0.4` is the important flag — raw language byte counts are brutally lopsided, and a
linear radar of them is one spike with six dots around it.

### The cards

```bash
python scripts/cards.py --user NabilcodHu7360 --projects assets/projects.json --out assets
```

Also needs the API. Stars, forks and language come live on every run; the descriptions
come from `projects.json`. Without a token the stat card renders three tiles instead of
six — contributions and streaks need the GraphQL API, which is what `METRICS_TOKEN` is
for.

### Looking at it before pushing

Open `preview.html` in a browser. It shows every asset twice, once on a dark card and
once on a light one, exactly the way GitHub will render them.

---

## The two GitHub settings that break everything

If a workflow fails on its first run, it is one of these two. It is essentially always
one of these two.

**A · let Actions write to the repo.**
Settings → Actions → General → Workflow permissions → **Read and write permissions** → Save.
By default workflows can only read, and all three of these need to commit files back.

**B · give the metrics workflow its own token.**
`lowlighter/metrics` reads profile-level data — the contribution calendar, achievements —
that the token GitHub hands every workflow cannot see.

1. github.com/settings/tokens → **Generate new token (classic)**. It must be classic; a
   fine-grained token will not work here, and that single mistake accounts for most
   "the metrics workflow just won't run" problems.
2. Scope: tick `read:user`. Add `repo` too if you want private contributions counted.
3. Copy it — that's the only time you'll see it.
4. Settings → Secrets and variables → Actions → New repository secret, named
   `METRICS_TOKEN` exactly, because the workflows reference it literally.

When that token expires, the metrics workflow starts failing months from now for no
apparent reason. That's the one recurring maintenance task this setup has.

---

## The three workflows

| workflow | makes | runs |
|---|---|---|
| `metrics.yml` | isometric calendar, language mix, achievements | every 6h |
| `snake.yml` | the snake eating the contribution graph | every 12h |
| `radar.yml` | both radars, the stat card, the project cards | daily, 03:30 UTC |

All three have `workflow_dispatch`, which is what puts the **Run workflow** button in the
Actions tab.

`metrics.yml` deliberately has no `push` trigger. It commits its own output using
`METRICS_TOKEN`, and a push made by a personal access token *does* re-trigger workflows —
unlike one made by the built-in `GITHUB_TOKEN`. With a push trigger, each run's commit
starts the next run.

The snake images live on an orphan `output` branch so the animation frames never clutter
the commit history, which is why the README references them by full raw URL instead of a
relative path. That URL 404s until the snake workflow has completed once. That's
expected, not a bug — the branch doesn't exist yet.

---

## When it breaks

| symptom | what it actually is |
|---|---|
| Images broken for everyone but you | The repo is private. You see them because you're authenticated. Make it public. |
| Images broken for you too | Not pushed yet. Relative paths only resolve once the file is on `main`. |
| Snake images 404 | The snake workflow hasn't finished a run, or setting A was skipped so it couldn't create the `output` branch. |
| Metrics workflow fails | `METRICS_TOKEN`: missing, misspelled, expired, or created fine-grained instead of classic. |
| Workflow is green but nothing changed | It couldn't push. Workflow permissions are still read-only — setting A. |
| `command not found` inside a `run:` block | CRLF line endings. This is exactly what `.gitattributes` prevents; don't delete it. |
| The typing banner is blank | Something in `lines` isn't URL-encoded. Spaces are `+`, a literal `+` is `%2B`, and an `&` must be `%26` or it silently cuts the URL in half. |
| A chart is invisible | You're viewing it in the theme it wasn't drawn for. The `<picture>` block is missing a `source`, or points at a file that didn't generate. |
| The language radar is one giant spike | Lower `--curve` to `0.3`, and check whether a committed `dist/` or `node_modules/` is inflating one language — if so add it to `--exclude`. |
| Stat numbers look low | Private contributions aren't counted unless the token carries the `repo` scope. |

When a workflow fails, open the Actions tab, click the red run, and expand the step that
failed. The error is nearly always in the last four lines, and it nearly always names the
token or the permission.

---

## One habit

The **currently building** line in §01 is the highest-value line on the whole page and
the one that goes stale fastest. Put a reminder somewhere to update it.

---

## Credit

The generator scripts (`dotify.py`, `radar.py`, `cards.py`), the workflow structure and
the `<picture>` approach come from [Gargi Bhardwaj's profile
repo](https://github.com/gargibhardwaj24/gargibhardwaj24) and the accompanying guide.
`surface.py`, the palette, the layout and all of the words are not hers.
