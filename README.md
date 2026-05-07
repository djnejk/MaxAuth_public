# MaxAuth

MaxAuth is a proxy-first authentication plugin for Minecraft networks.

It protects cracked accounts with passwords, supports premium players, manages UUIDs, sessions, two-factor authentication, captcha maps, backend protection, and safe migration from JPremium.

MaxAuth is designed as a modern successor to JPremium. It keeps the core authentication ideas and migration path familiar for existing JPremium networks, while adding MaxAuth-specific configuration, messages, backend protections, diagnostics, and audit tooling.

## Documentation

Full documentation is available on the GitHub wiki:

- English wiki: https://github.com/djnejk/MaxAuth_public/wiki
- Czech wiki: https://github.com/djnejk/MaxAuth_public/wiki/MaxAuth-Wiki-(CZ)

## What MaxAuth Does

MaxAuth has two parts:

- **Proxy plugin** for Velocity or BungeeCord.
- **Backend plugin** for Paper/Spigot servers behind the proxy.

The proxy plugin is the main authority. It decides whether a player is premium, cracked, or Bedrock, which UUID they should use, whether they are authenticated, where they should be redirected, and which profile is stored in the database.

The backend plugin protects players during authorization. It can restrict movement, interactions, chat, commands, and inventory, show a captcha map, lock player time/weather, move players to an auth spawn, and verify trusted proxy state updates.

## Features

- Velocity and BungeeCord proxy support.
- Paper/Spigot backend support.
- Premium and cracked account handling.
- Automatic premium player registration.
- Password registration and login.
- Optional password confirmation.
- Safe password regex and weak-password checks.
- Manual and automatic sessions.
- Email password recovery.
- Optional TOTP two-factor authentication.
- Optional captcha map for new cracked registrations.
- Limbo servers for unauthenticated players.
- Main-server redirects after `/login` and `/register`.
- Last-server redirection.
- Backend protection through access tokens and trusted proxy IPs.
- SQLite, MySQL, MariaDB, and PostgreSQL support.
- Configurable HikariCP connection pool.
- Configurable messages with chat, title, subtitle, bossbar, actionbar, and kick screen types.
- Hex colors and gradients in messages.
- Admin commands under `/maxauth` and `/mauth`.
- Discord audit webhooks.
- Automatic config/message updates for new settings.
- JPremium database migration helper.

## Requirements

- Java 17 or newer.
- Velocity 3.x or BungeeCord.
- Paper/Spigot backend servers.
- One supported database: SQLite, MySQL, MariaDB, or PostgreSQL.

Optional backend plugins:

- ProtocolLib
- BungeeGuard
- PlaceholderAPI

## Installation

1. Put the MaxAuth jar into your proxy plugins folder.
2. Put the same jar into every backend server that should be protected by MaxAuth.
3. Start the proxy and backend once to generate configuration files.
4. Set the same `accessToken` on proxy and every backend.
5. Configure `limboServerNames`, `mainServerNames`, storage, and the authentication options you want.

On first proxy startup, MaxAuth shuts the proxy down if the required access token placeholder is still present. This is intentional fail-closed behavior to prevent players from joining without proper authentication and UUID protection.

## JPremium Migration

MaxAuth includes a migration helper for networks moving from JPremium.

If the proxy detects the original JPremium SQL table `user_profiles`, it creates `migrate_DB.yml` in the MaxAuth plugin folder with a pre-migration report. After reviewing the report, the administrator can explicitly enable migration for the next restart. Existing MaxAuth accounts are preferred when collisions are found, and the old JPremium table is renamed after a successful migration.

See the wiki for the full migration flow and safety notes.

## Basic Proxy Example

```yml
storageType: MYSQL
storageTableName: maxauth_users
limboServerNames: [limbo]
mainServerNames: [lobby]
afterRegisterRedirect: true
afterLoginRedirect: true
verifyCaptchaCode: false
confirmPassword: true
registerPremiumUsers: true
```

## Basic Backend Example

```yml
accessToken: 'same-token-as-proxy'
accessTokenDisabled: false
restrictedMovement: true
restrictedInteractions: true
blindnessEffect: true
setPlayerTime: day
setPlayerWeather: clear
```

## Main Commands

Player commands:

```text
/login <password>
/register <password>
/startsession
/destroysession
/changepassword <current-password> <new-password>
/unregister <password>
/premium <password>
/cracked <password>
```

Admin commands:

```text
/maxauth
/mauth
/maxauth help
/maxauth reload
/maxauth configcheck
/maxauth checkflow
/maxauth spawnset
```

More command details are available in the wiki.

## Permissions

```text
maxauth.admin
maxauth.command.reload
```


## Author

MaxAuth by DjDevs.eu.
