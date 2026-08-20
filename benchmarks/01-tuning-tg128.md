# 01 - Tune: thread-count sweep

Model `gemma-4-E2B-it-UD-Q4_K_XL.gguf` · host `Windows-` · llama.cpp `b10488`
CPU: **12 physical · 16 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 39.5 | 100% |
| 6 | 39.3 | 100% |
| 12 | 39.5 | 100% |
| 16 | 39.1 | 99% |
| 32 | 39.5 | 100% |

**Best**: `-t 12` at 39.5 tok/s
**Slowest tested**: `-t 16` at 39.1 tok/s (1.01x spread)
**Against the physical-core default** (`-t 12`, 39.5 tok/s): 1.00x

Use this in your run:

```bash
LAB_N_THREADS=12 make bench
```

## Your explanation

Đường cong gần như phẳng: 1–32 thread chỉ dao động 39.1–39.5 tok/s (spread 1.01×),
không có knee CPU rõ ràng. Với `ngl=99`, phần lớn phép tính đã offload sang CUDA;
decode bị giới hạn bởi đường GPU/memory bandwidth nên thêm CPU thread không cải thiện.
Tôi giữ mặc định 12 thread vì đạt mức tốt nhất mà không oversubscribe CPU.
