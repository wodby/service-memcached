# Memcached on Wodby

What Wodby sets up for this Memcached service.

## How applications reach it

- Host: the name of this app service inside the environment. Port: `11211`, TCP only (UDP is switched off).
- There is no password or other authentication, and no token is generated. Access is limited to the environment's internal network.
- A service linked to this one receives the host and port in variables named by the linking service. Read them in code instead of hardcoding the host.

## Configuration

The image turns three settings into `memcached` command-line options at start. Change them through the service's settings, not with a command or a config file.

| Setting | Variable | Option | Default |
| --- | --- | --- | --- |
| Cache memory (MB) | `MEMCACHED_MEMORY` | `-m` | `64` |
| Worker threads | `MEMCACHED_THREADS` | `-t` | `4` |
| Maximum connections | `MEMCACHED_MAX_CONNECTIONS` | `-c` | `1024` |

`MEMCACHED_MEMORY` is the memory for cached items only; the process uses more, so the container memory limit must be higher.

## Persistence

- Nothing is stored on disk and the service has no volume. Every restart and every deployment of the service empties the cache (the service is redeployed by replacing the container, not by a rolling update).
- When the item memory is full, Memcached evicts the least recently used items. Do not keep data here that cannot be rebuilt.

## Check the result

From this service's container:

- `make check-ready -f /usr/local/bin/actions.mk` reports whether the server answers.
- `printf 'stats\r\nquit\r\n' | nc localhost 11211` shows `limit_maxbytes`, `curr_items` and `evictions`.
