# Golang Performance Analysis

## Pprof

```shell
curl -o cpu.pprof "http://localhost:9876/debug/pprof/profile?seconds=30";

curl -o heap.pprof "http://localhost:9876/debug/pprof/heap?seconds=30"

curl -o goroutine.pprof "http://localhost:9876/debug/pprof/goroutine?seconds=30"

curl -o block.pprof "http://localhost:9876/debug/pprof/block?seconds=30"

curl -o mutex.pprof "http://localhost:9876/debug/pprof/mutex?seconds=30"

curl -o threadcreate.pprof "http://localhost:9876/debug/pprof/threadcreate?seconds=30"

curl -o allocs.pprof "http://localhost:9876/debug/pprof/allocs?seconds=30"

go tool pprof -http=:8080 heap.pprof
```

## Heap View

### BurntCarrot HeapView

1. Download [heapview](https://github.com/burntcarrot/heapview). Build source using `go build`.
2. Run `./heapview -file=/Users/arjunsunilkumar/GolandProjects/matrixone/heapdump.out`
3. Go to `http://localhost:8080/`

### Heap Spurr

## Stats Viewer

1. Setup code
```go
package main

import (
    "time"

    "github.com/go-echarts/statsview"
)

func main() {
	mgr := statsview.New()

	// Start() runs a HTTP server at `localhost:18066` by default.
	go mgr.Start()

	// Stop() will shutdown the http server gracefully
	// mgr.Stop()

	// busy working....
	time.Sleep(time.Minute)
}

```

2. Go to http://localhost:18066/debug/statsview
