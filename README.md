# Munin Plugin for CouchDB #

This [Munin][1] plugin allows to monitor [Apache CouchDB][2] instance.


## Install and Setup ##

First of all, ensure that your system has installed [Perl 5.12+][3] and
two additional libraries: [LWP::UserAgent][4] and [JSON][5]. Sure, you would
also need to have [Munin][1] installed.

The plugin installation is quite trivial operation

1. `git clone https://github.com/kxepal/munin-plugin-couchdb`
2. `cp munin-plugin/couchdb_ /etc/munin/plugins`
3. Read the docs via `perldoc couchdb_`
4. Create `/etc/munin/plugin-conf.d/couchdb` and setup proper configuration
5. Check that it works: `su munin -c '/usr/sbin/munin-run couchdb_'`

For additional information about plugins installation consult with
[Munin docs][6].


## Monitoring ##


### HTTPD Metrics ###

These metrics are related to [Mochiweb](https://github.com/mochi/mochiweb) -
the CouchDB's HTTP server which runs the API and communicates with the world.


#### Request Methods ####

The `couchdb_httpd_request_methods` graph provides information about all HTTP
requests in context of used method. It counts the next methods:

- `HEAD`
- `GET`
- `POST`
- `PUT`
- `DELETE`
- `COPY`


#### Requests by Type ####

The `couchdb_httpd_requests` graph shows rate of HTTP requests in context of
their type:

- `HTTP requests`: overall amount of HTTP requests
- `bulk requests`: how often were used [bulk updates][8]
- `view reads`: amount of requests to the [view indexes][9]
- `temporary view reads`: amount of requests to the [temporary view indexes][10]


#### Clients on Continuous Changes Feed ####

While `clients_requesting_changes` metric is in the same group as
`bulk_requests`, `temporary_view_reads` and others,
the `couchdb_clients_requesting_changes` graph shows not requests rate, but
the current amount of active clients to continuous changes feeds.

This graph also helps to roughly estimate amount of continuous replications
that are running against monitored instance.


#### Response Statuse Codes ####

The `couchdb_httpd_status_codes` graph provides information about HTTP
responses in context of [status code][7].

Keeping eye on amount of HTTP `4xx` and `5xx` responses helps you provide
quality service for you users. Normally, you want to see no `500` errors at all.
Having high amount of `401` errors could say about authentication problems
while `403` tell you that something or someone actively doing things that he's
shouldn't do.


### Server Metrics ###

These metrics are related to whole server instance.


### Authentication Cache ###

The `couchdb_auth_cache` graph shows rate of authentication cache hits/misses.

CouchDB keeps some amount of user credentials in memory to speedup
authentication process by elimination of additional database lookups.
This cache size is limited by the configuration option [auth_cache_size][11].
On what this affects? In short, when user logins CouchDB first looks for user
credentials what associated with provided login name in auth cache and if they
miss there then it reads credentials from auth database (in other words,
from disk).

The `auth_cache_miss` metric is highly related to `HTTP 401` responses one,
so there are three cases that are worth to be looked for:

- High `cache misses` and high `401` responses: something brute forces your
  server by iterating over large set of user names that doesn't exists for your
  instance

- High `cache misses` and low `401` responses: your `auth_cache size` is
  too small to handle all your active users. try to increase his capacity
  to reduce disk I/O

- Low `cache misses` and high `401` responses: much likely something tries
  to brute force passwords for existed accounts on your server

Note that "high" and "low" in metrics world should be read as "anomaly high"
and "anomaly low".

Ok, but why do we need auth cache hit then? We need it as an ideal value
to compare misses counter with. Just for instance, is 10 cache misses a high
value? What about 100 or 1000? Having cache hits rate at some point helps
to answer on this question.


#### Databases I/O ####

The `couchdb_database_io` graph shows overall databases read/write rate.


#### Open Databases ####

The `couchdb_open_databases` graph shows amount of currently opened databases.

CouchDB only keeps opened databases which are receives some activity: been
requested or running the compaction. The maximum amount of opened
databases in the same moment of time is limited by [max_dbs_open][12]
configuration option.  When CouchDB hits this limit, any request to "closed"
databases will generate the error response: `{error, all_dbs_active}`.

However, once opened database doesn't remains open forever: in case of
inactivity CouchDB eventually closes it providing more space in the room for
others, but sometimes such cleanup may not help. This graph's goal is to help
you setup correct `max_dbs_open` value that'll fit your needs.

*Note:* Handling `max_dbs_open` configuration value and setting it as critical
mark isn't implemented yet.


#### Open Files ####

The `couchdb_open_files` graph shows amount of currently opened file
descriptors.

*Note:* Handling system `nofile` limit isn't implemented yet and couldn't be
possible for remote instances.


## License ##

[Beerware](https://tldrlegal.com/license/beerware-license)



[1]: http://munin-monitoring.org/
[2]: http://couchdb.apache.org/
[3]: http://www.perl.org/
[4]: http://search.cpan.org/dist/LWP-UserAgent-Determined/
[5]: http://search.cpan.org/dist/JSON/
[6]: https://munin.readthedocs.org/en/latest/plugin/use.html#installing
[7]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html
[8]: http://docs.couchdb.org/en/latest/api/database/bulk-api.html#post--db-_bulk_docs
[9]: http://docs.couchdb.org/en/latest/api/ddoc/views.html
[10]: http://docs.couchdb.org/en/latest/api/database/temp-views.html#post--db-_temp_view
[11]: http://docs.couchdb.org/en/latest/config/auth.html#couch_httpd_auth/auth_cache_size
[12]: http://docs.couchdb.org/en/latest/config/couchdb.html#couchdb/max_dbs_open
