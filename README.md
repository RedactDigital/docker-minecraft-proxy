# Minecraft Velocity Proxy
### Before starting the server, you need to create a `velocity.toml` file. You can use the example below as a template.
### Then mount the `velocity.toml` file into the container at `/velocity/velocity.toml`.

I'd recommend also creating a volume for the `/velocity` directory so it persists between container restarts. You can also mount the `/velocity` directory to your host machine if you prefer.

## Example `docker-compose.yml`
```yaml
version: '3.9'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: minecraft-proxy
    container_name: minecraft-proxy
    restart: unless-stopped
    ports:
      - '25577:25577'
    volumes:
      - minecraft-velocity:/velocity
      - ./velocity.toml:/velocity/velocity.toml

volumes:
  minecraft-velocity:
```

## Example `docker run` command
```bash
docker run -d \
  --name=minecraft-proxy \
  -p 25577:25577 \
  -v minecraft-velocity:/velocity \
  -v ./velocity.toml:/velocity/velocity.toml \
  --restart unless-stopped \
  minecraft-proxy
```

## Example `velocity.toml`
```toml
# Config version. Do not change this
config-version = "2.6"

# What port should the proxy be bound to? By default, we'll bind to all addresses on port 25577.
bind = "0.0.0.0:25577"

# What should be the MOTD? This gets displayed when the player adds your server to
# their server list. Only MiniMessage format is accepted.
motd = "<#09add3>A Velocity Server"

# What should we display for the maximum number of players? (Velocity does not support a cap
# on the number of players online.)
show-max-players = 500

# Should we authenticate players with Mojang? By default, this is on.
online-mode = true

# Should the proxy enforce the new public key security standard? By default, this is on.
force-key-authentication = true

# If client's ISP/AS sent from this proxy is different from the one from Mojang's
# authentication server, the player is kicked. This disallows some VPN and proxy
# connections but is a weak form of protection.
prevent-client-proxy-connections = false

# Should we forward IP addresses and other data to backend servers?
# Available options:
# - "none":        No forwarding will be done. All players will appear to be connecting
#                  from the proxy and will have offline-mode UUIDs.
# - "legacy":      Forward player IPs and UUIDs in a BungeeCord-compatible format. Use this
#                  if you run servers using Minecraft 1.12 or lower.
# - "bungeeguard": Forward player IPs and UUIDs in a format supported by the BungeeGuard
#                  plugin. Use this if you run servers using Minecraft 1.12 or lower, and are
#                  unable to implement network level firewalling (on a shared host).
# - "modern":      Forward player IPs and UUIDs as part of the login process using
#                  Velocity's native forwarding. Only applicable for Minecraft 1.13 or higher.
player-info-forwarding-mode = "NONE"

# If you are using modern or BungeeGuard IP forwarding, configure a file that contains a unique secret here.
# The file is expected to be UTF-8 encoded and not empty.
forwarding-secret-file = "forwarding.secret"

# Announce whether or not your server supports Forge. If you run a modded server, we
# suggest turning this on.
#
# If your network runs one modpack consistently, consider using ping-passthrough = "mods"
# instead for a nicer display in the server list.
announce-forge = false

# If enabled (default is false) and the proxy is in online mode, Velocity will kick
# any existing player who is online if a duplicate connection attempt is made.
kick-existing-players = false

# Should Velocity pass server list ping requests to a backend server?
# Available options:
# - "disabled":    No pass-through will be done. The velocity.toml and server-icon.png
#                  will determine the initial server list ping response.
# - "mods":        Passes only the mod list from your backend server into the response.
#                  The first server in your try list (or forced host) with a mod list will be
#                  used. If no backend servers can be contacted, Velocity won't display any
#                  mod information.
# - "description": Uses the description and mod list from the backend server. The first
#                  server in the try (or forced host) list that responds is used for the
#                  description and mod list.
# - "all":         Uses the backend server's response as the proxy response. The Velocity
#                  configuration is used if no servers could be contacted.
ping-passthrough = "DISABLED"

# If not enabled (default is true) player IP addresses will be replaced by <ip address withheld> in logs
enable-player-address-logging = true

[servers]
# Configure your servers here. Each key represents the server's name, and the value
# represents the IP address of the server to connect to.
lobby = "127.0.0.1:30066"
factions = "127.0.0.1:30067"
minigames = "127.0.0.1:30068"

# In what order we should try servers when a player logs in or is kicked from a server.
try = [
    "lobby"
]

[forced-hosts]
# Configure your forced hosts here.
"lobby.example.com" = [
    "lobby"
]
"factions.example.com" = [
    "factions"
]
"minigames.example.com" = [
    "minigames"
]

[advanced]
# How large a Minecraft packet has to be before we compress it. Setting this to zero will
# compress all packets, and setting it to -1 will disable compression entirely.
compression-threshold = 256

# How much compression should be done (from 0-9). The default is -1, which uses the
# default level of 6.
compression-level = -1

# How fast (in milliseconds) are clients allowed to connect after the last connection? By
# default, this is three seconds. Disable this by setting this to 0.
login-ratelimit = 3000

# Specify a custom timeout for connection timeouts here. The default is five seconds.
connection-timeout = 5000

# Specify a read timeout for connections here. The default is 30 seconds.
read-timeout = 30000

# Enables compatibility with HAProxy's PROXY protocol. If you don't know what this is for, then
# don't enable it.
haproxy-protocol = false

# Enables TCP fast open support on the proxy. Requires the proxy to run on Linux.
tcp-fast-open = false

# Enables BungeeCord plugin messaging channel support on Velocity.
bungee-plugin-message-channel = true

# Shows ping requests to the proxy from clients.
show-ping-requests = false

# By default, Velocity will attempt to gracefully handle situations where the user unexpectedly
# loses connection to the server without an explicit disconnect message by attempting to fall the
# user back, except in the case of read timeouts. BungeeCord will disconnect the user instead. You
# can disable this setting to use the BungeeCord behavior.
failover-on-unexpected-server-disconnect = true

# Declares the proxy commands to 1.13+ clients.
announce-proxy-commands = true

# Enables the logging of commands
log-command-executions = false

# Enables logging of player connections when connecting to the proxy, switching servers
# and disconnecting from the proxy.
log-player-connections = true

[query]
# Whether to enable responding to GameSpy 4 query responses or not.
enabled = false

# If query is enabled, on what port should the query protocol listen on?
port = 25577

# This is the map name that is reported to the query services.
map = "Velocity"

# Whether plugins should be shown in query response by default or not
show-plugins = false
```
