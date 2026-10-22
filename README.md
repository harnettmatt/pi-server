# pi-server

Host-level config for the Raspberry Pi: the web front door (Caddy + Cloudflare dynamic DNS) and
notes on how the apps that run on it get deployed.

## What's in here

`docker-compose.yml` runs:
- **cloudflare-ddns**: keeps the Cloudflare A record pointed at the Pi's public IP when it changes
- **caddy**: reverse proxy that forwards
    - `themesong.io/` to the themesong-client service on 8080 (from the themesong-client repo)
    - `api.themesong.io/` to the themesong-api service on 8000 (from the themesong-api repo)

## Apps on this Pi

| App | Repo | Deploys by |
|---|---|---|
| themesong client | [themesong-client](https://github.com/harnettmatt/themesong-client) | GitHub Actions → SSH (`PI_HOST`/`PI_USERNAME`/`PI_KEY` secrets) |
| themesong API | [themesong-api](https://github.com/harnettmatt/themesong-api) | GitHub Actions → SSH |
| this repo | pi-server | GitHub Actions → SSH, then `make rebuild` in `~/pi-server` |
| Wheel of Galby | [wheel-of-galby](https://github.com/harnettmatt/wheel-of-galby) (private) · https://wheelofgalby.com | Self-hosted GitHub Actions runner on the Pi |
| Media stack (Jellyfin, Dispatcharr, *arr) | pi-media-stack (private) | By hand; config kept in sync with the Pi |

The SSH deploys `cd` into a folder named after the repo, so each repo lives at `~/<repo name>`.

## Self-hosted runner (Wheel of Galby)

The Pi is only reachable from the home network for that app, so its deploys run on a GitHub Actions
runner installed on the Pi itself, as a service that starts on boot:

- installed in `~/actions-runner`, registered to wheel-of-galby (formerly galbraith-route-generator) as `galby-pi`
- service: `actions.runner.harnettmatt-galbraith-route-generator.galby-pi` (named before the rename)
  (`sudo ~/actions-runner/svc.sh status|stop|start`)
- to (re)install: download the arm64 release from https://github.com/actions/runner/releases, then

      ./config.sh --unattended --replace --url https://github.com/harnettmatt/wheel-of-galby \
        --token <token from: gh api -X POST repos/harnettmatt/wheel-of-galby/actions/runners/registration-token --jq .token> \
        --name galby-pi --labels galby-pi
      sudo ./svc.sh install mattpi && sudo ./svc.sh start

Self-hosted runners should only be attached to private repos: on a public repo, anyone's pull
request could end up running code on the Pi.
