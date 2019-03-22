<h1 align="center">
<img src="https://rawgit.com/jdorfman/rump/master/assets/images/rump_logo.svg">
</h1>

Hot sync two Redis databases using dumps.

## Why

There's no easy way to get/sync data from an [AWS ElastiCache]( http://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/ClientConfig.RestrictedCommands.html ) Redis cluster.

> **@bdq**: Hey, let's keep our staging Redis containers in sync with our AWS ElastiCache. `BGSAVE` and copy the .rdb?

>**@nixtrace**: Yeah, awesome, let me try... [Nope, not supported](http://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/ClientConfig.RestrictedCommands.html).

>**@bdq**: Ah, that's bad. We'll have to set the containers as `SLAVEOF`?

>**@nixtrace**: That makes sense, doing it... [Nope, not supported](http://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/ClientConfig.RestrictedCommands.html).

>**@bdq**: WAT. Let's use an open source tool to do the sync?

>**@nixtrace**: Most of them use `KEYS` to get the keys, we'd DoS our own server.

>**@bdq**: Let's write a script?

>**@nixtrace**: Tried. Bash doesn't like key dumps, Ruby/Python + deps take more space than Redis inside the container.

>**[@bdq](https://github.com/BDQ)** and **[@nixtrace](https://github.com/nixtrace)**: Let's write it in Go?


Rump is able to transfer keys from an ElastiCache cluster or any Redis server to another Redis server, by only using `SCAN`, `DUMP` and `RESTORE`.

## Features

- Uses `SCAN` instead of `KEYS` to avoid DoS your own server.
- Can sync any key type.
- Drops the TTL on purpose, since it wouldn't be in sync.
- Doesn't use any temp file.
- Uses buffered channels to optimize slow source servers.
- Uses pipelines to minimize network roundtrips.

## Examples

```sh
# Sync local Redis DB 1 to DB 2.
$ rump -from redis://127.0.0.1:6379/1 -to redis://127.0.0.1:6379/2

# Sync ElastiCache cluster to local.
$ rump -from redis://production.cache.amazonaws.com:6379/1 -to redis://127.0.0.1:6379/1

# Sync protected ElastiCache via EC2 port forwarding.
$ ssh -L 6969:production.cache.amazonaws.com:6379 -N ubuntu@xxx.xxx.xxx.xxx &
$ rump -from redis://127.0.0.1:6969/1 -to redis://127.0.0.1:6379/1
```

## Demo

[![asciicast](https://asciinema.org/a/94355.png)](https://asciinema.org/a/94355)

## Contribute

- We use GitHub issues to discuss everything: features, bugs, docs.
- Before sending a pull request always open an issue.

## Install

You can find pre-compiled binaries on the [releases](https://github.com/stickermule/rump/releases) page. If you don't see your OS/Arch there, just ask :)

## Development

```
brew install go
```

```
go get -u github.com/garyburd/redigo/redis
```

```
./multi-arch.sh 0.0.4
```

## Mentions

- [Sticker Mule Blog](https://www.stickermule.com/blog/introducing-rump)
- [The Changelog](http://email.changelog.com/t/ViewEmail/t/13CBF627BB99BB74/)
- [Go Weekly](http://golangweekly.com/issues/138)
- [DB Weekly](http://dbweekly.com/issues/132)
- [Redis Weekly](http://redisweekly.com/archive/172.html)
- [Cron Weekly](https://www.cronweekly.com/issue-59/)

## Maintainers

[nixtrace](https://github.com/nixtrace)

## License

Rump is licensed under the [MIT License](https://opensource.org/licenses/MIT)
