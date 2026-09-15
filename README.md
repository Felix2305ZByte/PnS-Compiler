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
