# Bright Sky - Infrastructure

Stackable Docker Compose configurations to set up your own instance of [Bright
Sky](https://github.com/jdemaeyer/brightsky/).

This is the exact configuration used for the public Bright Sky instance at
https://api.brightsky.dev/, which runs on a [Hetzner CX31
VPS](https://www.hetzner.com/cloud#pricing), usually not using more than 4 GB
of the available 8 GB of memory, and occupying about 20 GB of disk space for
all weather records starting January 1, 2010.


## Setup instructions

You will need a machine with [Docker
Engine](https://docs.docker.com/engine/install/) and [Docker
Compose](https://docs.docker.com/compose/install/) installed. Then, copy or clone
this repository to your machine, e.g.:

```console
$ git clone https://github.com/jdemaeyer/brightsky-infrastructure.git brightsky
$ cd brightsky
```

If you are not interested in historical data past a certain date, now is a good
time to configure it. Add a file `brightsky.env` with the following content:

```
BRIGHTSKY_MIN_DATE=2020-01-01
```

The next steps depend on how public you want your Bright Sky instance to be:


### Run a local (network-internal) instance

Run:

```console
# ./brightsky up -d
```

This will start all necessary containers and then detach from them. You can
view the live logs by running `./brightsky logs -f`, and stop the containers by
running `./brightsky down`.

The API will be available at `http://localhost:5000/`.


### Run a public instance behind a Traefik router

First, you will need a hostname with DNS records pointing to your server. Add a
file named `.env` with the following content:

```
HOSTNAME=your.host.name
LETSENCRYPT_EMAIL=your@mail.address
```

Next, create a file named `config` with the following content:

```
brightsky
traefik
```

Finally, run:

```console
# ./brightsky up -d
```

This will start a Bright Sky and Traefik router, acquire a TLS certificate from
Let's Encrypt, and make the API available at `https://your.host.name/`.


#### Authentication

If you wish to require authentication for your Bright Sky instance, you can
easily do so using Traefik's [BasicAuth
middleware](https://docs.traefik.io/v2.2/middlewares/basicauth/). Add a line
containing `auth` to your `config` file, so that it reads:

```
brightsky
traefik
auth
```

And run this command for every user/password combination you wish to allow
(replace `USERNAME` with the actual username, you will be prompted for the
password):

```console
# ./brightsky adduser USERNAME
```

Then use

```console
# ./brightsky up -d
```

to start your Bright Sky instance.


### Run a public instance behind a Traefik router with basic usage analytics

Make sure you have a separate hostname with DNS records pointing to your
server. Add it to your `.env` file as `HOSTNAME_GRAFANA`, e.g.:

```
HOSTNAME=your.host.name
HOSTNAME_GRAFANA=grafana.your.host.name
LETSENCRYPT_EMAIL=your@mail.address
```

Next, add an additional line containing `analytics` to your `config` file.
Then simply follow the instructions for setting up a public instance above.

Make sure to log into your Grafana instance and change the default admin
password.


## Additional Configuration


### Configuring Bright Sky options

Bright Sky allows configurating [all its
options](https://github.com/jdemaeyer/brightsky/blob/master/brightsky/settings.py)
through environment variables. You can set these variables in `brightsky.env`.
E.g. this `brightsky.env` would only parse/store data from 2018 onwards, allow
cross-origin requests from `myweatherapp.com`, and poll the DWD server only
every 10 minutes:

```
BRIGHTSKY_MIN_DATE=2018-01-01
BRIGHTSKY_CORS_ALLOWED_ORIGINS=https://myweatherapp.com
BRIGHTSKY_POLLING_CRONTAB_MINUTE=*/10
```


### Add Sentry error tracking

Add your Sentry DSN to `brightsky.env`, e.g.:

```
SENTRY_DSN=https://your_sentry_dsn_here
```
