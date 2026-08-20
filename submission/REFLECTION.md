# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Võ Quốc Huy
**Cohort:** K4
**Ngày submit:** 2026-08-20

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** Windows (release 10 theo `hardware.json`)
- **CPU:** 12th Gen Intel Core i5-1240P
- **Cores:** 12 physical / 16 logical
- **CPU extensions:** không được hardware probe xuất ra
- **RAM:** 15.7 GB
- **Accelerator:** NVIDIA GeForce MX570 2 GB, CUDA/Vulkan
- **llama.cpp asset đã tải:** `llama-b10488-bin-win-cuda-12.4-x64.zip`
- **Model đã dùng:** Gemma 4 E2B (`LAB_MODEL=gemma4-e2b`)
- **Quantization:** UD-Q4_K_XL + UD-Q2_K_XL

**Chạy ở đâu:** laptop của tôi
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

Setup local dùng Python 3.11 và runtime CUDA b10488. Hugging Face bị từ chối trong
sandbox nên download được chạy lại qua kết nối máy thật. Windows PowerShell 5.1 còn
lỗi UTF-8: runner không parse em dash và Python stdout dùng cp1252; tôi đổi help text
sang ASCII và đặt `PYTHONUTF8=1` trong `lab.ps1`.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 7187 | 160 / 441 | 26.4 / 26.5 | 1807 / 2110 / 2110 | 37.8 |
| UD-Q2_K_XL | 2.24 | 6455 | 157 / 295 | 26.7 / 26.9 | 1834 / 1977 / 1977 | 37.4 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

Q2 nhỏ hơn 0.73 GB (24.6%) nhưng không nhanh hơn: 37.4 so với 37.8 tok/s. Tôi chưa
đánh giá cùng prompt bằng mắt, nên không tuyên bố khác biệt chất lượng. Với kết quả
hiện có, Q4 đáng giữ hơn vì không cần đánh đổi precision cho một speedup không tồn tại.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 1.38 | 6100 | 8000 | 9400 | 8.5 | 0.0% |
| 50 | 1.19 | 27000 | 42000 | 43000 | 31.3 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** 0.86×
- **P95 tăng:** 5.25×
- **Effective concurrency ở 50 users:** 31.3 so với `--parallel` = 4 slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): 3.91 / 4 slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

Server bão hòa trước hoặc tại 50 users: tải tăng 5× nhưng RPS giảm còn 0.86×, P95
tăng 5.25× lên 42 s. Gauge đạt 3.91/4 slot và 46 request deferred, nên latency thêm
là queue time. Tôi sẽ tăng năng lực decode hoặc thêm replica trước, không tăng client
concurrency vì bốn slot đã gần kín.

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | localhost | stub |
| N17 Data pipeline | danh sách in-memory | stub |
| N18 Lakehouse | toy data trong code | stub |
| N19 Vector + features | keyword overlap | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: 0.0 ms
- retrieve: 0.3 ms
- llm: 6089.7 ms
- **stage chiếm nhiều nhất:** llm (100% của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

LLM là bottleneck với 6089.7/6090.0 ms trung bình, đúng kỳ vọng vì retrieval toy chỉ
mất 0.3 ms. Muốn giảm pipeline 2×, tôi sẽ tối ưu model serving/decode; tối ưu embed
hoặc retrieve gần như không tác động end-to-end latency trong cấu hình này.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** giảm `-t` từ 16 về mặc định 12 khi `ngl=99`

```
before:  39.1 tok/s (-t 16)
after:   39.5 tok/s (-t 12)
speedup: 1.01×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

Kết quả quan trọng không phải speedup 1.01× mà là đường cong gần như phẳng: từ 1 đến
32 thread chỉ dao động 39.1–39.5 tok/s. Điều này khác kỳ vọng CPU-only, nơi throughput
thường đạt knee quanh physical cores rồi giảm do tranh memory bandwidth.

Ở đây `ngl=99` offload phần lớn model sang CUDA, nên decode chủ yếu bị giới hạn bởi
GPU và đường truyền bộ nhớ; CPU thread không còn là knob chi phối. Tôi giữ 12 thread
vì đạt mức tốt nhất mà không oversubscribe CPU. Muốn cải thiện lớn hơn phải thay đổi
serving/GPU path, không tiếp tục tăng thread.

---

## 6. Bonus  *(optional — tối đa 20 điểm)*

> Bỏ trống nếu không làm. Xem `bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** _<B1 build-compare / B2 sweep nào / B4 challenge nào / B5 lựa chọn nào>_

**Numbers:**

```
before:  <số>
after:   <số>
speedup: <X.Y>×
```

**Điều này nói lên gì mà deck chưa nói:**

_(để trống nếu bạn không làm phần này)_

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

_(để trống nếu bạn không làm phần này)_

---

## 8. Self-check trước khi push

- [ ] `hardware.json` committed
- [ ] `models/active.json` committed
- [ ] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [ ] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [ ] `benchmarks/02-server-results.md` committed (`make load-report`)
- [ ] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [ ] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [ ] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [ ] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [ ] 5 screenshots trong `submission/screenshots/`
- [ ] `make verify` → **exit 0**
- [ ] Repo GitHub ở chế độ **public**
- [ ] Đã paste public URL vào VinUni LMS
- [ ] **Không** commit `models/*.gguf` hay `runtime/` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.
