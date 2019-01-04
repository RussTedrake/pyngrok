# pyngrok - a Python wrapper for ngrok

[![PyPI version](https://badge.fury.io/py/pyngrok.svg)](https://badge.fury.io/py/pyngrok)
[![image](https://img.shields.io/pypi/pyversions/pyngrok.svg)](https://pypi.org/project/pyngrok/)
[![codecov](https://codecov.io/gh/alexdlaird/pyngrok/branch/master/graph/badge.svg)](https://codecov.io/gh/alexdlaird/pyngrok)
[![Build Status](https://travis-ci.org/alexdlaird/pyngrok.svg?branch=master)](https://travis-ci.org/alexdlaird/pyngrok)

## install

`pyngrok` is available on [PyPI](https://pypi.org/project/pyngrok/) and can be installed
using `pip`.

```sh
pip install pyngrok
```

The `pyngrok` package manages its own [ngrok](https://ngrok.com/) binary, though this is
configurable.

## open a tunnel

To open a tunnel, use the `connect()` method, which returns the public URL generated by `ngrok`.

```python
from pyngrok import ngrok

# Open a tunnel on the default port 80
public_url = ngrok.connect()
```

The `connect()` can also take an `options` parameter, which allows us to pass additional options
that are [supported by ngrok](https://ngrok.com/docs#tunnel-definitions).

## get active tunnels

It can be useful to ask the `ngrok` client what tunnels are currently open, which can be
accomplished with the `get_tunnels()` method, which returns a list of `NgrokTunnel` objects.

```python
from pyngrok import ngrok

tunnels = ngrok.get_tunnels()
# A public ngrok URL that tunnels to port 80 (ex. http://<public_sub>.ngrok.io)
public_url = tunnels[0].public_url
```

## closing a tunnel

All open tunnels will automatically be closed when the Python process terminates, but we can
close them manually.

```python
from pyngrok import ngrok

public_url = "http://<public_sub>.ngrok.io"

ngrok.disconnect(public_url)
```

## the ngrok process

Opening a tunnel will start the `ngrok` process. This process will remain alive, and the tunnels
open, until `ngrok.kill()` is invoked, or until the Python process terminates.

If we are building a short-lived app, for instance a CLI, we may want to block on the `ngrok`
process so tunnels stay open until the user intervenes. We can do that by accessing the `NgrokProcess`.

```python
from pyngrok import ngrok

ngrok_process = ngrok.get_ngrok_process()
# Block until CTRL-C or some other terminating event
ngrok_process.process.wait()
```

The `NgrokProcess` also contains an `api_url` variable, usually initialized to
`http://127.0.0.1:4040`, from which we can access the [ngrok client API](https://ngrok.com/docs#client-api).
If some feature we need is not available in this package, the client API is accessible to us via the
`api_request()` method. Additionally, `NgrokTunnel` objects expose a `uri` variable, which contains
the relative path used to manipulate that resource against the client API.

## other useful configuration

### authtoken

Running `ngrok` with an auth token enables additional features available on our account (for
instance, the ability to open more tunnels concurrently). We can obtain our auth token from
the [ngrok dashboard](https://dashboard.ngrok.com) and install it like this:

```python
from pyngrok import ngrok

ngrok.set_auth_token("<NGROK_AUTH_TOKEN>")
```

This will set the auth token in the config file. We can also set it in a one-off fashion by
setting it for [the "auth" key](https://ngrok.com/docs#tunnel-definitions) of the `options` parameter
passed to `connect()`.

### config file

The default [`ngrok` config file](https://ngrok.com/docs#config) lives in the home
directory's `.ngrok2` folder. We can change this in one of two ways. Either pass the
`config_path` parameter to methods:

```python
from pyngrok import ngrok

CONFIG_PATH = "/opt/ngrok/config.yml"

ngrok.connect(config_path=CONFIG_PATH)
```

or override the `DEFAULT_CONFIG_PATH` variable:

```python
from pyngrok import ngrok

ngrok.DEFAULT_CONFIG_PATH = "/opt/ngrok/config.yml"

ngrok.set_auth_token("<NGROK_AUTH_TOKEN>")
```

### binary path

The `pyngrok` package manages its own `ngrok` binary. However, we can use our `ngrok` binary if we
want in one of two ways.  Either pass the `ngrok_path` parameter to methods:

```python
from pyngrok import ngrok

NGROK_PATH = "/usr/local/bin/ngrok"

ngrok.get_tunnels(ngrok_path=NGROK_PATH)
```

or override the `DEFAULT_NGROK_PATH` variable:

```python
from pyngrok import ngrok

ngrok.DEFAULT_NGROK_PATH = "/usr/local/bin/ngrok"

ngrok.connect(5000)
```

## contributing

If you find issues, [report them on GitHub](https://github.com/alexdlaird/pyngrok/issues). Pull
requests for fixes and features are also warmly welcomed.
