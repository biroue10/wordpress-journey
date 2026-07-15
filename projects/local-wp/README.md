# Local WordPress (Docker)

A local WordPress development environment running on my headless RHEL box,
built with Docker Compose. This is the foundation site I use throughout the
journey for hands-on practice.

## Stack

| Component | Image | Notes |
|-----------|-------|-------|
| WordPress | `wordpress:php8.3-apache` | PHP 8.3 + Apache, the app itself |
| Database  | `mariadb:11` | Stores posts, users, options |

Two containers on a private Docker network (`wpnet`). Data persists in **named
Docker volumes** (`wp_data`, `db_data`) so nothing large or secret lands in git.

## Architecture

```
http://<tailscale-ip>:8090  ─▶  wordpress (PHP+Apache)  ─▶  db (MariaDB)
                                 vol: wp_data                vol: db_data
```

The WordPress container reaches the database using the hostname `db` — the
Compose **service name**, resolved over the private network.

## Usage

```bash
# Start (from this folder)
docker compose up -d

# Stop (keeps data)
docker compose down

# Stop AND wipe all data (fresh start)
docker compose down -v

# Logs
docker compose logs -f wordpress

# Run WP-CLI against the running site (throwaway container).
# NOTE: the DB env vars must be passed in, otherwise wp-config falls back
# to the default host 'mysql' and the connection fails.
set -a; . ./.env; set +a
docker run --rm --network local-wp_wpnet --volumes-from wp_app \
  -e WORDPRESS_DB_HOST="$WORDPRESS_DB_HOST" \
  -e WORDPRESS_DB_USER="$MYSQL_USER" \
  -e WORDPRESS_DB_PASSWORD="$MYSQL_PASSWORD" \
  -e WORDPRESS_DB_NAME="$MYSQL_DATABASE" \
  -w /var/www/html wordpress:cli wp <command>
```

## Configuration

Secrets live in `.env` (git-ignored — **never committed**). To recreate it:

```
MYSQL_ROOT_PASSWORD=<random>
MYSQL_DATABASE=wordpress
MYSQL_USER=wp_user
MYSQL_PASSWORD=<random>
WORDPRESS_DB_HOST=db:3306
```

## Gotchas hit while building this (Day 02)

- **Port 8080 was already taken** by the host's native RHEL Apache/httpd, so the
  container's port publish collided. Diagnosed with `ss -ltn` + comparing the
  `Server:` header (`RHEL` httpd vs. the container's `Debian` Apache). Fixed by
  publishing on **8090** instead.
- **`wp core install` failed with "database server at `mysql`"** — the throwaway
  WP-CLI container didn't inherit `WORDPRESS_DB_HOST`, so `wp-config-docker.php`
  fell back to its default host `mysql`. Fixed by passing the DB env vars in.
- Access works over Tailscale because the `tailscale0` interface is in
  firewalld's **trusted** zone.

## Access

- URL: `http://<tailscale-ip>:8090`
- Admin: `http://<tailscale-ip>:8090/wp-admin`
- Credentials: stored in my password manager (not in this repo).
