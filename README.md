# kong-python-pluginserver

[![PyPI version](https://badge.fury.io/py/kong-pdk.svg)](https://badge.fury.io/py/kong-pdk)

Plugin server and PDK (Plugin Development Kit) for Python language support in Kong.

Requires Kong >= 2.3.0.

## Documentation

- [PDK API Reference](https://kong.github.io/kong-python-pdk/)
- [Examples](https://github.com/Kong/kong-python-pdk/tree/master/examples)

## Installation

Install the plugin server globally with pip:

```
pip3 install kong-pdk
```

## Plugin Development

A valid Python plugin must define:

```python
Schema = (
    { "message": { "type": "string" } },
)
version = '0.1.0'
priority = 0

class Plugin(object):
    pass
```

Each plugin includes:
- A `Plugin` class with optional phase handler methods
- A `Schema` for plugin configuration
- `version` and `priority` values

Example with an `access` handler:

```python
class Plugin(object):
    def __init__(self, config):
        self.config = config

    def access(self, kong):
        host, err = kong.request.get_header("host")
        kong.log.info(f"Host header: {host}")
```

### Supported phases

- `certificate`
- `rewrite`
- `access`
- `response`
- `preread`
- `log`

### Type Hints

For autocomplete in IDEs, you can add type hints:

```python
import kong_pdk.pdk.kong as kong

class Plugin(object):
    def __init__(self, config):
        self.config = config

    def access(self, kong: kong.kong):
        host, err = kong.request.get_header("host")
```

> ⚠️ The `kong` module is only for type hints—always use the `kong` instance passed into the phase handler.

## Embedded Plugin Server

To run the plugin with an embedded server:

```python
if __name__ == "__main__":
    from kong_pdk.cli import start_dedicated_server
    start_dedicated_server("py-hello", Plugin, version, priority)
```

## Kong Configuration

To load a Python plugin in Kong Gateway via `kong.conf`, add:

```
pluginserver_names = my-plugin

pluginserver_my_plugin_start_cmd = /path/to/my-plugin.py
pluginserver_my_plugin_query_cmd = /path/to/my-plugin.py --dump
pluginserver_my_plugin_socket = /usr/local/kong/my-plugin.socket
```

Enable verbose logging:

```
pluginserver_my_plugin_start_cmd = /path/to/my-plugin.py -v
```

## Concurrency Options

You can configure concurrency models in `kong.conf` using:

- `-g` — enables [Gevent](http://www.gevent.org/) for IO-bound workloads
- `-m` — enables multiprocessing for CPU-bound workloads

```
pluginserver_my_plugin_start_cmd = /path/to/my-plugin.py -g
```

## CLI Usage

```
usage: kong-python-pluginserver [-h] [-p prefix] [-v] [--version] [--socket-name SOCKET_NAME] [--listen-queue-size LISTEN_QUEUE_SIZE]
                         [--no-lua-style] [-m | -g] -d directory [--dump-plugin-info name] [--dump-all-plugins]

Kong Python Plugin Server.

optional arguments:
  -h, --help            show help message and exit
  -p prefix             socket path prefix (default: /usr/local/kong/)
  -v, --verbose         enable verbose logging
  --socket-name         name of the socket file
  --listen-queue-size   socket listen queue size
  --no-lua-style        switch to Python-style error handling (default: False)
  -m                    enable multiprocessing
  -g                    enable gevent
  -d directory          plugin directory path
  --dump-plugin-info    dump info for one plugin
  --dump-all-plugins    dump info for all plugins
```

## API Reference

The PDK API documentation is available here:  
📚 [https://kong.github.io/kong-python-pdk/](https://kong.github.io/kong-python-pdk/)

To regenerate it locally:

```
git worktree add docs/build/html gh-pages
cd docs
make deps && make html
```

## Deprecation Notice

In the next major release, the PDK will use Python-style error handling by default.

**Old Lua-style API:**

```python
host, err = kong.request.get_header("host")
if err:
    handle_error(err)
```

**New Python-style API:**

```python
try:
    host = kong.request.get_header("host")
except Exception as ex:
    handle_error(ex)
```

Enable this now with `--no-lua-style`.

## TODO

- Tests
- Hot reload
