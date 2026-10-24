# Instructions

Node.js 24.15.0 image for Pterodactyl and Pelican. Works on x64 (amd64) and arm64 nodes.

Built on the official `node:24.15.0-trixie-slim` image (Debian 13), set up the same way as the official Pelican Node.js yolk. It includes:

- Node.js 24.15.0 and the latest npm at `/usr/local/bin`
- `tsx`, `typescript`, `ts-node` and `@types/node`
- pnpm (through corepack)
- Build tools and python3 for packages that need compiling (e.g. `sqlite3`, `bcrypt`)
- ffmpeg, git, sqlite3, curl, zip/unzip and network tools

## Changing the Node.js version

Change the tag on the `FROM` line in the `Dockerfile`, for example `node:24.16.0-trixie-slim`, and update the `version` label and the image tag you push. Check that the tag exists at https://hub.docker.com/_/node/tags first.

## Clone repo

```
git clone https://github.com/Plux-Modifications/node-ptero.git
cd node-ptero
```

## Automatic builds (GitHub Actions)

The workflow in `.github/workflows/build.yml` builds the image for x64 and arm64 and pushes it to `ghcr.io/<OWNER>/node-ptero`. It runs when `Dockerfile` or `entrypoint.sh` changes on `main`/`master`, or manually from the Actions tab (Build and push image → Run workflow).

It tags the image with the `version` label from the `Dockerfile` (e.g. `24.15.0`) and `latest`. No secrets are needed, it uses the repo's built-in `GITHUB_TOKEN`.

After the first run, go to the package on GitHub (your profile or org → Packages → node-ptero → Package settings) and set its visibility to Public, so your nodes can pull it without logging in.

## Build & Push manually

Log in to GitHub's container registry:

```
echo <YOUR_GITHUB_TOKEN> | docker login ghcr.io -u <YOUR_GITHUB_USERNAME> --password-stdin
```

Build for both x64 and arm64 and push in one step:

```
docker buildx create --use
docker buildx build --platform linux/amd64,linux/arm64 -t ghcr.io/<YOUR_GITHUB_USERNAME>/node-ptero:24.15.0 --push .
```

If you only need the same architecture as the machine you're building on, a normal build works too. This needs BuildKit, which is on by default since Docker 23 (on older versions, run `export DOCKER_BUILDKIT=1` first):

```
docker build -t ghcr.io/<YOUR_GITHUB_USERNAME>/node-ptero:24.15.0 .
docker push ghcr.io/<YOUR_GITHUB_USERNAME>/node-ptero:24.15.0
```

To create a GitHub token, go to https://github.com/settings/tokens and click Generate new token (classic). Check `write:packages` and create the key.

If you're pushing to an organization, you'll need to also check `write:org`

## Use it in the panel

This image is a drop-in replacement for the official `ghcr.io/pelican-eggs/yolks:nodejs_24` image, so it works with the generic Node.js egg for both panels:

- Pelican: [egg-nodejs-generic.yaml](https://raw.githubusercontent.com/pelican-eggs/generic/refs/heads/main/nodejs/egg-nodejs-generic.yaml)
- Pterodactyl: [egg-pterodactyl-nodejs-generic.json](https://raw.githubusercontent.com/pelican-eggs/generic/refs/heads/main/nodejs/egg-pterodactyl-nodejs-generic.json)

Import the egg, then add this image to its Docker Images list (Pterodactyl: Nests → egg → Docker Images, one per line as `Nodejs 24.15.0|ghcr.io/...`; Pelican: Eggs → egg → Docker Images):

```
ghcr.io/<YOUR_GITHUB_USERNAME>/node-ptero:24.15.0
```

Then pick it as the server's image under Startup.

Make the package public on GitHub, or add registry credentials to Wings, so your nodes can pull it. The egg's startup command runs as normal, and `{{VARIABLES}}` like `{{SERVER_PORT}}` get filled in.
