# 🚀 PnS++ Language (Probabilistic & Statistical Language)

**PnS++** là một ngôn ngữ lập trình chuyên biệt (Domain-Specific Language - DSL) hiệu năng cao được thiết kế dành riêng cho mô phỏng xác suất (Monte Carlo), phân tích thống kê và tính toán mảng dữ liệu quy mô lớn. 

PnS++ giải quyết bài toán giao thoa giữa **cú pháp tối giản của Python** và **tốc độ thực thi tiệm cận C/C++**, giúp các nhà nghiên cứu và kỹ sư dữ liệu biểu diễn bài toán phức tạp chỉ trong vài dòng code mà vẫn đạt hiệu năng xử lý tối đa của phần cứng.

---

## 🌟 Triết lý thiết kế (Core Design Principles)

* **Expressive & Minimalist Syntax:** Cú pháp hướng khai báo (declarative) giúp biểu diễn các phép toán mảng và phân phối thống kê một cách tự nhiên.
* **JIT Native Execution:** Không sử dụng Python Interpreter cho vòng lặp tính toán. Mã nguồn PnS++ được phân tích AST và hạ cấp (lower) trực tiếp xuống mã máy Native (x86_64) thông qua backend LLVM JIT Compiler.
* **Auto-Hardware Acceleration:** Tự động kích hoạt Vectorization (SIMD) và tính toán đa luồng (OpenMP) mà người dùng không cần quản lý bộ nhớ hay viết code song song thủ công.

---

## 🏗️ Kiến trúc Compiler (Architecture Overview)

```text
[ File Mã Nguồn (.pspp) ]
          │
          ▼
   ┌──────────────┐
   │ Lark Parser  │  ==> Phân tích cú pháp EBNF & dựng cây AST
   └──────┬───────┘
          │
          ▼
   ┌──────────────┐
   │ AST Codegen  │  ==> Tự động phân giải Module (@stats, @math) & sinh IR
   └──────┬───────┘
          │
          ▼
   ┌──────────────┐
   │ Numba JIT    │  ==> LLVM Optimization: Auto-SIMD / OpenMP Parallel
   └──────┬───────┘
          │
          ▼
 [ Native Machine Code ] (Thực thi siêu tốc trên CPU)

```

---

## 💻 Cú pháp minh họa (Syntax Preview)

```pspp
# Sinh 10 triệu mẫu ngẫu nhiên theo phân phối chuẩn N(0, 1)
data = @stats.norm(mean=0.0, std=1.0, n=10000000)

# Lọc dữ liệu theo điều kiện (Vectorized Filter)
filtered = data > 2.0

# Tính trung bình xác suất P(X > 2.0)
result = @stats.mean(filtered)

# Xuất kết quả
out result

```

---

## 🛠️ Yêu cầu môi trường (Prerequisites)

* **Python:** 3.9+
* **Các thư viện phụ thuộc:**
* `lark` (Cú pháp EBNF Parser)
* `numba` (LLVM JIT Backend)
* `numpy` (Cấu trúc dữ liệu mảng nền tảng)



---

## 🚀 Hướng dẫn nhanh (Quick Start)

### 1. Cài đặt các thư viện phụ thuộc

```bash
pip install lark numba numpy

```

### 2. Thực thi chương trình PnS++

```bash
python pns_cli.py run demo.pspp

```

---

## 🤝 Đóng góp phát triển (Contributing)

Mọi đóng góp cho dự án PnS++ đều được hoan nghênh! Bạn có thể đóng góp bằng cách:

* Mở **Issue** để báo lỗi hoặc đề xuất tính năng mới.
* Gửi **Pull Request** để cải thiện Lark Grammar hoặc thêm hàm mới cho module `@stats` / `@math`.

---

## 📜 Giấy phép (License)

Dự án được phân phối theo giấy phép **MIT License**. Chi tiết xem tại file [LICENSE](https://www.google.com/search?q=LICENSE).

---

## 👤 Tác giả (Author)

* **Felix2305Zbyte** - *PnS++ Core Architect*
