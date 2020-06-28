# Ristretto: A High-Performance Go Cache

_Source: [Introducing Ristretto](https://dgraph.io/blog/post/introducing-ristretto-high-perf-go-cache/), [Github](https://github.com/dgraph-io/dgraph)_

It all started with needing a memory-bound, concurrent Go cache in Dgraph. 

Specifically, they were unable to find a cache that met the following requirements:

* Concurrent
* High cache-hit ratio
* Memory-bounded (limit to configurable max memory usage)
* Scale well as the number of cores and goroutines increases
* Scale well under non-random key access distribution (e.g. Zipf).

They assembled a team to build a high-performance cache based on [Caffeine](http://highscalability.com/blog/2016/1/25/design-of-a-modern-cache.html), a high-performance cache written in Java, used by many Java-based databases, like Cassandra, HBase, and Neo4j.

## Usage

Full documentation can be found on [Github](https://github.com/dgraph-io/dgraph).

### Example

```go
func main() {
	cache, err := ristretto.NewCache(&ristretto.Config{
		NumCounters: 1e7,     // number of keys to track frequency of (10M).
		MaxCost:     1 << 30, // maximum cost of cache (1GB).
		BufferItems: 64,      // number of keys per Get buffer.
	})
	if err != nil {
		panic(err)
	}

	// set a value with a cost of 1
	cache.Set("key", "value", 1)
	
	// wait for value to pass through buffers
	time.Sleep(10 * time.Millisecond)

	value, found := cache.Get("key")
	if !found {
		panic("missing value")
	}
	fmt.Println(value)
	cache.Del("key")
}
```
