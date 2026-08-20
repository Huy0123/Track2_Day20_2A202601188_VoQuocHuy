# 02 - Continuous batching under load (u50)

Host `Windows-` · `--parallel 4` · 15 samples over
60s at 2.0s intervals · raw CSV: `02-server-metrics-u50.csv`

| Gauge | Peak observed |
|:--|--:|
| `n_busy_slots_per_decode` (avg/decode) | 3.91 of 4 slots (98%) |
| `requests_processing` | 4 |
| `requests_deferred` | 46 |
| `kv_cache_usage_ratio` | n/a — not exported by llama.cpp `b10488` |
| `tokens_predicted_total` (final) | 7884 |

Highest sampled value was **3.91 of 4** slots. Note this gauge is llama.cpp's *average* busy slots per decode step, so the number below is the highest average we sampled, not an instantaneous maximum batch width. A peak near 1 means
requests were served one at a time -- either the load was too light to overlap, or
they arrived too far apart. A peak approaching `--parallel` means the scheduler was
genuinely packing concurrent requests into shared decode steps.
`requests_deferred` went above zero: more requests arrived than there were slots, so some waited. That wait is the queue time in your P95.

## Your observation

Peak batching đạt 3.91/4 slot (98%), đồng thời có 46 request deferred. Effective
concurrency 31.3 cao hơn nhiều vì Little's Law tính cả request đang xếp hàng; nó không
phải số slot thực thi. Tôi dùng gauge 3.91/4 để kết luận slot utilisation và dùng 31.3
để định lượng tổng occupancy gồm cả queue.
