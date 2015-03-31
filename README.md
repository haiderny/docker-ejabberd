# docker-ejabberd

[Ejabberd][ejabberd] server version 15.03 with SSL and internal auth enabled by default. To control the XMPP server, register an admin user 'admin@\<domain\>' with your prefered XMPP client or [ejabberdctl][register]. You can change the default domain `localhost` and other settings through [environment variables][envvar].

[ejabberd]: http://ejabberd.im
[register]: #register-the-admin-user
[envvar]: #environment-variables--runtime-configuration

## Quickstart example

You can start of with the following container:

    docker run -d \
        --name "ejabberd" \
        -p 5222:5222 \
        -p 5269:5269 \
        -p 5280:5280 \
        -h 'example.de' \
        -e "XMPP_DOMAIN=example.de" \
        -e "ERLANG_NODE=ejabberd" \
        -e "EJABBERD_ADMIN=admin@example.de admin2@example.de"
        -e "TZ=Europe/Berlin" \
        rroemhild/ejabberd
    docker exec -ti ejabberd ejabberdctl register admin example.de password1234
    docker exec -ti ejabberd ejabberdctl register admin2 example.de password12345

## Usage

### Run in background

```
$ docker run -d -P rroemhild/ejabberd
```

### Run in foreground with attached erlang shell

Set `-i` and `-t` option and append `live` to get an interactive erlang shell:

```
$ docker run -i -t -P rroemhild/ejabberd live
```

You can terminate the erlang shell with `q().`.

### Using your own ssl certificates

TLS is enabled by default and the run script will auto-generate two snakeoil certificates during boot if you don't provide your ssl certificates.

To use your own certificates mount the volume `/opt/ejabberd/ssl` to a local directory with the `.pem` files:

* /tmp/ssl/host.pem (SERVER_HOSTNAME)
* /tmp/ssl/xmpp_domain.pem (XMPP_DOMAIN)

Make sure that the certificate and private key are in one `.pem` file. If one file is missing it will be auto-generated. I.e. you can provide your certificate for your `XMMP_DOMAIN` and use a snakeoil certificate for the `SERVER_HOSTNAME`.

## Using docker-ejabberd as base image

The image is called `rroemhild/ejabberd` and is available on the Docker registry.

```
FROM rroemhild/ejabberd
ADD ./ejabberd.yml.tpl /opt/ejabberd/conf/ejabberd.yml.tpl
ADD ./ejabberd.yml.tpl /opt/ejabberd/conf/ejabberdctl.cfg.tpl
```

If you need root privileges switch to `USER root` and go back to `USER ejabberd` when you're done.

## Environment variables / Runtime configuration

You can additionally provide extra runtime configuration in a downstream image by replacing the config template `ejabberd.yml.tpl` with one based on this image's template and include extra interpolation of environment variables. The template is parsed by Jinja2 with the runtime environment (equivalent to Python's `os.environ` available as `env`).

### Additional options

With `ERL_OPTIONS` you can overwrite additional options passed to erlang while starting ejabberd.

### XMPP domain

By default the container will serve the XMPP domain `localhost`. In order to serve a different domain at runtime, provide the `XMPP_DOMAIN` variable as such:

```
$ docker run -d -P -e "XMPP_DOMAIN=foo.com" rroemhild/ejabberd
```

### Set admin users

Set one or more admin user (seperated by whitespace) with the `EJABBERD_ADMIN` environment variable:

```
$ docker run -d --name ejabberd -P \
    -e "XMPP_DOMAIN=foo.com" \
    -e "EJABBERD_ADMIN=admin@foo.com" \
    rroemhild/ejabberd
```

To automatically register admin users with a random password set the variable `EJABBERD_AUTO_RANDPWD` to `true` and check the container logs:

```
$ docker run -d --name ejabberd -P \
    -e "XMPP_DOMAIN=foo.com" \
    -e "EJABBERD_ADMIN=admin@foo.com admin2@foo.com" \
    -e "EJABBERD_AUTO_RANDPWD=true" \
    rroemhild/ejabberd

$docker logs ejabberd
[...]
User admin@foo.com successfully registered
Password for user admin@foo.com is 7CsnbmMW-9w0Ka06
```

Or provide the passwords with the `EJABBERD_ADMIN_PWD` environment variable:

```
$ docker run -d --name ejabberd -P
    -e "XMPP_DOMAIN=foo.com" \
    -e "EJABBERD_ADMIN=admin1@foo.com admin2@foo.com" \
    -e "EJABBERD_ADMIN_PWD=pass123 pass234" \
    rroemhild/ejabberd
```

### Loglevel

By default the loglevel is set to INFO (4). To set another loglevel provide the `LOGLEVEL` variable as such:

```
$ docker run -d -P -e "LOGLEVEL=5" rroemhild/ejabberd
```

```
loglevel: Verbosity of log files generated by ejabberd.
0: No ejabberd log at all (not recommended)
1: Critical
2: Error
3: Warning
4: Info
5: Debug
```

### Erlang node name

By default the erlang node name is set to `ejabberd@localhost`. If you want to set the erlang node name to hostname provide the `ERLANG_NODE` variable such as:

```
$ docker run -d -P -e "ERLANG_NODE=ejabberd" rroemhild/ejabberd
```

For more `ERLANG_NODE` formats see `ejabberdctl.cfg.tpl`.

### Erlang cookie

By default the erlang cookie is generated when ejabberd starts and can't find the `.erlang.cookie` file in $HOME. To set your own cookie provide the `ERLANG_COOKIE` variable with your cookie such as:

```
$ docker run -d -P -e "ERLANG_COOKIE=YOURERLANGCOOKIE" rroemhild/ejabberd
```

### Anonymous auth

Set the `AUTH_METHOD` variable to enable `anonymous`:

```
$ docker run -d -P -e "AUTH_METHOD=anonymous" rroemhild/ejabberd
```

## Run ejabberdctl in container

The `ejabberdctl` command is in the search path and can be run by:

```
$ docker exec CONTAINER ejabberdctl help
```

### Register the admin user

```
$ docker exec CONTAINER ejabberdctl register admin XMPP_DOMAIN PASSWORD
```

## Exposed ports

* 5222
* 5269
* 5280
* 4560

## Exposed volumes

* /opt/ejabberd/database
* /opt/ejabberd/ssl
