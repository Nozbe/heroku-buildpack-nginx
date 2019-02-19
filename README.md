# Bidvine's Openresty Heroku Buildpack

Openresty-buildpack vendors Openresty inside a dyno and runs NGINX per your configuration.

## Versions

* Openresty: 1.13.6.2
* PCRE: 8.42
* Nginx Brotli: https://github.com/eustas/ngx_brotli/commit/8104036af9cff4b1d34f22d00ba857e2a93a243c
* Brotli: https://github.com/google/brotli/tree/c6333e1e79fb62ea088443f192293f964409b04e
* Heroku Stack: heroku-18

## Requirements

* Add a custom nginx config to your app source code at `nginx/config/nginx.conf.erb`.

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

### Language/App Server Agnostic

nginx-buildpack provides a command named `bin/start-nginx` this command takes another command as an argument. You must pass your app server's startup command to `start-nginx`.

For example, to get NGINX and working within a bash script:

```bash
nginx/bin/start-nginx npm run start-server:$env --prefix react/web
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

### Application/Dyno Coordination

Your app should listen to `/tmp/nginx.socket`:

```bash
PORT='/tmp/nginx.socket'
```

```JavaScript
  express().listen(process.env.PORT, function(err) { ... })
```

Touch `/tmp/app-initialized` once your app is listening to the port:

```
fs.closeSync(fs.openSync('/tmp/app-initialized', 'w'));
```

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

# Download Nginx Brotli
curl -O -L https://github.com/eustas/ngx_brotli/archive/master.zip
unzip master.zip

# Download Brotli (based on the commit version specified at https://github.com/eustas/ngx_brotli/tree/master/deps)
cd ngx_brotli-master/deps
rmdir brotli
curl -O -L https://github.com/google/brotli/archive/c6333e1e79fb62ea088443f192293f964409b04e.zip
unzip c6333e1e79fb62ea088443f192293f964409b04e.zip
mv brotli-c6333e1e79fb62ea088443f192293f964409b04e brotli

# Build Openresty
cd ../..
PATH=$PATH:/sbin ./configure --add-module=ngx_brotli-master --with-pcre=pcre-8.42 --with-luajit --with-http_postgres_module --with-file-aio --with-ipv6 --with-http_realip_module --with-http_addition_module --with-http_xslt_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_stub_status_module --with-mail --with-mail_ssl_module --with-pcre-jit --with-http_iconv_module -j2
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
