---
layout: post
title: "解决 redis-search 在 Passenger 下的 reconnect 错误"
date: 2012-11-17 18:12
comments: true
categories: 
---
[redis-search](https://github.com/huacnlee/redis-search) 是一款基于 Redis 的高效搜索组件, 使用起来非常方便, 效果也非常不错, 但是偶尔搜索的时候有这个错误:

`Tried to use a connection from a child process without reconnecting. You need to reconnect to Redis after forking.`


这个错误是如何产生的呢？
我看了一下 redis-rb 的 unicorn example，找到了这个：

```ruby
  require "redis"

  worker_processes 3

  # If you set the connection to Redis *before* forking,
  # you will cause forks to share a file descriptor.
  #
  # This causes a concurrency problem by which one fork
  # can read or write to the socket while others are
  # performing other operations.
  #
  # Most likely you'll be getting ProtocolError exceptions
  # mentioning a wrong initial byte in the reply.
  #
  # Thus we need to connect to Redis after forking the
  # worker processes.

  after_fork do |server, worker|
    Redis.current.quit
  end
```

然后我试着找资料写了一个 PhusionPassenger 的解决方案

```ruby
  # config/initializers/redis_search.rb
  require "redis"
  require "redis-namespace"
  require "redis-search"
  # don't forget change namespace

  if defined?(PhusionPassenger)
    PhusionPassenger.on_event(:starting_worker_process) do |forked|
      if forked
        Redis.current.client.reconnect
      else
        Redis.current = Redis.new(:host => "127.0.0.1",:port => "6379")
      end
    end
  end

  redis = Redis.current
  redis.select(3)
  # We suggest you use a special db in Redis, when you need to clear all data, you can use flushdb command to clear them.
  # 
  # Give a special namespace as prefix for Redis key, when your have more than one project used redis-search, this config will make them work fine.
  redis = Redis::Namespace.new("wakmj:redis_search", :redis => redis)
  Redis::Search.configure do |config|
    config.redis = redis
    config.complete_max_length = 100
    config.pinyin_match = true
    # use rmmseg, true to disable it, it can save memroy
    config.disable_rmmseg = false
  end
```

好了，重新启动，现在没有如上的错误了。
参考链接：[https://github.com/redis/redis-rb/blob/master/lib/redis/client.rb#L277](https://github.com/redis/redis-rb/blob/master/lib/redis/client.rb#L277)

