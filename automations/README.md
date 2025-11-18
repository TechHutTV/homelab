## Work in Progress
I'll be working on building out this page with all my automations. Until then subscribe to the channel so you don't miss it!

## n8n compose.yml
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
