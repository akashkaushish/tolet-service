# Tolet Service

Containerized development stack:

- Next.js on <http://localhost:3000>
- Symfony 7 behind Nginx on <http://localhost:8080>
- PHP 8.3 FPM with Composer, ZIP, and PDO MySQL
- MariaDB on `localhost:3306`

No host installation of PHP or Composer is required.

## 1. Build and start the empty development stack

Run from PowerShell in the repository root:

```powershell
docker compose build
docker compose up -d
docker compose ps
```

The frontend container deliberately stays idle until a `package.json` exists.
Nginx may return `404` or `502` until Symfony has been scaffolded.

## 2. Scaffold Next.js

Run the generator inside the active frontend container:

```powershell
docker compose exec frontend sh -lc "rm -f .gitkeep && npx create-next-app@latest . --typescript --eslint --tailwind --app --src-dir --import-alias '@/*' --use-npm --yes"
```

Restart the frontend service so its startup command detects the new application:

```powershell
docker compose restart frontend
```

## 3. Scaffold Symfony 7

Run Composer inside the active PHP container:

```powershell
docker compose exec php sh -lc "rm -f .gitkeep && composer create-project symfony/skeleton:'7.4.*' ."
```

The Symfony skeleton intentionally has no home-page controller, so a Symfony
`404` response at <http://localhost:8080> confirms that Nginx and PHP-FPM are
communicating correctly.

If the project needs Doctrine, install it from the PHP container:

```powershell
docker compose exec php composer require symfony/orm-pack
```

The `DATABASE_URL` passed by Compose already points to the MariaDB container.

## Useful commands

```powershell
docker compose logs -f
docker compose ps
docker compose down
docker compose down -v
```

`docker compose down` preserves database data. The `-v` variant permanently
deletes the MariaDB volume and should only be used when a full database reset is
intended.
