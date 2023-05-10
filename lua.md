可以结合 lua 脚本来实现若干个操作的原子性。本文重点介绍在 redis 中如何使用 lua 脚本。

# lua 脚本语法

redis 中，通常通过 `EVAL` 来执行 lua 脚本。

语法为：
```sh

EVAL script numkeys key [key ...] arg [arg ...]

script: lua 脚本
numkeys: 键的数量
key: 键，lua 脚本中通过 KEYS[1], KEYS[2] 获取。注意第一个 key 是 KEYS[1] 而不是 KEYS[0]
arg: 其他参数，通过 ARGV[1], ARGV[2] 获取。

```

在 lua 脚本中，通常通过 `redis.call` 来执行一个 redis 命令，多个命令之间通过 `;` 分割。比如：
给 key1 设置值为 val1，同时设置过期时间为 100.

EVAL "redis.call('SET', KEYS[1], ARGV[1]); redis.call('EXPIRE', KEYS[1], ARGV[2]); return 1;" 1 key1 val1 100

# lua 脚本复用

每次调用 eval 的时候都需要传递一遍脚本，不管是网络开销还是对于开发人员都不是很友好。为此，redis 提供了 `EVALSHA` 命令，可直接通过 sha 的方式来指定 lua 脚本。该命令的语法为：

```sh

EVALSHA sha numkeys key [key ...] arg [arg ...]

```

我们可以通过 `script exists script_sha` 的方式来判断某个脚本是否已经被 redis 服务端缓存。

我们可以通过 `script load script` 的方式来获取某个脚本的 sha.

我们可以通过 `script flush` 来情况服务端保存的所有 lua 脚本。

脚本复用的实现原理也比较简单，服务器中会维护一个 dict， key 是 sha, val 是脚本内容。

# 使用 lua 脚本的优势

- 支持原子操作。
- 降低网络开销，将多个命令打包到一个请求中，降低网络开销。
- 脚本复用，redis 服务器会将 lua 脚本存储下来，后续可以只通过脚本的 hash 来执行该脚本，无需重新写一遍 lua 脚本。
