# StringRedisTemplate集群使用文档

标签（空格分隔）： 使用文档

---

## 注入

```

@Autowired
@Qualifier("redisClusterTemplate")
StringRedisTemplate clusterTemplate;


```


## Key

```

   /**
     * get set del has getAndSet getIfAbsent increment expire expireAt
     */
    @Test
    public void redisKeyCommandTest() {
        ValueOperations<String, String> ops = clusterTemplate.opsForValue();
        // del
        clusterTemplate.delete("getAndSet");

        //dump
        ops.set("dumpKey", "dumpValue");
        ops.get("dumpKey");

        //exist
        clusterTemplate.hasKey("dumpKey");

        // expire
        clusterTemplate.expire("dumpKey", 10, TimeUnit.MINUTES);
        clusterTemplate.getExpire("dumpKey", TimeUnit.SECONDS);

        //getAndSet
        ops.getAndSet("getAndSet", getRandom());
        clusterTemplate.randomKey();
        // keys type ** 慎用 **
        clusterTemplate.keys("*").forEach(k -> System.out.println(clusterTemplate.type(k)));
    }

```



## String

```

/**
     * String Command
     */
    @Test
    public void redisStringCommandTest() {
        ValueOperations<String, String> ops = clusterTemplate.opsForValue();
        String key = "opsForValueSimpleKeyValueTest";
        //set
        ops.set(key, getRandom());
        //get
        System.out.println(ops.get(key));
        //getRange
        System.out.println(ops.get(key, 1, 6));
        //getSet
        ops.getAndSet(key, getRandom());
        //getBit setBite
        ops.getBit(key, 1);
        ops.setBit(key, 1, true);
        //mget mset
        ops.multiSet(getMap());
        ops.multiGet(Arrays.asList("a", "b"));
        //setex
        ops.set(key, "1", 1, TimeUnit.SECONDS);
        //setnx
        ops.setIfAbsent(key, getRandom());
        //setRange
        ops.set(key, "asdasd", 1);
        //strlen
        ops.size(key);
        //msetnx
        ops.multiSetIfAbsent(getMap());
        //incr
        ops.set(key, "1");

        ops.increment(key, 1d);
        //append
        ops.append(key, "suffix");
    }

```


## HASH


```

 /**
     * HASH
     * 支持多个键值对的操作
     */
    @Test
    public void redisHashTest() {
        HashOperations<String, Object, Object> ops = clusterTemplate.opsForHash();
        String key = "redisHashTest";
        //HMSET
        ops.putAll(key, getMap());
        //HEDL
        ops.delete(key, "key1", "key2");
        //HEXISTS
        ops.hasKey(key, "key1");
        //HGETALL
        ops.entries(key);
        //HINCRBY
        ops.increment(key, "key1", 1f);
        //HKEYS
        ops.keys(key);
        //HLEN
        ops.size(key);
        //HMGET
        ops.multiGet(key, Arrays.asList("key1", "key2"));
        //HVALS
        ops.values(key);

    }

```


## List

```

/**
     * 链表
     * 支持阻塞操作
     */
    @Test
    public void redisListTest() {
        ListOperations<String, String> ops = clusterTemplate.opsForList();
        String key = "redisListTest1";
        String key2 = "redisListTest2123123123";


        //BLPOP 阻塞
        ops.leftPop(key, 1, TimeUnit.SECONDS);
        //BRPOP
        ops.rightPop(key, 1, TimeUnit.SECONDS);
        //BLPOPRPUSH
        ops.rightPopAndLeftPush(key, key2, 1, TimeUnit.SECONDS);
        // LINDEX
        ops.index(key, 1);
        // LINSERT
        ops.leftPush(key, "1");
        ops.set(key, 0, "insertValue");
        //LLEN
        ops.size(key);
        //LPOP
        ops.leftPop(key);
        //LPUSH
        ops.leftPush(key, "1");
        //LPUSH BEFORE
        ops.leftPush(key, "1", "2");
        //LPUSH 多个
        ops.leftPushAll(key, "v1", "v2", "v3");
        //LPUSHX 存在才PUSH
        ops.leftPushIfPresent(key, "4");
        //LRANGE
        ops.range(key, 0, 1);
        //LREM
        ops.remove(key, 1, "4");
        //LTRIM
        ops.trim(key, 0, 3);
        //RPOP
        ops.rightPop(key);
    }

```


## SET


```

 /**
     * SET操作
     */
    @Test
    public void redisSetTest() {
        SetOperations<String, String> ops = clusterTemplate.opsForSet();
        String key = "redisSetTest";
        String key2 = "redisSetTest2";
        //SADD 多个
        ops.add(key, "v1", "v2");
        ops.add(key2, "v1", "v3");
        //SCARD
        ops.size(key);
        //SDIFF
        ops.difference(key, key2);
        //SDIFFSTORE
        ops.differenceAndStore(key, key2, "temp");
        //SINNER
        ops.intersect(key, key2);
        //SINNERSTORE
        ops.intersectAndStore(key, key2, "temp2");
        //SISMEMBER
        ops.isMember(key, "v1");
        //SMEMBERS ** 慎用 **
        ops.members(key);
        //SMOVE
        ops.move(key, "v1", key2);
        //SPOP
        ops.pop(key);
        //SRANDOMMEMBER
        ops.randomMember(key);
        //SREM
        ops.remove(key, key2);
        //SUNION
        ops.union(key, key2);
        //SUNIONSTORE
        ops.add(key, "v1", "v2", "v3");
        ops.add(key2, "v11", "v22", "v33");
        ops.unionAndStore(key, key2, "temp2");
        //SCAN
        ops.add(key, "v1", "v2", "v3");
        ScanOptions options = ScanOptions.scanOptions().build();
        Cursor<String> scan = ops.scan(key, options);

        while (scan.hasNext()) {
            System.out.println(scan.next());
        }
    }

```



## ZSET


```

 /**
     * ZSET 操作
     */
    @Test
    public void redisZsetTest() {
        String key = "redisZsetTest";
        String key2 = "redisZsetTest2";
        ZSetOperations<String, String> ops = clusterTemplate.opsForZSet();
        //ZADD
        ops.add(key, "v1", 1d);
        ops.add(key, "v2", 2d);
        ops.add(key, "v3", 3d);
        ops.add(key, "v4", 4d);
        ops.add(key2, "v1", 1d);
        ops.add(key2, "v3", 3d);
        ops.add(key2, "v5", 5d);
        //ZCARD
        ops.zCard(key);
        //ZCOUNT
        ops.count(key, 1d, 2d);
        //ZINCRBY
        ops.incrementScore(key, "v4", 1.5d);
        //ZLEXCOUNT 字母序
        //ZRANGE 位置
        ops.range(key, 1, 3);
        //ZRANGEBYLEX 字母序
        ops.rangeByLex(key, RedisZSetCommands.Range.range().gt("v1").lte("v2"));
        //ZRANGEBYSCORE  分数
        ops.rangeByScore(key, 1d, 4d);
        //ZRANK 排名
        ops.rank(key, "v1");
        //ZREM
        ops.remove(key, "v1");
        //ZREMRANGEBYLEX
        //ZREMRANGEBYSCORE
        ops.removeRangeByScore(key, 2d, 3d);
        //ZREVRANGE
        ops.reverseRange(key, 0, -1);
        //ZREVRANGEBYSCORE
        ops.removeRangeByScore(key, 1d, 5d);
        //ZREVRANK
        ops.reverseRank(key, "v1");
        //ZSCORE
        ops.score(key, "v1");
        //ZZSCAN
        ops.scan(key, ScanOptions.scanOptions().build());
    }

```


## HyperLogLog

```

  /**
     * 基数统计结构
     */
    @Test
    public void redisHyperLogLogTest() {
        HyperLogLogOperations<String, String> ops = clusterTemplate.opsForHyperLogLog();
        String key = "redisHyperLogLogTestKey";
        String key2 = "redisHyperLogLogTestKey2";
        ops.add(key, "asd");
        ops.add(key2, "asdasd");
        ops.size(key);
        ops.union("redisHyperLogLogTestTemp", key, key2);
    }

```


## 发布订阅

```

 /**
     * 发布
     */
    @Test
    public void redisPublishTest() {
        String channle = "testChannel";
        clusterTemplate.execute((RedisCallback<Object>) connection -> {
            connection.publish(channle.getBytes(), "message1".getBytes());
            return null;
        });

    }

    /**
     * 订阅
     */
    @Test
    public void redisSubscribeTest() {
        String channle = "testChannel";
        clusterTemplate.execute((RedisCallback<Object>) connection -> {
            connection.subscribe((message, pattern) -> {
                System.out.println(new String(message.getBody()));
            }, channle.getBytes());
            return null;
        });
    }


```


----------


## 不支持的命令


```


/**
     * 事务 集群不支持
     * - 批量操作在发送 EXEC 命令前被放入队列缓存。
     * - 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。
     * - 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。
     */
    @Test
    public void redisTransactionTest() {
        clusterTemplate.setEnableTransactionSupport(true);
        //开启事务
        clusterTemplate.multi();
        redisKeyCommandTest();
        redisStringCommandTest();
        //提交事务
        clusterTemplate.exec();
        //取消事务
        //clusterTemplate.discard();
        //监视key
        clusterTemplate.watch("watchKey");
        //取消监视
        clusterTemplate.unwatch();
    }

    /**
     * 脚本
     * 集群不支持脚本
     */
    @Test
    public void redisScriptTest() {
        String lua = "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}";
        DefaultRedisScript<List> script = new DefaultRedisScript<>(lua, List.class);

        String sha1 = script.getSha1();
        System.out.println(sha1);
        System.out.println(clusterTemplate.execute(script, Arrays.asList("key1", "key2"), "arg1", "arg2"));
    }

    private String getRandom() {
        return String.valueOf(Math.random());
    }

    private Map<String, String> getMap() {
        Map<String, String> map = new HashMap<>();
        map.put("a", "1");
        map.put("b", "2");
        map.put("c", "3");
        map.put("d", "4");
        return map;
    }

    @Test
    public void test() {
        clusterTemplate.opsForZSet().intersectAndStore("key1", "key2", "zSetTempKey");

    }

    /**
     * 不支持
     */
    @Test
    public void testTransaction() {
        // 事务
        clusterTemplate.multi();
        redisKeyCommandTest();
        redisStringCommandTest();
        clusterTemplate.exec();
    }

    /**
     * 不支持
     */
    @Test
    public void testMove() {
        clusterTemplate.opsForValue().set("a", "1");
        //move
        clusterTemplate.move("a", 1);
    }

    /**
     * 不支持
     */
    @Test
    public void testScan() {
        //scan
        clusterTemplate.execute((RedisCallback<Object>) connection -> {
            Cursor<byte[]> scan = connection.scan(ScanOptions.scanOptions().build());
            while (scan.hasNext()) {
                System.out.println(scan.next());
            }
            return null;
        });
    }

    /**
     * 不支持
     */
    @Test
    public void testHypLogLogUnion() {
        //hyploglog union
        clusterTemplate.opsForHyperLogLog().union("redisHyperLogLogTestTemp", "key", "key2");
    }

    /**
     * 不支持
     */
    @Test
    public void testZsetInner() {
        //ZINTERSTORE 交集分数相加
        clusterTemplate.opsForZSet().intersectAndStore("key1", "key2", "zSetTempKey");
    }

    /**
     * 不支持
     */
    @Test
    public void testZSetunion() {
        //ZUNIONSTORE
        clusterTemplate.opsForZSet().unionAndStore("key", "key2", "zSetTempKey2");
    }

```