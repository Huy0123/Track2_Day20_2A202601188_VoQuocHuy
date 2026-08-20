# 03 - Integrate: RAG pipeline run

Host `Windows-` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.7 | 12186.2 | 12187.0 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.1 | 3054.0 | 3054.0 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.0 | 3029.0 | 3029.1 |

Mean per stage (ms): embed **0.0** · retrieve **0.3** ·
llm **6089.7** · total **6090.0**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Goodput@SLO counts only the requests per second that met the TTFT and TPOT targets. Throughput at saturation ignores SLOs.

**What problem does PagedAttention actually solve?**

> PagedAttention stores the KV cache in non-contiguous pages, removing the internal fragmentation that wasted most GPU memory.

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps because prefill is compute-bound and decode is memory-bandwidth-bound.


## Which N16-N19 pieces are real

- N16 Cloud/IaC: stub, chạy localhost.
- N17 data pipeline: stub, danh sách in-memory.
- N18 lakehouse: stub, dữ liệu toy trong code.
- N19 vector/features: stub, keyword overlap; không có embedding server.
- N20 serving: real, llama-server xử lý đủ ba query.

LLM chiếm 6089.7/6090.0 ms trung bình (xấp xỉ 100%), đúng kỳ vọng vì retrieval toy
chỉ mất 0.3 ms. Muốn giảm tổng latency 2×, tôi sẽ tối ưu decode/model serving trước;
tối ưu retrieval gần như không thay đổi end-to-end latency.
