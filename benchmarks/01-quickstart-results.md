# 01 - Measure: latency baseline

Model `Gemma 4 E2B` · host `Windows-` · llama.cpp `b10488`
Settings: `threads=12` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `UD-Q4_K_XL` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 7187 | 160 / 441 | 26.4 / 26.5 | 1807 / 2110 / 2110 | 37.8 |
| UD-Q2_K_XL | 2.24 | 6455 | 157 / 295 | 26.7 / 26.9 | 1834 / 1977 / 1977 | 37.4 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` and `UD-Q4_K_XL` decode within 2% of each other here, for 0.73 GB difference on disk.

## Your observation

Q2 tiết kiệm 0.73 GB (24.6%) nhưng decode chậm hơn nhẹ: 37.4 so với 37.8 tok/s.
Vì không có speedup và chất lượng câu trả lời chưa được kiểm chứng trực quan bằng
cùng một prompt, tôi giữ Q4 làm lựa chọn an toàn thay vì đánh đổi precision chỉ để
giảm dung lượng.
