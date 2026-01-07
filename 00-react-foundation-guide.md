# 📘 SUMMARY: REACT FOUNDATION & MINDSET (SESSION 0)

Chào mừng bạn đến với ReactJS Bootcamp. Đây là tài liệu tóm tắt các kiến thức cốt lõi của Session 0.

---

## 1. TƯ DUY CỐT LÕI: TẠI SAO DÙNG REACT?

Sự khác biệt lớn nhất giữa cách viết web truyền thống (Vanilla JS) và React nằm ở **tư duy lập trình**.

| Đặc điểm | Vanilla JS (Cũ) | React (Mới) |
| :--- | :--- | :--- |
| **Phong cách** | **Imperative (Mệnh lệnh)** | **Declarative (Khai báo)** |
| **Cách làm** | Bạn chỉ dẫn từng bước: *"Tìm nút ID này, sửa text thành X, đổi màu thành đỏ..."* | Bạn mô tả kết quả: *"Khi dữ liệu là X, giao diện sẽ trông như thế này."* |
| **Tương tác DOM** | Tự tay thao tác (`document.getElementById`) | **React tự động cập nhật DOM** |
| **Hệ quả** | Dễ sót trường hợp, khó quản lý khi app lớn | Code dễ đoán, ít lỗi logic, dễ bảo trì |

> **💡 Nguyên tắc:** Trong React, bạn **KHÔNG** chạm trực tiếp vào DOM. Bạn thay đổi **State**, React sẽ thay đổi giao diện.


---

## 2. BẢN ĐỒ TƯ DUY (THE MENTAL MODEL) 🔥

Đây là kiến thức quan trọng. Hãy ghi nhớ vòng lặp này:


### Công thức: `State → Render → UI`

1. **STATE (Dữ liệu):** Nơi chứa "sự thật" (Source of Truth). Ví dụ: `count = 0`.
2. **RENDER (Tính toán):** React chạy Component để tính toán giao diện dựa trên State hiện tại.
3. **UI (Hiển thị):** Kết quả hiển thị lên màn hình.
4. **EVENT (Tương tác):** User click/type → gọi hàm update State → **Quay lại bước 1**.

> **⚠️ Quy tắc vàng:**
> * UI là **hệ quả** của State.
> * Muốn UI thay đổi? **Hãy thay đổi State.**
> 
> 

---

## 3. CÁC KHÁI NIỆM NỀN TẢNG

### A. Component là gì?

Trong React hiện đại, định nghĩa Component rất đơn giản:

> **Component = Một hàm JavaScript (Function) trả về JSX.**

**3 Đặc điểm nhận dạng:**

1. Tên hàm phải **Viết Hoa Chữ Cái Đầu** (VD: `Header`, `MyButton`).
2. Phải trả về (`return`) JSX.
3. Hoạt động độc lập, có thể tái sử dụng (Reusable).

```tsx
// Ví dụ một Component hợp lệ
function MyButton() {
  return <button>Click me</button>;
}
```

**Cây Component (Component Tree):**

```text
App
├─ Header
└─ Main
   ├─ Sidebar
   └─ Content
      ├─ Post
      └─ Post
```

- `App` là “rễ cây”, chứa các component con.  
- Mỗi component lại có thể chứa thêm component con nữa.  
- Một component (ví dụ: `Post`) có thể được dùng nhiều lần với dữ liệu khác nhau.

### B. JSX (JavaScript XML)

JSX trông giống HTML nhưng thực chất là **JavaScript**. Vì vậy nó có quy tắc riêng:

**✅ 3 Điểm khác biệt BẮT BUỘC nhớ:**

1. **Class:**
   * ❌ HTML: `class="container"`  
   * ✅ JSX: `className="container"` (Vì `class` là từ khóa của JS).

2. **Đóng thẻ:**
   * ❌ HTML: `<img src="...">`  
   * ✅ JSX: `<img src="..." />` (Thẻ đơn **bắt buộc** phải có dấu đóng `/`).

3. **Nhúng JavaScript:**
   * Dùng cặp ngoặc nhọn `{ }` để viết **biểu thức** JS bên trong JSX.  
   * Ví dụ: `<h1>Xin chào {userName}</h1>` hoặc `{age >= 18 ? "Đủ tuổi" : "Chưa đủ"}`.

> **Lưu ý:** Component chỉ được return **1 phần tử cha**.  
> Để tránh tạo thẻ `div` thừa, hãy dùng **Fragment**: `<> ... </>`.

### C. Hooks & useState (giới thiệu nhanh)

Trong các buổi sau, bạn sẽ dùng rất nhiều **Hook** – các hàm đặc biệt bắt đầu bằng `use`:

- `useState`, `useEffect`, `useRef`, ...

**Rules of Hooks (nhớ trước):**

1. Chỉ gọi Hook ở **top-level** của component (không gọi trong `if`, `for`, function con).  
2. Chỉ gọi Hook trong **React function component** hoặc **custom Hook** (hàm bắt đầu bằng `use...`).

Hook đầu tiên và quan trọng nhất bạn cần nắm: **`useState`**.

```tsx
import { useState } from "react";

function CounterDemo() {
  const [count, setCount] = useState(0);

  const handleIncrement = () => {
    setCount(count + 1);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleIncrement}>Tăng</button>
    </div>
  );
}
```

- `count`: giá trị hiện tại của state.  
- `setCount`: hàm để cập nhật state.  
- `useState(0)`: khởi tạo state với giá trị ban đầu là `0`.

> ❗ Không gán thẳng `count = 5`.  
> Hãy luôn dùng `setCount(5)` để React biết cần render lại UI.

---

## 4. THIẾT LẬP DỰ ÁN (SETUP)

Chúng ta sử dụng **Vite** + **TypeScript** chuẩn ngay từ đầu.

**Câu lệnh tạo project:**

```bash
npm create vite@latest ten-project -- --template react-ts
cd ten-project
npm install
npm run dev
```

- `react-ts`: Template dành cho React kết hợp TypeScript.  
- `npm install`: Cài đặt thư viện.  
- `npm run dev`: Lệnh chạy server ảo để code.


---

## ✅ CHECKLIST BÀI TẬP VỀ NHÀ

**Phần bắt buộc**

0. [ ] Code lại bài test đầu giờ bằng react, dùng tailwind css cho đẹp. 
1. [ ] Tạo (hoặc hoàn thiện) project React + TypeScript với Vite (`react-ts`) và chạy được `npm run dev`.  
2. [ ] Trong `App.tsx`, hiển thị **tên của bạn** (ví dụ: `<h1>Xin chào, Nam!</h1>`).  
3. [ ] Tạo component `ProductCard` nhận props: `title` (string), `price` (number), `image` (string URL).  
4. [ ] Trong `App.tsx`, tạo một mảng 3 sản phẩm và dùng `.map()` để render danh sách `ProductCard` (import có thể dùng `./components/ProductCard`).  
5. [ ] Thêm `useState<number>` để làm nút **“Add to Cart”**: mỗi lần click thì tăng `cartCount` và hiển thị số lượng item trong giỏ hàng.  
6. [ ] **Quan trọng:** Tự vẽ lại sơ đồ `State → Render → UI → Event → State` ra giấy và ghi chú đầy đủ (không nhìn tài liệu) - cái này không cần nộp, tự giác làm nếu thật sự có trách nhiệm với bản thân.

---
