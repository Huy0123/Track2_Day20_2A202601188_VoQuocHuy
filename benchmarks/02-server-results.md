# 02 - Serve: load test + saturation reading

Host `Windows-` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=12` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 81 | 1.38 | 6100 | 8000 | 9400 | 8.5 | 0.0% |
| 50 | 69 | 1.19 | 27000 | 42000 | 43000 | 31.3 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **0.86x** (17% of linear) |
| P95 latency | **5.25x** |
| Effective concurrency at 50 users | 31.3 vs `--parallel 4` slots (occupancy/slot ratio 7.83) |

**Saturated.** Throughput delivered only 0.86x for 5x the offered load, and effective concurrency (31.3) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 0.86x while P95 moved 5.25x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Your reading

Server đã bão hòa trước hoặc tại 50 users: tải tăng 5× nhưng throughput giảm từ 1.38
xuống 1.19 RPS, trong khi P95 tăng 5.25× lên 42 s. Gauge 3.91/4 slot và 46 request
deferred chứng minh phần tăng thêm là queue time. Để tăng goodput@SLO, tôi ưu tiên
tăng năng lực decode/model replica thay vì thêm client concurrency, vì slot hiện đã kín.
