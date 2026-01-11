# 📘 SUMMARY: STATE, PROPS & RENDERING (SESSION 02)

Chào mừng bạn đến với Session 02. Hôm nay chúng ta đi sâu vào cơ chế hoạt động thực sự của React: **Rendering Model**.

---

## 1. MENTAL MODEL: PROPS VS STATE 🧠

Để làm chủ React, bạn cần phân biệt rạch ròi hai khái niệm này.

| Đặc điểm | Props (Properties) | State (Trạng thái) |
| :--- | :--- | :--- |
| **Vai trò** | **Dữ liệu từ bên ngoài** truyền vào. | **Dữ liệu nội bộ** của component. |
| **Quyền hạn** | **Read-only (Chỉ đọc)**. Component con không được sửa props. | **Read-write**. Component tự do thay đổi (qua `setState`). |
| **Ẩn dụ** | 🎁 **Món quà từ bố mẹ**. Bố mẹ cho sao nhận vậy, cấm đòi hỏi. | 👛 **Ví tiền riêng**. Mình tự kiếm, tự tiêu, tự quản lý. |

> **💡 Nguyên tắc:** Dữ liệu nên được quản lý bởi component nào **cần nó nhất** hoặc **component cha chung** của các component cần nó.

---

## 2. RENDERING MODEL: KHI NÀO REACT VẼ LẠI? 🎨

Một câu hỏi phỏng vấn kinh điển: *"Khi nào thì component re-render?"*

### Cơ chế mặc định
1.  Khi **State** của nó thay đổi (`setState`).
2.  Khi **Props** truyền vào thay đổi.
3.  Và quan trọng nhất: **Khi Component CHA render, toàn bộ Component CON sẽ render theo.** (Dù con không nhận props gì).

> **⚠️ Lưu ý:** Đây là tính năng "an toàn" của React để đảm bảo UI luôn đồng bộ. Đừng lo lắng về hiệu năng quá sớm.

---

## 3. COMPOSITION PATTERN: `CHILDREN` PROP 🧩

Làm sao để tạo một component linh hoạt, ví dụ cái khung `Card` có thể chứa bất cứ thứ gì (Text, Button, Image)?
Đừng dùng quá nhiều props (`text`, `showImage`, `showButton`...). Hãy dùng **Composition**.

**Code mẫu `Card.tsx`:**

```tsx
interface CardProps {
  children: React.ReactNode; // 👈 Chứa mọi thứ nằm giữa cặp thẻ đóng mở
}

function Card({ children }: CardProps) {
  return <div className="border p-4 shadow">{children}</div>;
}
```

**Sử dụng:**

```tsx
<Card>
  <h1>Tiêu đề</h1>
  <button>Click</button> {/* Children là tất cả đống này */}
</Card>
```

---

## 4. LISTS & KEYS 🔑

Khi dùng `.map()` để render danh sách, React bắt buộc phải có `key`.

### Tại sao cần Key?
Key giống như **CCCD (Căn cước công dân)**. Giúp React phân biệt được item nào đã thêm, sửa, hay xoá để cập nhật DOM hiệu quả nhất.

### Quy tắc vàng
1.  ✅ **Luôn dùng ID từ dữ liệu:** `key={user.id}`.
2.  ⛔ **Hạn chế dùng Index:** `key={index}` (Chỉ dùng khi list *tuyệt đối* không bao giờ thêm/sửa/xoá/sắp xếp).

```tsx
// ✅ Đúng
{products.map(p => (
  <ProductCard key={p.id} data={p} />
))}
```

---

## 5. DATA FLOW: "REMOTE CONTROL" 🎮

Làm sao component CON (nút "Add to Cart") thay đổi được STATE nằm ở CHA (số lượng giỏ hàng)?

**Giải pháp: Inverse Data Flow (Dòng dữ liệu ngược)**
1.  **Cha:** Giữ State (`cartCount`) + Tạo hàm thay đổi (`handleAddToCart`).
2.  **Cha:** Truyền hàm đó xuống cho Con qua Props (như đưa cái Remote).
3.  **Con:** Khi cần, bấm nút trên Remote (`onClick={props.onAddToCart}`).

```tsx
// Parent
function App() {
  const [count, setCount] = useState(0);
  return <Child onAdd={() => setCount(count + 1)} />; // 👈 Đưa Remote
}

// Child
function Child({ onAdd }) {
  return <button onClick={onAdd}>Tăng</button>; // 👈 Bấm Remote
}
```

---

## 6. FORMS & CONTROLLED INPUT 📝

Trong React, State là "Single Source of Truth". Input không được tự ý lưu giá trị, mà phải nghe theo State.

```tsx
const [val, setVal] = useState("");

<input 
  value={val}                // 1. Hiển thị theo State
  onChange={e => setVal(e.target.value)} // 2. Cập nhật State khi gõ
/>
```

---

## 7. IMMUTABILITY & C.U.D 🛡️

Không bao giờ thay đổi trực tiếp State (Mutation). Luôn tạo ra bản copy mới.

| Hành động | KHÔNG LÀM (Mutation) ❌ | NÊN LÀM (Immutability) ✅ |
| :--- | :--- | :--- |
| **Thêm (Create)** | `arr.push(newItem)` | `[...arr, newItem]` (Spread Syntax) |
| **Xoá (Delete)** | `arr.splice(index, 1)` | `arr.filter(item => item.id !== id)` |
| **Sửa (Update)** | `item.name = "Mới"` | `arr.map(item => item.id === id ? { ...item, name: "Mới" } : item)` |

---

## ✅ CHECKLIST BÀI TẬP VỀ NHÀ

1.  [ ] **Refactor Component:** Sử dụng `children` prop để tái sử dụng `UserCard` hoặc `ProductCard` (như ví dụ `Card` trong bài học).
2.  [ ] **Hoàn thiện tính năng Update:** Trong bài học chúng ta đã làm ProductManager nhưng có thể chưa kịp làm phần **Sửa (Update)**. Hãy tự mình hoàn thiện logic `handleEdit` để cập nhật tên sản phẩm (gợi ý: dùng `window.prompt` hoặc thêm ô input sửa).

3.  [ ] **Mini Project: Todo List App (CRUD + Filter)** 🚀
    Đây là bài tập trọng tâm để rèn luyện tư duy Data Flow và State.

    **Yêu cầu chức năng:**
    *   **Hiển thị (Read):** Danh sách công việc. Mỗi việc có tên và trạng thái (Hoàn thành / Chưa hoàn thành).
    *   **Thêm (Create):** Ô input để thêm việc mới.
    *   **Sửa (Update):**
        *   Cho phép đánh dấu **Done / Not Yet** (Toggle status - ví dụ: click vào item hoặc nút check).
    *   **Xoá (Delete):** Có nút xoá từng công việc.
    *   **Tìm kiếm (Search):** Tìm theo tên công việc (Real-time filter).
    *   **Bộ lọc (Filter):** Có 3 chế độ xem (dùng nút bấm hoặc select):
        *   *All* (Tất cả)
        *   *Done* (Đã xong)
        *   *Not Yet* (Chưa xong)

    **Gợi ý kỹ thuật:**
    *   Dùng `useState` cho danh sách todos.
    *   Dùng `useState` cho `filterStatus` ('all', 'done', 'notYet').
    *   Dùng **Derived State** để tính toán danh sách hiển thị (kết hợp logic search + logic filter), KHÔNG tạo state thừa `filteredTodos` (nhớ bài học về Component Search nhé).

---
