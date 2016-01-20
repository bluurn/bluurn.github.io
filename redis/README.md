class: center, middle
# Redis::Recipes
![redis logo](http://download.redis.io/logocontest/88.png)

---
### Redis?

* Key -> Data Structure storage
* RAM + Persistence layer
* Fastness!

---
### Структуры данных

* Scalar: Strings, Blobs, Bitmaps
* Hash Tables: {k1 v1 k2 v2}
* Linked Lists: '(A B C B C C A)
* Sets: #{A B C D}
* Sorted Sets: #{'(A 1) '(B 7) '(C 11) '(D 10)}

---
class: center, middle
## Примеры использования

---
### Case 0: Кеширование запросов к БД

``` ruby
%w{digest redis}.each { |l| require l }

def cached_query query
  redis = Redis.new
  digested = Digest::MD5.hexdigest query
  result = redis.get digested     
  if result.nil?
    result = db.execute sql
    redis.set digested, result
  end
  result
end
```

---
### Case 1: Хранение очередей

``` ruby
class RQueue
  def initialize redis
    local_id = r.incr "rq_space"
    @redis, @id_name = redis, "queue:#{local_id}"
  end 
  def push elt
    @redis.lpush @id_name, elt 
  end
  def pop
    @redis.rpop @id_name
  end
end
```

---
### Case 2: Определение общего

``` redis
SADD tags:article:1 politics economics obama
SADD tags:article:2 obama politics usa

SINTER tags:article:1 tags:article:2
```

---
### Case 3: Список с сортировкой по полю

``` redis
HMSET task:1 priority 4 description "Помыть посуду"
HMSET task:2 priority 1 description "Купить молока"
HMSET task:3 priority 3 description "Посмотреть сериальчик"
HMSET task:4 priority 2 description "Смыть с топора кровь"
HMSET task:5 priority 5 description "Саша Грей"

SADD user:1:tasks 1 2 3 4

SORT user:1:tasks BY task:*->priority GET task:*->description
```

---
### Case 4: Удаление записей по шаблону

``` lua
for _,k in ipairs(redis.call('keys','abonents:[^7]*')) do
  redis.call('del',k)
end
```

``` shell
#!/bin/sh

redis-cli eval "$(cat remove_bad_abonents.lua)" 0
```

---
### Case 4b: Корректное удаление записей

``` ruby
def perform
  @client.scan_each(:match => ABONENTS_GLOB).each do |item|
    msisdn = item[/abonents:(.*)/, 1]
    unless msisdn[0].to_i == VALID_MSISDN_CC || 
           msisdn.length == VALID_MSISDN_CC
      @client.del item
    end
  end
end
``` 

---
### Case 5: Склейка PDU, готовим данные:

``` redis
MULTI
  RPUSH  "$pdu.key" $pdu
  ZADD  times $now “$pdu.key/$pdu.nparts" 
EXEC
```

``` erl
% key
{29,"ACME","79374577606",47} 
```

---
### Case 5: Склейка PDU, процесс:

* Периодически выполняем:

``` redis
ZRANGEBYSCORE times -inf $threshold_time LIMIT 0 1
```

* Для каждого $key:

``` redis
MULTI
  RENAME $key "$key-$pid"
  ZREM times $key
  ZADD flushes $now "$key-$pid"
  LRANGE "$key-$pid" 0 $nparts
EXEC
```

---
### Case 5: Склейка PDU, GC

* Периодически выполняем:

``` redis
ZRANGEBYSCORE flushes -inf $threshold_time LIMIT 0 100
```

* Для каждого $key:

``` redis
MULTI
  RENAME "$key-$pid" $key
  ZADD times $threshold_time "$key" 
EXEC

```
