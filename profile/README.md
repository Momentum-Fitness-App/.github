# Momentum-Fitness-App

Momentum records and explains your sport (skiing, snowboarding, racquet sports, runs, rides, hikes,
karting and sim racing) on your watch and phone, and shares it with friends. This page is the
starting point for everyone joining the project.

File paths below are in [momentum-app](https://github.com/Momentum-Fitness-App/momentum-app) unless a section says otherwise.

## Where we talk

- **Planning, ideas and decisions:** [GitHub Discussions](https://github.com/orgs/Momentum-Fitness-App/discussions).
  Start a discussion before any large change, a new feature or anything that touches another
  maintainer's area. Decisions are recorded there, then in the repo (`.agents/rules/decision-log.md`).
- **Work in progress:** pull requests in each repository. **Bugs:** issues in the repository they belong to.
- **Cross-team to-dos** (an API change, a feature another client must mirror): a file in
  [`sync/`](https://github.com/Momentum-Fitness-App/momentum-app/tree/master/sync) in `momentum-app`.

## People

| Who | Owns |
| --- | --- |
| [@makbn](https://github.com/makbn) | iPhone and Apple Watch apps, design system, overall product |
| [@BeardedDonut](https://github.com/BeardedDonut) | Backend, database, API performance, Explore map, hosting, `gearscraper`, `hikescraper` |
| [@MajeedAskari](https://github.com/MajeedAskari) | Android phone and Wear OS apps (mirroring iOS) |

## Repositories

| Repository | What it is | Owner |
| --- | --- | --- |
| [momentum-app](https://github.com/Momentum-Fitness-App/momentum-app) | The monorepo: API contract (`api/openapi.yaml`), Spring Boot backend, iOS + watchOS apps, Android + Wear OS apps | everyone, per folder |
| [gearscraper](https://github.com/Momentum-Fitness-App/gearscraper) | Builds the gear catalogue: current retail models plus 2015–2024 models from review archives, with specs and images | @BeardedDonut |
| [hikescraper](https://github.com/Momentum-Fitness-App/hikescraper) | Builds one JSON file per hiking route or trail from OpenStreetMap regional extracts (not AllTrails) | @BeardedDonut |

## Your first day

1. **Get access:** an invite to this organization; backend maintainers also need the production
   host and the Cloudflare account (ask @makbn).
2. **Read, in this order:** `README.md` (setup), `CLAUDE.md` (how we work together),
   `feature_list.md` (what exists, per platform), `ios/DESIGN.md` (the design system for every
   client), then your area's `CLAUDE.md` (`backend/`, `ios/`, `android/`).
3. **Clone and set your role.** In `momentum-app`, copy `.claude/settings.local.json.example` to
   `.claude/settings.local.json` and set `MOMENTUM_ROLE` to `backend`, `apple` or `android`.
   Claude Code then runs `tools/sync-check.py` at every session start and tells you what the others
   changed that you have to follow.
4. **Run your part** with the quick start in `README.md`. The backend needs JDK 21 and Docker; the
   Apple apps need Xcode 26.1+ and XcodeGen.
5. **Say hello** in Discussions and pick an item from your `sync/` queue or `feature_list.md`.

### Rules that keep three people in sync

- `api/openapi.yaml` is the contract. Change it first, add a `sync/` entry for everyone who must
  follow, run `./generate-api.sh`. CI rejects a contract change without a `sync/` entry.
- Every client follows `ios/DESIGN.md` and mirrors the others; only the platform idiom changes.
- Anything a user can see updates `feature_list.md` in the same PR; setup changes update `README.md`.
- Branch from `master`, one topic per PR; the folder's owner reviews (`.github/CODEOWNERS`).

## The scrapers

Both are Python command-line tools that build datasets offline. Today neither is wired into the
backend yet: the backend still builds its gear catalogue with `backend/tools/gear-catalog/build_catalog.py`
and fetches trails live from Overpass (`backend/.../trail/`). Connecting them is open backend work (below).

### gearscraper

```sh
python3 -m venv .venv && source .venv/bin/activate && pip install -e .   # Python 3.11+
python -m gearscraper doctor                     # check engines, proxy and kinds.json
python -m gearscraper scrape --kinds skis --sites evo --engine auto --limit-products 50
python -m gearscraper harvest-historic --kinds skis,snowboard --years 2018-2024
python -m gearscraper historic next --limit 10   # agent extraction protocol, resumable
```

- `kinds.json` is the schema (fields, units, choices); it must stay aligned with the backend's
  `backend/tools/gear-catalog/kinds.json`, which is what the app's gear forms use.
- Output (`catalog/`, `images/`, `state/progress.sqlite`, `logs/`) is git-ignored; it lives on the
  machine that ran the scrape. Agents working on it read `HISTORIC_AGENT_GUIDE.md` first.
- Respect each site's terms and rate limits (`--delay-ms`); never commit proxy credentials.

### hikescraper

```sh
python3 -m venv .venv && source .venv/bin/activate && pip install -e .   # Python 3.10+
hikescrapper --location Alberta                  # downloads the Geofabrik extract once, reuses it
hikescrapper --location Canada                   # several GB; needs disk and memory
pip install -e ".[dev]" && pytest
```

- The command is still spelled `hikescrapper` (package name in `pyproject.toml`); the repo was renamed.
- Output is `output/<location>/trails/osm_<type>_<id>.json` plus `manifest.json` and `summary.json`.
  The repository currently commits about 1.2 GB of output (~250,000 files): move it to a release
  asset or object storage and git-ignore `output/` before it grows further.
- Data is OpenStreetMap, under the ODbL: attribution and share-alike apply to what we ship. Never
  copy AllTrails content.

## Environments

| What | Where |
| --- | --- |
| Production API (release builds) | `https://momentum.makbn.com/api/v1` |
| Admin panel | [https://momentum.makbn.com/admin](https://momentum.makbn.com/admin) (admin accounts only) |
| Local development | `http://localhost:8080/api/v1` (`backend/docker-compose.yml`, or `./gradlew bootRun`) |

## Backend hosting

Production is served from `momentum.makbn.com` through a Cloudflare Tunnel and deployed by hand
today. The backend maintainer's next job is to automate that: either a CI deploy to the current
host, or moving the backend to @BeardedDonut's environment behind the same hostname. The full
current setup, both options step by step, and the open backend work are in
[`organization.md`](https://github.com/Momentum-Fitness-App/.github/blob/main/profile/README.md)
(members only). Decide in Discussions.
