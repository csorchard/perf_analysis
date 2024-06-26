# Golang Performance Analysis
<!-- TOC -->
* [Golang Performance Analysis](#golang-performance-analysis)
  * [Pprof](#pprof)
  * [Git Bisect](#git-bisect)
  * [Heap View](#heap-view)
    * [BurntCarrot HeapView](#burntcarrot-heapview)
    * [Heap Spurr](#heap-spurr)
  * [Stats Viewer](#stats-viewer)
  * [MO](#mo)
<!-- TOC -->

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

## Git Bisect

```shell

git bisect start
git bisect good 8e041398486f167cf53b75b28706107c6d820b3d
git bisect bad 9b39aae4865c60b5e4f9acb8bea4e1bcb5476995

git bisect good
git bisect bad


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

![img.png](img.png)

## MO

1. Hotspot Analysis
![image](https://github.com/csorchard/perf_analysis/assets/9638314/b9f52181-f525-42e2-ae2e-ee279fd81238)

2. Finding statement id
```sql
select * from system.statement_info ssh where `statement` like "%t5%"

select * from system.statement_info si where statement_id  = "018f99d3-414d-7b10-9a57-a8eaa38c7814";

select * from system.statement_info where `statement` like "%SELECT a FROM t5%" order by request_at desc;

```
![image](https://github.com/csorchard/perf_analysis/assets/9638314/cd99cc9f-3e4b-417a-a76a-e8ca3520ecaa)

3. Golang Trace

```shell
curl -o trace.out http://localhost:9876/debug/pprof/trace?seconds=30

go tool trace trace.out

-- go code
_, task := gotrace.NewTask(context.TODO(), "pipeline.Compile")

```

Looks: https://github.com/matrixorigin/matrixone/issues/16124#issuecomment-2122027022

4. GC control

```shell
make debug

GODEBUG=invalidptr=2,cgocheck=2,madvdontneed=1 GOGC=2 GOMEMLIMIT=10MiB ./mo-service -debug-http :9876 -launch ./etc/launch-tae-CN-tae-DN/launch.toml >out.log  2>err.log

```
5. Chucked Heap Profile

https://github.com/arjunsk/go_profile_serde
