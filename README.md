golang-lru
==========

This provides fork of the [hashicorp lru package](https://github.com/hashicorp/golang-lru) which implements a fixed-size thread safe LRU cache. Original is based on the cache in Groupcache.

Documentation
=============

Full docs are available on [Godoc](https://godoc.org/github.com/paskal/golang-lru)

LRU cache example
=======

```go
package main

import (
	"fmt"

	"github.com/paskal/golang-lru"
)

func main() {
	l, _ := lru.New(128)
	for i := 0; i < 256; i++ {
		l.Add(i, nil)
	}

	if l.Len() != 128 {
		panic(fmt.Sprintf("bad len: %v", l.Len()))
	}
}
```

Expirable LRU cache example
=======

```go
package main

import (
	"fmt"
	"time"

	"github.com/paskal/golang-lru/simplelru"
)

func main() {
	// make cache with short TTL and 3 max keys, purgeEvery time.Millisecond * 10
	cache := NewExpirableLRU(3, nil, time.Millisecond*5, time.Millisecond*10)
	// expirable cache need to be closed after used
	defer cache.Close()

	// set value under key1.
	cache.Add("key1", "val1")

	// get value under key1
	r, ok := cache.Get("key1")

	// check for OK value, because otherwise return would be nil and
	// type conversion will panic
	if ok {
		rstr := r.(string) // convert cached value from interface{} to real type
		fmt.Printf("value before expiration is found: %v, value: %v\n", ok, rstr)
	}

	time.Sleep(time.Millisecond * 11)

	// get value under key1 after key expiration
	r, ok = cache.Get("key1")
	// don't convert to string as with ok == false value would be nil
	fmt.Printf("value after expiration is found: %v, value: %v\n", ok, r)

	// set value under key2, would evict old entry because it is already expired.
	cache.Add("key2", "val2")

	fmt.Printf("Cache len: %d", cache.Len())
	// Output:
	// value before expiration is found: true, value: val1
	// value after expiration is found: false, value: <nil>
	// Cache len: 1
}
```
