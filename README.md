# fluent-plugin-http-pull

[![Build Status](https://travis-ci.org/HatsuneMiku3939/fluent-plugin-http-pull.svg?branch=master)](https://travis-ci.org/HatsuneMiku3939/fluent-plugin-http-pull)
[![Build status](https://ci.appveyor.com/api/projects/status/k91x4jyhahoo2it3?svg=true)](https://ci.appveyor.com/project/HatsuneMiku3939/fluent-plugin-http-pull)
[![Gem Version](https://badge.fury.io/rb/fluent-plugin-http-pull.svg)](https://badge.fury.io/rb/fluent-plugin-http-pull)
[![Coverage Status](https://coveralls.io/repos/github/HatsuneMiku3939/fluent-plugin-http-pull/badge.svg?branch=master)](https://coveralls.io/github/HatsuneMiku3939/fluent-plugin-http-pull?branch=master)


[Fluentd](http://fluentd.org/) input plugin to pull log from rest api.

Many of modern server application offer status reporting API via http (even 'fluentd' too). This plugin will help to gathering status log from these status api.

## Installation

### RubyGems

```
$ gem install fluent-plugin-http-pull
```

### Bundler

Add following line to your Gemfile:

```ruby
gem "fluent-plugin-http-pull"
```

And then execute:

```
$ bundle
```

## Example

You can found more examples in `test/plugin/test_in_http_pull.rb`

### Monitoring http status code only
```
<source>
  @type http_pull

  tag test
  url http://www.google.com
  interval 1s

  format none
  status_only true
</source>

<match test>
  @type stdout
</match>

# 2017-05-17 21:40:40.413219000 +0900 test: {"url":"http://www.google.com","status":200}
# 2017-05-17 21:40:41.298215000 +0900 test: {"url":"http://www.google.com","status":200}
# 2017-05-17 21:40:42.310993000 +0900 test: {"url":"http://www.google.com","status":200}
# 2017-05-17 21:40:43.305947000 +0900 test: {"url":"http://www.google.com","status":200}
```

### Monitoring fluentd itself
```
<source>
  @type monitor_agent

  port 24220
</source>

<source>
  @type http_pull

  tag fluentd.status
  url http://localhost:24220/api/plugins.json
  interval 1s

  format json
</source>

<match fluentd.status>
  @type stdout
</match>

# 2017-05-17 21:41:47.872951000 +0900 fluentd.status: {"url":"http://localhost:24220/api/plugins.json","status":200,"message":{"plugins":[{"plugin_id":"object:1e7e3d...
# 2017-05-17 21:41:48.955316000 +0900 fluentd.status: {"url":"http://localhost:24220/api/plugins.json","status":200,"message":{"plugins":[{"plugin_id":"object:1e7e3d...
# 2017-05-17 21:41:50.033628000 +0900 fluentd.status: {"url":"http://localhost:24220/api/plugins.json","status":200,"message":{"plugins":[{"plugin_id":"object:1e7e3d...
# 2017-05-17 21:41:51.107372000 +0900 fluentd.status: {"url":"http://localhost:24220/api/plugins.json","status":200,"message":{"plugins":[{"plugin_id":"object:1e7e3d...
```


### Monitoring elasticsearch cluster health
```
<source>
  @type http_pull

  tag es.cluster.health
  url http://localhost:9200/_cluster/health
  interval 1s

  format json
</source>

<match es.cluster.health>
  @type stdout
</match>

# 2017-05-17 12:49:09.886298008 +0000 es.cluster.health: {"url":"http://localhost:9200/_cluster/health","status":200,"message":{"cluster_name":"elasticsearch","status":"green",...
# 2017-05-17 12:49:10.669431296 +0000 es.cluster.health: {"url":"http://localhost:9200/_cluster/health","status":200,"message":{"cluster_name":"elasticsearch","status":"green",...
# 2017-05-17 12:49:11.668789668 +0000 es.cluster.health: {"url":"http://localhost:9200/_cluster/health","status":200,"message":{"cluster_name":"elasticsearch","status":"green",...
# 2017-05-17 12:49:12.668789849 +0000 es.cluster.health: {"url":"http://localhost:9200/_cluster/health","status":200,"message":{"cluster_name":"elasticsearch","status":"green",...
```

## Configuration

### tag (string) (required)

The tag of the event.

### url (string) (required)

The url of remote server.

### agent (string) (optional, default: fluent-plugin-http-pull)

The user agent string of request.

### interval (time) (required)

The interval time between periodic request.

### format (required)

The format of the response body. Due to limitation of current implement it is
always required regardless `status_only` option.

`http_pull` uses parse plugin to parse the response body. See
[parser article](https://docs.fluentd.org/v0.12/articles/parser-plugin-overview)
for more detail.

### status_only (bool) (optional, default: false)

If `status_only` is true, body is not parsed.

### http_method (enum) (optional, default: :get)

The http request method for each requests. Avaliable options are listed below.

* `get`
* `post`
* `delete`

If `status_only` is true, `http_method` was override to `head`

### timeout (time) (optional, default: 10s)

The timeout of each request.

### proxy (string) (optional, default: nil)

The HTTP proxy URL to use for each requests

### user (string) (optional, default: nil)

The user for basic auth

### password (string) (optional, default: nil)

The password for basic auth

### response_header (section) (optional, default: nil)

The name of response header for capture.

### request_header (section) (optional, default: nil)

The name, value pair of custom reuqest header.

## In case of remote error

### Can receive response from remote

```
{
  "url": url of remote
  "status": status code of response
  "error": "RestClient::NotFound: 404 Not Found" or something similar
}
```

### All the other case

```
{
  "url": url of remote
  "status": 0
  "error": "Errno::ECONNREFUSED: Connection refused - connect(2) for "localhost" port 12345" or something similar
}
```

## Copyright

* Copyright(c) 2017- filepang
* License
  * Apache License, Version 2.0
