# SpaceAPI

Our Implementation of the [SpaceAPI](http://spaceapi.net/).

## Server

Serve the [SpaceAPI](http://spaceapi.net/) and some door API end nodes.

### Installation:

```sh
pip3 install --upgrade -r ./requirements-server.txt
```

### Usage:

```
usage: Small script to serve the SpaceAPI JSON API. [-h] [--debug] --key KEY [--host HOST] [--port PORT] [--sql SQL]

optional arguments:
  -h, --help   show this help message and exit
  --debug      Enable debug output
  --key KEY    Path to HMAC key file
  --host HOST  Host to listen on (default 0.0.0.0)
  --port PORT  Port to listen on (default 8888)
  --sql SQL    SQL connection string
```

### Example:

```sh
./spaceapi/spaceapi.py --key /etc/machine-id --host 0.0.0.0 --port 1337 --debug --sql "mysql+pymysql://user:password@host/database"
```

Notes:

- for usage with MySQL you need a MySQL driver like
  [`PyMySQL`](http://docs.sqlalchemy.org/en/latest/dialects/mysql.html#module-sqlalchemy.dialects.mysql.pymysql) installed.
- it is tested with SQLite3 and MySQL but may work with other SQL databases, too. See http://docs.sqlalchemy.org/en/latest/dialects/
- default driver is `sqlite3` with database `sqlite:///:memory:` (does not persists during restarts of the server)
- You can also make *some* configurations for the server script in `/etc/spaceapi.py` or as
  environment variable
  - the config file syntax is a key value py file syntax
  - you can't configure the key there (for reasons)
  - example environment variables: `SPACEAPI_SQL="sqlite:///db.db"`, `SPACEAPI_HOST="0.0.0.0"`
  - example config file:
```py
SQL = "sqlite:///db.db"
HOST = "192.168.1.1"
```

## Client

Can update entries of SpaceAPI server and plot opening hour graphs.

### Installation:

```sh
pip3 install --upgrade -r ./requirements-client.txt
```

### Usage:

```
usage: Client script to update the doorstate on our website or to make plots. [-h] {update,plot} ...

optional arguments:
  -h, --help     show this help message and exit

actions:
  {update,plot}
    update       update door state
    plot         Plot history
```

```
usage: Client script to update the doorstate on our website or to make plots. update
       [-h] [--debug] [--url URL] --key KEY [--time TIME] --state {opened,closed}

optional arguments:
  -h, --help            show this help message and exit
  --debug               Enable debug output
  --url URL             URL to API endpoint
  --key KEY             Path to HMAC key file
  --time TIME           Timestamp since state changed (default now)
  --state {opened,closed}
                        New state
```

```
usage: Client script to update the doorstate on our website or to make plots. plot
       [-h] [--url URL] [--debug] --plot-type {by-hour,by-week} --out OUT

optional arguments:
  -h, --help            show this help message and exit
  --url URL             URL to API endpoint
  --debug               Enable debug output
  --plot-type {by-hour,by-week}
                        The type of the graph to plot
  --out OUT             The file to write the image
```

### Example:

```sh
./spaceapi/doorstate_client.py update --key /etc/machine-id --url http://127.0.0.1:1337/spaceapi/door/ --debug --state open
./spaceapi/doorstate_client.py plot --url http://127.0.0.1:1337/spaceapi/door/all/ --debug --plot-type by-hour --out image.png
```

Notes:

- Only data from within 365 days and at most 2000 entries will be used for plotting

![example by hour plot](./example_by_hour_plot.png)

![example by week plot](./example_by_week_plot.png)

## Installation on `ws01`

We have a serial read sensor to query the door state.
It is located in our `ws01` Linux pc.
It uses the script `misc/update-status.sh`.
This script will be started every minute by the systemd timer in `misc/`.
To install the timer run:

```sh
cp misc/update_doorstate.{service,timer} /etc/systemd/system/
systemctl daemon-reload
systemctl start update_doorstate.timer
systemctl enable update_doorstate.timer
systemctl list-timers
```

## Embed on Website

We embed the door state information provided by `/spaceapi/door/` on our WordPress website using
the widget in https://github.com/fau-fablab/wp-fau-fablab-mods/

## License

[GPLv3](LICENSE)
