# Immich - Self-Hosted Photo and Video Backup

Immich is a self-hosted alternative to Google Photos with a real focus on privacy. It keeps your photos and videos on your own hardware with full support for RAW files, facial recognition, and automatic organization. The mobile app for both [Android](https://play.google.com/store/apps/details?id=app.immich) and [iOS](https://apps.apple.com/app/immich/id1613945652) handles auto-backup out of the box, which is honestly the killer feature.

> [!NOTE]
> Most of the steps below are based on their [official docs](https://docs.immich.app/install/docker-compose). I'd give those a read too, especially as things change between versions.

## Prerequisites
1. A server running Docker. Check out our [docker guide](https://techhut.tv/7-docker-basics-for-beginners/) if you're new to it.
2. At least 6GB of RAM and a 4-core CPU is recommended ([learn more](https://immich.app/docs/install/requirements)).
3. Enough storage for your media. This really depends on your collection. I'd plan for at least double what you currently have to leave room for growth.

## Setup

### Prepare Your Environment
Create a directory for Immich. If your root filesystem is small you'll probably want to point `library` at a network drive or external storage. Check out [this guide on auto-mounting drives](https://techhut.tv/auto-mount-drives-in-linux-fstab/) in Linux.

```bash
mkdir -p ~/docker/immich
cd ~/docker/immich
```

Now go ahead and grab `compose.yaml` and the safe `.env.example` template from this repo. Copy the template to `.env`, restrict its permissions, and then edit your local values.

```bash
wget https://github.com/TechHutTV/homelab/raw/refs/heads/main/cloud/immich/compose.yaml
wget https://github.com/TechHutTV/homelab/raw/refs/heads/main/cloud/immich/.env.example
cp .env.example .env
chmod 600 .env
```

### Configure the .env File

```bash
nano .env
```

**You have to change these:**
- `DB_PASSWORD` — Set this to a random strong password before deploying. Stick to `A-Za-z0-9` so Docker doesn't choke on special characters. The placeholder is syntactically valid and does not prevent startup by itself.
- `TZ` — Set your [timezone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List).
- `UPLOAD_LOCATION` — Where your photos and videos will live. Network shares are fine here.
- `DB_DATA_LOCATION` — Where the Postgres database lives. Do **not** put this on a network share, Immich will not be happy.

### Deploy the Stack
```bash
docker compose up -d
```

Give it a few minutes to pull images and initialize. If something fails to start, check the logs:
```bash
docker logs immich_server
```

### Access Immich
1. Open `http://your-server-ip:2283` in your browser.
2. Create your admin account. The first user to register becomes admin, so do this before sharing the URL with anyone.
3. Configure the basics:
   - **Storage Template** — Make your library [human readable](https://immich.app/docs/install/post-install) so backups outside Immich are useful.
   - **Machine Learning** — Turn on facial recognition and smart search if you want it.

And there we go, you should have Immich running.

## Mobile App Setup
This is where Immich really shines. Auto-backup from your phone is the whole reason most people self-host this.

1. Install Immich from the [App Store](https://apps.apple.com/app/immich/id1613945652) or [Play Store](https://play.google.com/store/apps/details?id=app.immich).
2. Connect to your server:
   - Server URL: `http://your-server-ip:2283`
   - Log in with the admin account you just created (or a regular user account if you set one up).
3. Head into settings and turn on auto-backup. Pick your photo and video albums, set the backup behavior (foreground, background, on Wi-Fi only, and whatnot), and you're good to go.

## Remote Access
Hosting Immich at home is great, but you'll probably want to reach it from outside your network for backups while traveling.

- **[NetBird](https://netbird.io/) (Recommended)** — A zero-trust mesh network built on WireGuard. No port forwarding, no exposing services to the open internet. Follow our [NetBird setup guide](https://techhut.tv/self-host-netbird-pocketid) to get going.
- **Reverse Proxy** — If you want a clean URL like `photos.yourdomain.com`, set up NGINX Proxy Manager ([video guide](https://www.youtube.com/watch?v=79e6KBYcVmQ)). Forward your hostname to `http://immich-server:2283` and you're set.

## Optional Configuration

### Hardware Transcoding
For Intel QuickSync, uncomment the `devices` section in the `immich-server` service in `compose.yaml`. This speeds up video transcoding significantly. [Learn more](https://immich.app/docs/features/hardware-transcoding/#single-compose-file).

```yaml
devices:
  - /dev/dri:/dev/dri
```

### Hardware-Accelerated Machine Learning
The `compose.yaml` has commented blocks for Intel OpenVINO and Google Coral. Pick the one matching your accelerator, uncomment, and recompose. For NVIDIA, AMD, ARM NN, or RKNN check the [full hardware ML docs](https://immich.app/docs/features/ml-hardware-acceleration).

If you don't have any hardware acceleration, that's fine. ML runs on CPU by default and works well enough for most homelab libraries.

### External Libraries
External libraries let Immich index existing photo directories without copying them into its managed storage. Super useful if you already have a sorted library on disk and don't want to import everything.

To use them, mount the directory into the `immich-server` container so it can read it:
```yaml
immich-server:
  volumes:
    - ${UPLOAD_LOCATION}:/data
    - /mnt/photos:/mnt/photos:ro  # read-only is safer for existing libraries
    - /etc/localtime:/etc/localtime:ro
```

Then in the Immich web UI go to _Administration > Libraries_, create a new External Library for the admin user, and add `/mnt/photos` as the import path. Do note that the path is the path **inside the container**, not your host. [Learn more](https://immich.app/docs/features/libraries/).

### Disable Machine Learning
If you don't want facial recognition or smart search, you can drop the whole `immich-machine-learning` service from `compose.yaml`. This saves a few GB of RAM, which is nice on a smaller server.

### Bulk Uploading with the CLI
If you've got a giant existing library you want to import (rather than mount as external), the [Immich CLI](https://immich.app/docs/features/command-line-interface) is the way to go. It's way faster than dragging files into the web UI and handles huge batches without timing out.

Install it on your desktop or any machine that can reach the server:
```bash
npm install -g @immich/cli
```

Then log in and upload:
```bash
immich login http://your-server-ip:2283 your-api-key
immich upload --recursive /path/to/your/photos
```

You can grab an API key from _Account Settings > API Keys_ in the web UI.

## Storage Templates
Storage templates automatically sort photos into folders based on metadata like date or camera model. This is what makes your library actually browsable outside Immich, which matters a lot if you're ever doing manual backups or migrating.

A solid default:
```
{{y}}/{{MMMM}}-{{DD}}/{{filename}}
```
Generates: `2024/July-15/IMG_1234.jpg`

Go to _Settings > Storage Template_ in the web UI to set this up. You can also re-run it on existing libraries, which is great if you change your mind on the structure later.

## Backup Strategy

Immich needs **both its database and original files**. Its automatic database dumps are stored in `UPLOAD_LOCATION/backups`; configure their schedule and retention under _Administration > Settings > Backup_. These dumps contain no photos or videos and still need copying off the server. [Official backup and restore guide](https://docs.immich.app/administration/backup-and-restore/).

For a matching database/file backup, stop `immich-server`, leave PostgreSQL running, create a successful `pg_dump` using your configured database name/user, then back up that dump and all of `UPLOAD_LOCATION` before restarting the server. Include external-library source folders, `compose.yaml`, and your protected `.env`. Use the [upstream backup script](https://docs.immich.app/guides/template-backup-script/) as a starting point for automation. **Do not copy a running `DB_DATA_LOCATION` as a file backup.**

Follow the shared [schedule, encryption, retention, and alerting plan](../../storage/README.md#backups). Rehearse the official restore flow on a separate instance with compatible versions, matching file paths, and the database backup. Open several photos/videos and check users/albums before relying on it.

## Troubleshooting

**Port Conflicts** — Make sure port 2283 is free, or remap with `IMMICH_PORT` in your `.env` (e.g. `IMMICH_PORT=8283`).

**Permission Errors** — Check ownership on your `UPLOAD_LOCATION` and `DB_DATA_LOCATION` directories with `ls -l`. The Immich containers run as their own user inside.

**ML Container Failing** — Usually a RAM issue. Either give the host more memory or temporarily disable ML by removing the `immich-machine-learning` service.

**Reset Admin Password:**
```bash
docker exec immich_server npm run reset-admin-password
```

## Maintenance

### Updating
To pull the latest images and recreate the containers:
```bash
docker compose pull && docker compose up -d
```

The `.env.example` template sets `IMMICH_VERSION=v3`, which tracks stable v3 minor and patch releases after you copy it to `.env`. If you want to pin an exact version for stability, use a full tag such as `v3.0.0` instead.

If you're upgrading an existing v2 installation, update your mobile apps and back up your database and library first. This repo already uses Immich's VectorChord PostgreSQL image, so you do not need to change the database image. Change `IMMICH_VERSION` from `v2` to `v3` in your local `.env`, then run the update commands above. If your existing deployment still uses pgvecto.rs, complete the official [VectorChord migration](https://docs.immich.app/install/upgrading/#migrating-to-vectorchord) before starting v3. Check the full [v3 migration guide](https://immich.app/blog/v3-migration) for removed environment variables and other integration changes.

### Housekeeping
- Watch your `UPLOAD_LOCATION` for storage growth. The mobile app will quietly fill it up.
- Set retention policies in _Administration > Settings > Trash_ if you want auto-cleanup.
- Run _Administration > Jobs > External Library Scan_ periodically if you're using external libraries that change.

With all that, you should have a pretty solid Immich setup. I do hope this helps, have a great one.
