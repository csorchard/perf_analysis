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
1. Setup code
```go
        defer func() {
            f, err := os.Create("heapdump.out")
            if err != nil {
                panic("Could not open file for writing:" + err.Error())
            } else {
                runtime.GC()
                debug.WriteHeapDump(f.Fd())
                f.Close()
            }
            os.Exit(1)
        }()
```
2. Download [heapview](https://github.com/burntcarrot/heapview). Build source using `go build`.
3. Run `./heapview -file=/Users/arjunsunilkumar/GolandProjects/matrixone/heapdump.out`
4. Go to `http://localhost:8080/`

### Heap Spurr
1. Build code `go build ./cmd/heapspurs`
2. Run to print all the pointers
```shell
./heapspurs /Users/arjunsunilkumar/GolandProjects/matrixone/heapdump.out --print;

Object @ 0x14000192b40 with 2 pointers in 48 bytes
  Pointer[0]@0x14000192b40 = 0x1079df577
  Pointer[1]@0x14000192b58 = 0x1079bef59
Object @ 0x14000192b70 with 3 pointers in 48 bytes
  Pointer[0]@0x14000192b80 = 0x14000621b00
```
3. Run to build a graph `./heapspurs /Users/arjunsunilkumar/GolandProjects/matrixone/heapdump.out --address 0x1079df577`. Image name `heapdump.svg`.

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
