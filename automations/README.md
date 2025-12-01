## Work in Progress
I'll be working on building out this page with all my automations. Until then subscribe to the channel so you don't miss it!

## Installing n8n with PM2 on Ubuntu

This guide walks through setting up n8n as a persistent background service using PM2 on Ubuntu.

### Why PM2?

PM2 is a production-ready process manager for Node.js applications. Running n8n directly in a terminal means it stops when you close the session. PM2 solves this by:

- Keeping n8n running in the background
- Automatically restarting it if it crashes
- Starting n8n on system boot
- Managing environment variables cleanly
- Providing monitoring and log management

### Prerequisites

Ensure Node.js (v18.17+) is installed:

```bash
node -v
```

If not installed, add the NodeSource repository and install:

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

### Step 1: Install or Update n8n

Install n8n globally via npm:

```bash
npm install -g n8n
```

To update an existing installation to the latest version:

```bash
npm update -g n8n
```

Verify the installation:

```bash
n8n --version
```

### Step 2: Install PM2

Install PM2 globally:

```bash
npm install -g pm2
```

### Step 3: Create the PM2 Configuration File

Create a configuration file in your home directory. This file tells PM2 how to run n8n and what environment variables to use.

```bash
nano ~/n8n.config.js
```

Add the following content:

```javascript
module.exports = {
  apps: [{
    name: 'n8n',
    script: 'n8n',
    env: {
      NODE_ENV: 'production',
      N8N_RELEASE_TYPE: 'stable',
      N8N_HOST: '0.0.0.0',
      N8N_PORT: '5678',
      WEBHOOK_URL: 'https://your-domain.com/',
    }
  }]
};
```

Save and exit (Ctrl+X, then Y, then Enter).

#### Environment Variables Explained

| Variable | Purpose |
|----------|---------|
| `NODE_ENV` | Set to `production` to remove the [DEV] tag in the browser and enable production optimizations |
| `N8N_HOST` | The IP address n8n listens on. Use `0.0.0.0` to accept connections from any interface, or a specific IP to restrict access |
| `N8N_PORT` | The port n8n runs on (default: 5678) |
| `WEBHOOK_URL` | The public URL for webhooks. Required for triggers and external integrations to reach your instance |

### Step 4: Start n8n with PM2

Launch n8n using the configuration file:

```bash
pm2 start ~/n8n.config.js
```

### Step 5: Enable Startup on Boot

Generate the startup script and save the current process list:

```bash
pm2 startup
```

This outputs a command you need to run with sudo—copy and execute it. Then save the process list:

```bash
pm2 save
```

Now n8n will automatically start when your system boots.

### PM2 Command Reference

#### Basic Process Management

| Command | Description |
|---------|-------------|
| `pm2 start ~/n8n.config.js` | Start n8n using the config file |
| `pm2 stop n8n` | Stop the n8n process |
| `pm2 restart n8n` | Restart n8n |
| `pm2 delete n8n` | Remove n8n from PM2's process list |

#### Monitoring and Logs

| Command | Description |
|---------|-------------|
| `pm2 list` | Show all managed processes and their status |
| `pm2 monit` | Real-time monitoring dashboard (CPU, memory, logs) |
| `pm2 logs n8n` | View n8n logs (Ctrl+C to exit) |
| `pm2 logs n8n --lines 100` | View last 100 lines of logs |

#### Configuration Changes

When you modify `n8n.config.js`, apply the changes with:

```bash
pm2 delete n8n
pm2 start ~/n8n.config.js
pm2 save
```

Alternatively, for environment variable updates only:

```bash
pm2 restart n8n --update-env
```

#### Useful Diagnostics

| Command | Description |
|---------|-------------|
| `pm2 show n8n` | Detailed information about the n8n process |
| `pm2 env 0` | Show environment variables for process ID 0 |

### Where is My Data Stored?

Your n8n data (workflows, credentials, settings) is stored separately from PM2 in:

```
~/.n8n/
```

Deleting and recreating the PM2 process does not affect your workflows or credentials.

## Updating n8n

To update n8n to the latest version:

```bash
pm2 stop n8n
npm update -g n8n
pm2 start n8n
```

Verify the new version:

```bash
n8n --version
```

# n8n compose.yml (switching away from)
This is running without ssl behind NetBird. Considering setting up ssl if not useing wireguard.
```
services:
    n8n:
        stdin_open: true
        tty: true
        container_name: n8n
        ports:
            - 5678:5678
        environment:
            - GENERIC_TIMEZONE=America/Los_Angeles
            - TZ=America/Los_Angeles
            - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
            - N8N_RUNNERS_ENABLED=true
            - N8N_SECURE_COOKIE=false
        volumes:
            - n8n_data:/home/node/.n8n
        image: docker.n8n.io/n8nio/n8n
volumes:
    n8n_data:
        name: n8n_data
```
