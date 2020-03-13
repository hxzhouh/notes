# 基于redis 的延时队列

``` go


package main

import (
   "fmt"
   "github.com/go-redis/redis"
   "strconv"
   "sync"
   "time"
)


var client *redis.Client

// 使用lua 保证原子性
var zrangebyscoreExpireScript = redis.NewScript(`
local cnt  = redis.call("ZRANGEBYSCORE",KEYS[1],ARGV[1],ARGV[2],ARGV[3])
if not cnt then
   return string.format("hget not found: %d",ARGV[1])
else
   redis.call("ZREMRANGEBYSCORE",KEYS[1],ARGV[1],ARGV[2])
   return cnt
end
`)


//
func main() {
   wg := sync.WaitGroup{}
   client = createClient()
   wg.Add(1)
   go func(wg *sync.WaitGroup) {
      defer wg.Done()
      ticker1 := time.Tick(1 * time.Second)
      for {
         select {
         case <-ticker1:
            result, err := FindMemberByScores("salary1", -1, time.Now().Unix()-1)
            if err != nil {
               fmt.Println(err)
            }
            if value, ok := result.([]interface{}); ok {
               for k, v := range value {
                  if k%2 == 0 {
                     fmt.Println(fmt.Sprintf("now:%v,value:%v", time.Now().Unix(), v.(string)))
                  }
               }
            } else {
               fmt.Println(result)
            }
         }
      }
   }(&wg)


   wg.Add(1)
   go func(wg *sync.WaitGroup) {
      defer wg.Done()
      ticker1 := time.Tick(1 * time.Second)
      for {
         select {
         case <-ticker1:
            SaddValue("salary1", time.Now().Unix()+5, time.Now().Unix())
         }
      }
   }(&wg)
   wg.Wait()
}


func SaddValue(key string, socre int64, member interface{}) error {
   result := client.ZAdd(key, redis.Z{
      Score:  float64(socre),
      Member: member,
   })
   return result.Err()
}
func SaddValues(keys string, value ...redis.Z) {
   client.ZAdd(keys, value...)
}
func FindMemberByScores(key string, min, max int64) (interface{}, error) {
   var minString, maxString string
   if min == -1 {
      minString = "-inf"
   } else {
      minString = strconv.FormatInt(min, 10)
   }
   if max == -1 {
      maxString = "+inf"
   } else {
      maxString = strconv.FormatInt(max, 10)
   }
   result, err := zrangebyscoreExpireScript.Run(client, []string{key}, []string{minString, maxString, "WITHSCORES"}).Result()
   if err != nil {
      return nil, err
   }
   return result, nil
}
func createClient() *redis.Client {
   client := redis.NewClient(&redis.Options{
      Addr:     "127.0.0.1:6379",
      Password: "",
      DB:       0,
   })
   // 通过 cient.Ping() 来检查是否成功连接到了 redis 服务器
   pong, err := client.Ping().Result()
   fmt.Println(pong, err)
   return client
}

```

