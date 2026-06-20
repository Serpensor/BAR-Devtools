# Team Setup — Serpensor/Brian BAR Fork

This is our fork of BAR-Devtools, the orchestrator that stands up the full self-hosted
BAR stack: **Teiserver** (lobby/accounts), **SPADS** (autohost), **RAPID** (content
delivery), **PostgreSQL**, and the **bar-lobby/Chobby** client.

`repos.conf` already points every component at our **Serpensor forks** — so cloning this
one repo and running setup pulls the entire wired stack. We fork everything; we edit
whatever we want across the whole stack.

## One-time machine setup (Windows)

The whole server stack runs inside **WSL2** (Linux). The game engine runs as a native
Windows process. Do this once:

```powershell
# In PowerShell
wsl --install -d Ubuntu-24.04      # reboot if prompted; set a UNIX username + password on first launch

# Optional: cap WSL2 resources (tune to your machine; leave ~5-6 GB for Windows)
@"
[wsl2]
memory=12GB
swap=8GB
processors=4
"@ | Set-Content $env:USERPROFILE\.wslconfig
```

You also need **Docker Desktop** running with WSL2 integration enabled for your Ubuntu distro.

## Bring up the stack (inside WSL Ubuntu)

Clone into the WSL **ext4** home (`~/`), NOT `/mnt/c` — the Windows filesystem is too slow:

```bash
git clone https://github.com/Serpensor/BAR-Devtools.git ~/BAR-Devtools
cd ~/BAR-Devtools

bash scripts/bootstrap.sh          # installs `just` >= 1.31 into ~/.local/bin
exec "$SHELL" -l                   # reload PATH

just setup::init                   # one-time setup; answers all prompts up front.
                                   #   On WSL it also asks for BAR_DATA_DIR (the
                                   #   Windows-side data dir the engine reads).
just services::up                  # Postgres + Teiserver
just services::up spads            # add SPADS autohost (~300MB game-data download first run)
```

Teiserver comes up at **http://localhost:4000** — log in `root@localhost` / `password`.
The `spadsbot` account (`spadsbot` / `password`) is created automatically.

## Daily commands

```bash
just doctor                 # read-only env audit — run this first when anything's off
just --list                 # all recipe modules
just services::up | down    # start / stop the stack
just services::logs <svc>   # tail logs (teiserver, spads, postgres)
just services::reset        # nuke volumes + restart (destructive, last resort)
just repos::clone [repo]    # clone/sync component repos per repos.conf
just bar::launch            # launch the client against the local stack
```

## Working on the forks

Each component lives in its own gitignored subdir (cloned by `just repos::clone`) and is a
full git checkout of our fork. Standard flow: branch → commit → push to our fork → PR.

To pull upstream (official BAR) changes into a component, add the upstream remote once:

```bash
cd <component-dir>
git remote add upstream <original-upstream-url>   # the beyond-all-reason/... or Yaribz/... URL
git fetch upstream && git merge upstream/<branch>
```

## Personal overrides

`repos.local.conf` (gitignored) overrides any `repos.conf` row for just your machine —
e.g. to point a component at a local checkout via a 5th `local_path` column, or test a
personal branch. It never affects teammates. See README.md for the format.

## Scope note

This stack is currently **localhost-only**. Letting teammates + friends connect remotely
(exposing Teiserver/SPADS over Tailscale or a VPS, publishing mods to our RAPID) is the
next milestone — not yet wired here.
