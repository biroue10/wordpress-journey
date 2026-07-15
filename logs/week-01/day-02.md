# Day 02 — 2026-07-15

**Week 01 · Focus:** Standing up a local WordPress on the headless RHEL box (Docker + WP-CLI)

## What I did
- Built a local WordPress dev environment with **Docker Compose**: two containers
  (`wordpress:php8.3-apache` + `mariadb:11`) on a private network, data in named
  volumes. Recipe committed at `projects/local-wp/docker-compose.yml`.
- Kept DB secrets in a git-ignored `.env`; verified with `git check-ignore`.
- Completed the WordPress install **entirely from the command line** with WP-CLI
  (`wp core install`) — no browser wizard.
- Verified end to end: homepage returns `HTTP 200`, `wp core is-installed` = YES,
  running WordPress 7.0.1, reachable over Tailscale on port 8090.

## What I learned
- A WordPress site = **two moving parts** (the PHP app + a database), and in Docker
  each is its own container. Containers find each other by **service name** (`db`).
- `docker compose config` validates a recipe (with `.env` substitution) before running.
- Named volumes keep site files/DB out of the git repo — commit the *recipe*, not the data.
- WP-CLI is run against a Dockerized site via a throwaway `wordpress:cli` container that
  shares the app's volume (`--volumes-from`) and network.

## Blockers / open questions (and how I diagnosed them)
- **Port 8080 conflict.** First `docker compose up` failed; `wp_app` had no published
  port. Diagnosed by `ss -ltn` (something on `*:8080`) and comparing the HTTP `Server:`
  header — the responder was `Apache (RHEL)` = the host's native httpd, not my container
  (`Apache (Debian)`). **Fix:** publish on port **8090** instead.
- **`wp core install` → "database server at `mysql`".** The CLI container didn't inherit
  `WORDPRESS_DB_HOST`, so `wp-config-docker.php` fell back to the default host `mysql`.
  **Fix:** pass the DB env vars into the CLI container.

## Links
- WP-CLI `core install`: https://developer.wordpress.org/cli/commands/core/install/
- Docker WordPress image docs: https://hub.docker.com/_/wordpress

## Tomorrow
- Log into `/wp-admin`, tour the dashboard, and explore themes (Week 1 next step).
