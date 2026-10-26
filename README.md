# MaxAuth

MaxAuth is a proxy-first authentication and identity plugin for Minecraft networks running behind Velocity or BungeeCord.

It protects cracked accounts with passwords, verifies premium Java players through the current connection, recognizes Bedrock players through Floodgate API proof, assigns stable backend UUIDs, and keeps unauthenticated players isolated from protected servers.

MaxAuth is a modern successor to JPremium. Version 1.0 replaces the earlier MaxAuth BETA authentication model with explicit account types, connection-bound identity proof, signed proxy/backend state, and a separate migration tool.

## Download and documentation

- [Download MaxAuth on SpigotMC](https://www.spigotmc.org/resources/135024/)
- [English wiki](https://github.com/djnejk/MaxAuth_public/wiki)
- [Česká wiki](https://github.com/djnejk/MaxAuth_public/wiki/MaxAuth-Wiki-(CZ))
- [Authentication Flow](https://github.com/djnejk/MaxAuth_public/wiki/Authentication-Flow)
- [Authentication Recovery](https://github.com/djnejk/MaxAuth_public/wiki/Authentication-Recovery)
- [Data Migration](https://github.com/djnejk/MaxAuth_public/wiki/Data-Migration)
- [Runtime Libraries](https://github.com/djnejk/MaxAuth_public/wiki/Runtime-Libraries)
- [Issues and feature requests](https://github.com/djnejk/MaxAuth_public/issues)

## Architecture

The same MaxAuth JAR is installed in two places:

- **Proxy:** owns account detection, authentication, storage, sessions, routing, commands, e-mail recovery, 2FA, and audit logging.
- **Backend:** validates signed proxy state, protects unauthenticated players, renders captcha maps, and applies the configured authentication environment.

Every backend that accepts players through the network should run MaxAuth. The proxy and all backends must use the same private `accessToken`, unless the documented forwarding setup explicitly disables only the handshake-token check. Signed MaxAuth state messages remain mandatory.

## Security model

- A nickname lookup or client-supplied UUID never authenticates a premium player.
- Premium Java access requires a matching Mojang online-mode proof from the current connection.
- Floodgate access requires connection-specific Floodgate API proof; nickname prefixes and UUID shapes are not trusted.
- Existing accounts always follow their persisted type before any new-account discovery is attempted.
- Stable backend UUIDs are stored separately from Mojang and Floodgate identities.
- Zero, malformed, conflicting, expired, unsigned, replayed, or wrong-player identity state fails closed.
- A Mojang-owned nickname used by a cracked launcher is rejected with a MaxAuth explanation instead of Minecraft's generic session error. An administrator can intentionally authorize one cracked-name registration with `/maxauth allowcrackedname <nickname>`.
- Unauthenticated players can reach only configured limbo servers.

Read [Authentication Flow](https://github.com/djnejk/MaxAuth_public/wiki/Authentication-Flow) for the complete decision process.

## Main features

- Velocity and BungeeCord proxy support.
- Paper, Purpur, Spigot, and Folia backend support.
- Premium Java, cracked Java, Floodgate, and linked Bedrock/Java accounts.
- Password registration and login with Argon2id, bcrypt, PBKDF2, and compatible legacy verification hashes.
- Automatic password rehash after a successful login.
- Manual and automatic sessions with address-rate limits.
- E-mail password recovery with SMTP diagnostics and HTML templates.
- TOTP two-factor authentication.
- Captcha maps with case-insensitive code entry.
- Global and player-controlled last-server redirection.
- Safe `/premium` and `/cracked` conversion flows.
- Risky-command confirmation with clickable `/confirm` messages.
- Configurable limbo/main fallbacks and clean handling when every target is unavailable.
- Signed, expiring, replay-protected proxy/backend state.
- Trusted proxy IP/CIDR restrictions and access-token rotation.
- Discord audit webhooks, protected accounts, storage audits, and UUID repair tooling.
- Automatic configuration/message key updates.
- Runtime dependency downloading with SHA-256 verification and storage-driver selection.

## Requirements

- Java 17 or newer.
- Velocity 3.x or official BungeeCord.
- Paper, Purpur, Spigot, or Folia backend servers.
- SQLite, MySQL, MariaDB, or PostgreSQL.
- Outbound HTTPS on the first configured start so runtime libraries can be downloaded and verified.
- Floodgate on the proxy when Bedrock support is enabled.

Official BungeeCord 1.21 build 2104 is the release-tested baseline. Waterfall is not currently release-validated. BungeeGuard 1.4.0 or newer is supported when installed on both the proxy and backend.

## Quick installation

1. Download MaxAuth from [SpigotMC](https://www.spigotmc.org/resources/135024/).
2. Put the same JAR into the proxy plugin directory and every protected backend plugin directory.
3. Start every node once to generate its files and runtime-library cache.
4. Generate a private random `accessToken` of at least 32 UTF-8 bytes and configure the same value everywhere.
5. Configure `limboServerNames`, `mainServerNames`, storage, and optional Floodgate support on the proxy.
6. Configure forwarding, firewall rules, and `trustedProxyIps` as appropriate for the network.
7. Restart the complete network and test cracked, premium, and—when enabled—Bedrock logins before opening it to players.

MaxAuth deliberately stops an unsafe or incomplete installation instead of letting players through without authentication protection.

### Velocity

Use Velocity modern forwarding and configure the forwarding secret normally. Backend servers must not be publicly reachable. MaxAuth detects modern forwarding and does not install an unnecessary legacy handshake listener.

### BungeeCord

Set the following values in BungeeCord `config.yml`:

```yml
online_mode: false
ip_forward: true
```

MaxAuth chooses online or offline verification for each connection. A globally online BungeeCord cannot admit both cracked and verified identities correctly, so MaxAuth refuses unsafe global settings. Backends must use offline mode with Bungee forwarding and must accept traffic only from the proxy.

Without BungeeGuard, use the same MaxAuth `accessToken` everywhere. With BungeeGuard 1.4.0 or newer installed on both sides, BungeeGuard owns handshake admission while MaxAuth continues to sign its own backend-state channel. Sensitive MaxAuth command arguments are redacted from Bungee's command log.

## Minimal proxy configuration

```yml
storageType: MYSQL
storageTablePrefix: 'maxauth_'
storageTableName: ''

accessToken: 'replace-with-a-private-random-token-of-at-least-32-bytes'

limboServerNames: [limbo]
mainServerNames: [lobby]

uniqueIdsType: REAL
registerPremiumUsers: true
premiumNamePolicy: REQUIRE_ONLINE_PROOF

floodgateSupport:
  enabled: false
```

## Minimal backend configuration

```yml
accessToken: 'replace-with-the-exact-same-private-token'
previousAccessToken: ''
accessTokenDisabled: false
trustedProxyIps: ['127.0.0.1']

restrictedMovement: true
restrictedInteractions: true
stateMessageTimeoutSeconds: 10
```

The generated files contain comments for every setting. The full reference is maintained in the [English wiki](https://github.com/djnejk/MaxAuth_public/wiki).

## Data migration

The production plugin does not contain MaxAuth BETA or JPremium row importers. Conversion is handled by the separate **MaxAuth Migration Tool** described in [Data Migration](https://github.com/djnejk/MaxAuth_public/wiki/Data-Migration).

Run migration only while the network is closed, keep a restorable backup, rehearse against staging first, and validate the result with `/maxauth authaudit`. The old `storageTableName` option is only a legacy-configuration detector. Any non-empty value blocks startup before the new storage opens; after verified migration it must be `storageTableName: ''`.

Read [Authentication Recovery](https://github.com/djnejk/MaxAuth_public/wiki/Authentication-Recovery) before upgrading a production network.

## Runtime libraries

MaxAuth keeps downloaded dependencies in its plugin data directory under `libraries`. MaxAuth-managed artifacts are downloaded over HTTPS, checked against pinned SHA-256 hashes, and loaded before storage starts; BungeeCord resolves its descriptor-managed common libraries through the platform. Only the JDBC driver selected by `storageType` is loaded. Changing the storage type requires a proxy restart so the matching driver can be verified or downloaded.

The plugin JAR does not need to be replaced when only a cached dependency is missing. Read [Runtime Libraries](https://github.com/djnejk/MaxAuth_public/wiki/Runtime-Libraries) for repositories, cache behavior, offline preparation, and troubleshooting.

## Common commands

Player commands include:

```text
/register, /login, /confirm
/startsession, /destroysession
/changepassword, /createpassword, /unregister
/premium, /cracked
/changeemailaddress, /requestpasswordrecovery, /confirmpasswordrecovery
/requestsecondfactor, /activatesecondfactor, /deactivatesecondfactor
/maxauth lastserver
```

Administrative operations are available under `/maxauth` and `/mauth`, including configuration reload/checks, account inspection and mutation, session/password/e-mail/2FA management, storage audit, pending-claim recovery, UUID repair, cracked-name exceptions, SMTP testing, and backend spawn setup.

## Support

Before reporting an authentication problem, include:

- proxy type and exact build;
- backend type and exact build;
- Java version;
- MaxAuth version on proxy and backend;
- Geyser, Floodgate, and BungeeGuard versions when used;
- storage type;
- relevant startup and connection logs with secrets removed;
- `/maxauth configcheck`, `/maxauth checkflow`, or `/maxauth authaudit` output when applicable.

Report bugs and feature requests through [MaxAuth_public issues](https://github.com/djnejk/MaxAuth_public/issues).

## Author

MaxAuth is developed by [DjDevs.eu](https://djdevs.eu/).
