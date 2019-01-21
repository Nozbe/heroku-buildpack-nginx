# Bidvine's Openresty Heroku Buildpack

Openresty-buildpack vendors Openresty inside a dyno and runs NGINX per your configuration.

## Versions

* Openresty Version: 1.13.6.2
* Heroku Stack: heroku-18

## Requirements

* Add a custom nginx config to your app source code at `config/nginx.conf.erb`.

## Features

* Unified NXNG/App Server logs.
* [L2met](https://github.com/ryandotsmith/l2met) friendly NGINX log format.
* [Heroku request ids](https://devcenter.heroku.com/articles/http-request-id) embedded in NGINX logs.
* Crashes dyno if NGINX or App server crashes. Safety first.
* Language/App Server agnostic.
* Customizable NGINX config.
* Application coordinated dyno starts.

### Logging

NGINX will output the following style of logs:

```
measure.nginx.service=0.007 request_id=e2c79e86b3260b9c703756ec93f8a66d
```

You can correlate this id with your Heroku router logs:

```
at=info method=GET path=/ host=salty-earth-7125.herokuapp.com request_id=e2c79e86b3260b9c703756ec93f8a66d fwd="67.180.77.184" dyno=web.1 connect=1ms service=8ms status=200 bytes=21
```

### NGINX Solo Mode

openresty-buildpack provides a command named `bin/start-nginx-solo`.
This requires you to put a `config/nginx.conf.erb` in your app code. You can start by coping the [sample config for nginx solo mode](config/nginx-solo-sample.conf.erb).
For example, to get NGINX up and running:

```bash
$ cat Procfile
web: bin/start-nginx-solo
```

### Setting the Worker Processes

You can configure NGINX's `worker_processes` directive via the
`NGINX_WORKERS` environment variable.

For example, to set your `NGINX_WORKERS` to 8 on a PX dyno:

```bash
$ heroku config:set NGINX_WORKERS=8
```

### Customizable Config

You can provide your own config by creating a file named `nginx.conf.erb` in the config directory of your app. Start by copying the buildpack's [default config file](config/nginx.conf.erb).

### Customizable Openresty Compile Options

See below for the build steps. Configuring is as easy as changing the "./configure" options.

You can run the builds in a [Docker](https://www.docker.com/) container:


### Building a new Openresty binary:

Download and install [Docker](https://www.docker.com/)

From your command line:
```
# Pull the Heroku build environment image you want to use
docker pull heroku/heroku:18-build

# Run the Heroku image
docker run -it heroku/heroku:18-build
```

From the container's terminal you're now connected to:
```
# Download the Openresty version you're using
curl -O -L https://openresty.org/download/openresty-1.13.6.2.tar.gz
tar xzf openresty-1.13.6.2.tar.gz
cd openresty-1.13.6.2

# Download the PCRE version you're using
curl -O -L http://iweb.dl.sourceforge.net/project/pcre/pcre/8.42/pcre-8.42.tar.bz2
tar xjf pcre-8.42.tar.bz2

# Build Openresty
PATH=$PATH:/sbin ./configure --with-pcre=pcre-8.42 --with-luajit --with-http_postgres_module --with-file-aio --with-ipv6 --with-http_realip_module --with-http_addition_module --with-http_xslt_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_stub_status_module --with-mail --with-mail_ssl_module --with-pcre-jit --with-http_iconv_module -j2
make
make install

# Zip Openresty
cd /usr/local
tar -czvf openresty-1.13.6.2-heroku-build.tar.gz openresty

# Don't exit the container until after you've copied the zip file out of it below
```

From your command line:
```
# List running containers
docker ps

# Copy the zip file you built off the running container
docker cp cbbe1368db40:/usr/local/openresty-1.13.6.2-heroku-build.tar.gz .
```
