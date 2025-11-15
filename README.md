# ansible_homelab_setup

## Bring up Docker compose stacks
Run `ansible-playbook -i inventory.ini main.yml` to install packages and deploy compose stacks. The playbook:
- Installs Docker and required packages (via install_packages.yml),
- Starts Docker and copies your compose files to `/home/compose_file` (via configure_docker.yml),
- Runs `docker compose -f <file> up -d` for each stack.

Notes:
- Edit `files/n8n-docker-compose.yml` to set N8N_HOST and WEBHOOK_URL to your VM's public IP or domain before running the playbook for webhook functionality.
- If your Docker packages aren't available from apt by default, ensure the Docker apt repo is configured or use the OS specific Docker install tasks.
- The configure_docker playbook will stop and remove all containers and run `docker system prune -af --volumes` before creating the compose projects. This will remove images, networks, and volumes — use with caution if you have persistent data you want to keep.
