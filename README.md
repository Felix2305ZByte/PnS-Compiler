# 🚀 PnS++ — Probabilistic & Statistical Language

**PnS++** là ngôn ngữ chuyên biệt (DSL) cho **mô phỏng Monte Carlo và thống kê trên mảng dữ liệu lớn**.
Cú pháp gọn như Python, nhưng chương trình được **biên dịch thẳng ra mã máy** bằng LLVM và chạy song song trên mọi nhân CPU.

![version](https://img.shields.io/badge/version-0.2.0-blue) ![license](https://img.shields.io/badge/license-MIT-green) ![status](https://img.shields.io/badge/status-experimental-orange)

```pspp
// Định giá quyền chọn Monte Carlo trên 10 triệu mẫu
stats.seed(2026)
z  = stats.norm(0, 1, 10_000_000)
st = 100 * math.exp((0.03 - 0.5 * 0.2 ** 2) + 0.2 * z)
out "giá ≈", math.exp(-0.03) * mean(math.fmax(st - 105, 0))
```

> **Trạng thái:** v0.2.0, giai đoạn thử nghiệm. Có phép gán, `out`, `import`, toán tử số học / so sánh / logic và thư viện **140 hàm**.
> Chưa có `if`, vòng lặp hay hàm tự định nghĩa.

---

## 🌟 Điểm nổi bật

- **Cú pháp tối giản, hướng khai báo.** Viết `mean(x > 1.96)` thay vì vòng lặp; không cần quản lý bộ nhớ hay viết code song song.
- **Sinh mã máy trực tiếp.** Với mỗi reducer (`mean`, `std`, `quantile`, ...), compiler sinh LLVM IR cho *cả chuỗi biểu thức*,
  để LLVM tối ưu và vector hóa (SIMD) theo đúng CPU đang chạy. Mã máy được cache trên đĩa nên lần chạy sau bỏ qua bước biên dịch.
- **Gộp vòng lặp (loop fusion).** `x*x + y*y < 1` hay `math.exp(-x*x)` nằm trong *một* vòng lặp, xử lý theo khối nhỏ trong cache L1, không có mảng trung gian.
- **Vector là một "công thức", không phải mảng.** `x = stats.norm(...)` chưa tính gì; 10 triệu mẫu không tốn 80 MB.
- **Lặp lại được 100%.** Cùng seed cho cùng kết quả, **kể cả khi đổi số luồng**.
- **Đa nền tảng.** Runtime C99 có bản dựng sẵn cho Windows / Linux / macOS (x86_64 và ARM); thiếu thì tự biên dịch bằng gcc / clang / zig.

## 🏗️ Kiến trúc

```text
 file .pspp
     │
     ▼
 Parser (viết tay)  →  cây cú pháp (AST)
     │
     ▼
 Resolver           →  kiểm tra import / tên hàm trước khi chạy
     │
     ▼
 Interpreter        →  số: tính ngay;  vector: dựng đồ thị biểu thức lười
     │  gặp reducer (mean, std, quantile, ...)
     ▼
 Codegen LLVM IR    →  MỘT hàm kernel cho cả biểu thức (khối 65 536 mẫu)
     │
     ▼
 LLVM (llvmlite)    →  tối ưu + vector hóa theo CPU + sinh MÃ MÁY (cache trên đĩa)
     │
     ▼
 pns_rt (C99)       →  chia khối cho các luồng; RNG, phân phối, gộp thống kê
```

## 💻 Cú pháp

```pspp
import stats, prob              // không bắt buộc: stats.norm(...) hay norm(...) đều chạy
import math as m
from stats import mean, std as sd

stats.seed(42)
x = stats.norm(mean=0, std=1, n=10_000_000)
out "P(X > 1.96) =", mean(x > 1.96)
out "lý thuyết    =", 1 - prob.norm_cdf(1.96)
out sd(x), m.pi
```

- Comment: `//` và `/* ... */`. Số: `10_000_000`, `1.96`, `2.5e-3`.
- Toán tử (thấp → cao): `or` · `and` · `not` · `> < >= <= == !=` · `+ -` · `* / %` · `-x` · `**`.
- Hai vector cùng độ dài tính được với nhau: `x*x + y*y < 1`, `where(x > 0, x, -x)`.

## 📚 Thư viện (140 hàm)

| Module | Nội dung |
| :--- | :--- |
| `stats` | sinh dữ liệu (`norm`, `uniform`, `poisson`, `gamma`, ...), thống kê (`mean`, `std`, `quantile`, `corr`, ...), suy diễn (`tstat`, `ttest_p`, `ci_lo`, `slope`, `r2`, ...) |
| `math` | `sqrt`, `exp`, `log`, lượng giác, `erf`, `gamma`, `where`, `clip`, `comb`, ... |
| `prob` | pdf / pmf / cdf / ppf của các phân phối phổ biến (chuẩn, mũ, gamma, beta, t, nhị thức, Poisson, ...) |
| `io` | `io.load("file.csv", col=0)` đọc dữ liệu từ tệp |

Danh sách đầy đủ có mô tả: `python pns_cli.py help` (hoặc `help stats`, `help math`, `help prob`, `help io`).

## 🛠️ Cài đặt & chạy

Yêu cầu: **Python 3.9+** và **llvmlite**.

```bash
pip install -r requirements.txt
python pns_cli.py selftest                       # kiểm tra runtime + backend mã máy
python pns_cli.py run demo.pspp
python pns_cli.py run demo_option.pspp --seed 1 --threads 4
python pns_cli.py run demo.pspp --emit-asm       # xem mã máy được sinh ra
```

Tùy chọn của `run`: `--seed N`, `--threads N`, `--profile`, `--emit-ir`, `--emit-asm`, `--no-cache`.

Test và đo tốc độ:

```bash
pip install -r requirements-dev.txt
pytest -q
python benchmark.py
python duel.py          # đấu 1v1 với C++ (cần g++ / clang++ / cl)
```

## 📊 Tốc độ

10 triệu mẫu, đo trên Linux 2 nhân, g++ 13. Số trên máy bạn sẽ khác, hãy chạy `benchmark.py`.

| Bài toán | NumPy 1 luồng | PnS++ 1 luồng | PnS++ 2 luồng |
| :--- | ---: | ---: | ---: |
| P(X > 2): norm → so sánh → mean | 153 ms | 42 ms | 22 ms |
| Ước lượng π (5 triệu cặp) | 96 ms | 21 ms | 11 ms |
| mean + std của N(0,1) | 189 ms | 48 ms | 25 ms |
| E[exp(−x²)] trên lưới | 98 ms | 56 ms | 34 ms |

Đấu với C++ (P(X > 2), 1 luồng): PnS++ 40.9 ms, C++ cùng thuật toán 43.3 ms, C++ thông thường (`mt19937` + `std::normal_distribution`) 119.6 ms.

## ⚠️ Hạn chế đã biết

- Mã máy được JIT trong tiến trình Python; chưa có chế độ biên dịch trước (AOT) ra tệp thực thi.
- Khởi động còn ~100 ms, nên chương trình rất nhỏ vẫn chậm hơn C++ thuần.
- Chưa có `if`, vòng lặp, hàm tự định nghĩa.
- `median`, `quantile`, `iqr`, `mad` cần lưu mảng thật.
- `sum` / `mean` số thực có thể lệch vài bit cuối giữa các CPU khác nhau (cùng một máy thì luôn giống nhau).
- Chuỗi số ngẫu nhiên của v0.2 khác v0.1.x với cùng seed.

## 🤝 Đóng góp

Mọi đóng góp đều được hoan nghênh: mở **Issue** để báo lỗi hoặc đề xuất tính năng, gửi **Pull Request** để thêm hàm mới
(`pns/registry.py`) hoặc cải thiện ngữ pháp (`pns/parser.py`, `pns/pspp.lark`). Chạy `pytest -q` trước khi gửi.

## 📜 Giấy phép

Phân phối theo [MIT License](LICENSE).

## 👤 Tác giả

**Felix2305Zbyte** — *PnS++ Core Architect*
