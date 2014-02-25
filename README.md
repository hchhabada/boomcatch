# boomcatch

[![Build status][ci-image]][ci-status]

Standalone,
node.js-based
beacon server for [boomerang]/[kylie].

* **boomcatch version**: *1.0.0*
* **node.js versions**: *0.8, 0.10, 0.11*

## Installation

### At the system level

First you must
[install node][node].

You can then
install boomcatch via npm:

```
npm install -g boomcatch
```

### Local to a node.js project

Add boomcatch
to the dependencies
in your project's `package.json`,
then run:

```
npm install
```

## Usage

### From the command line

To see
the list of command line options
run:

```
boomcatch --help
```

Available options are:

* `--host <name>`:
  Host name to accept HTTP connections on.
  The default is 0.0.0.0 (INADDR_ANY).

* `--port <port>`:
  Port to accept HTTP connections on.
  The default is 80.

* `--path <path>`:
  URL path to accept requests to.
  The default is /beacon.

* `--referer <regex>`:
  HTTP referers to accept requests from.
  The default is `.*`.

* `--limit <milliseconds>`:
  Minimum elapsed time to allow
  between requests from the same IP adderss.
  The default is 0.

* `--silent`:
  Prevent the command
  from logging output
  to the console.

* `--validator <path>`:
  Validator used to accept or reject request data.
  The default is permissive
  (always accept requests).

* `--mapper <path>`:
  Data mapper used to transform data before forwarding,
  loaded with [require].
  The default is statsd.

* `--prefix <prefix>`:
  Prefix for mapped metric names.
  The default is the empty string
  (no prefix).

* `--forwarder <path>`:
  Forwarder used to send data,
  loaded with require.
  The default is udp.

* `--fwdHost <name>`:
  Host name to forward mapped data to.
  The default is 127.0.0.1.

* `--fwdPort <port>`:
  Port to forward mapped data on.
  The default is 8125.

### From a node.js project

```javascript
var path = require('path'),
    boomcatch = require('boomcatch');

boomcatch.listen({
    host: 'rum.example.com',                  // Defaults to '0.0.0.0' (INADDR_ANY)
    port: 8080,                               // Defaults to 80
    path: '/perf',                            // Defaults to '/beacon'
    referer: /^\w+\.example\.com$/,           // Defaults to /.*/
    limit: 100,                               // Defaults to 0
    log: console.log,                         // Defaults to `function () {}`
    validator: path.resolve('./myvalidator'), // Defaults to 'permissive'
    mapper: path.resolve('./mymapper'),       // Defaults to 'statsd'
    prefix: 'mystats.rum.',                   // Defaults to ''
    forwarder: path.resolve('./myforwarder'), // Defaults to 'udp'
    fwdHost: '192.168.50.4',                  // Defaults to '127.0.0.1'
    fwdPort: 5001                             // Defaults to 8125
});
```

### Data mappers

Mappers are used
to transform data
into appropriate formats
for back-end stats consumers.
Currently, one mapper is available out-of-the-box,
which formats the metrics
as [statsd] timers.

Defining a custom data mapper
is straightforward.
The [source code for the statsd mapper][mapper]
should be easy to follow,
but the basic pattern
is to export an interface
that looks like this:

```javscript
{
    initialise: function (options) {
    }
}
```

The `initialise` function
should return a function
that is passed a data object
as its only parameter
and returns the mapped data.

If you then specify
the path to your new mapper
with the `mapper` option,
a first attempt to load it
is made relative
to this project's `src/mappers` directory.
When that call to `require` fails,
a second attempt will be made
using the path that you specified verbatim.

### Data forwarders

Forwarders are used
to send mapped data
to back-end stats consumers.
Currently, one forwarder is available out-of-the-box,
which dispatches the data over UDP.

Defining a custom forwarder
takes broadly the same form
as [defining a custom mapper](#data-mappers)
and can be seen
in the [source code for the udp forwarder][forwarder].
Again, the module should export
an interface that looks like this:

```javscript
{
    initialise: function (options) {
    }
}
```

In this case,
the `initialise` function
should return a function
that is passed
the mapped data
and a callback function
as its two parameters.
When the forwarding process has completed,
the callback function should be invoked,
following the node.js convention
of the first argument containing any error,
or a falsey value if things went okay.
The second argument should contain
the number of bytes that were sent.

Once your custom forwarder is ready,
you can specify the path to it
using the `forwarder` option.

### Request validators

Validators are used
to test whether each request
should be accepted or rejected.
Typically, they check
for the presence
of a valid nonce
in the query string,
as a preventative measure
against denial-of-service attacks.
One validator is available out-of-the-box,
which simply accepts all requests.

As with mappers and forwarders,
defining your own validator is easy
and the basic interface
looks the same:

```javscript
{
    initialise: function (options) {
    }
}
```

Here,
the `initialise` function
should return a function
that receives an object
representing the parsed query string
and returns either `true` or `false`,
signifying that the request
is valid or invalid respectively.

Requests that are identified as invalid
will fail with an HTTP 400 status.

## Development

Before sumitting any pull requests,
please ensure that you have
adhered to the [contribution guidelines][contrib].

To clone the repository:

```
git clone git@github.com:nature/boomcatch.git
```

To set up the development environment:

```
npm install
```

To lint the code:

```
npm run lint
```

To run the unit tests:

```
npm test
```

## Change log

[History]

## License

[GPL 3][license]

Copyright © 2014 Nature Publishing Group

[ci-image]: https://secure.travis-ci.org/nature/boomcatch.png?branch=master
[ci-status]: http://travis-ci.org/#!/nature/boomcatch
[boomerang]: https://github.com/lognormal/boomerang
[kylie]: https://github.com/forcedotcom/kylie
[node]: http://nodejs.org/download/
[require]: http://nodejs.org/api/globals.html#globals_require
[statsd]: https://github.com/etsy/statsd/
[mapper]: https://github.com/nature/boomcatch/blob/master/src/mappers/statsd.js
[forwarder]: https://github.com/nature/boomcatch/blob/master/src/forwarders/udp.js
[contrib]: https://github.com/nature/boomcatch/blob/master/CONTRIBUTING.md
[history]: https://github.com/nature/boomcatch/blob/master/HISTORY.md
[license]: https://github.com/nature/boomcatch/blob/master/COPYING
