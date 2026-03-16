# BÀI 09: PRODUCTION HARDENING & FE THEORY BOOST

## 🎯 Mục tiêu học tập
Nâng cấp chất lượng dự án từ "chạy được" sang "sẵn sàng ship" (Ship-ready) và nắm vững bộ khung lý thuyết Frontend chuyên sâu để giải quyết các bài toán kỹ thuật phức tạp.

---

## 🕒 I. DONE CODE VS. SHIP-READY CODE
| DONE CODE | SHIP-READY CODE |
| :--- | :--- |
| Chạy được ở local. | Chạy ổn định trên Internet (Production). |
| Feature có đủ nhưng chưa tinh tế. | Có đầy đủ các trạng thái Loading / Error / Empty. |
| Không báo đỏ console. | Code sạch (`lint pass`), gỡ bỏ toàn bộ debug log. |
| Auth/Session chạy kiểu "may mắn". | Auth vững chắc, có cơ chế Silent Refresh & Logout an toàn. |

---

## 🛠️ II. PRODUCTION HARDENING (LÀM VỮNG HỆ THỐNG)

### 1. Cleanup Checklist (Làm sạch chuyên nghiệp)
- [ ] **Lint & Build**: Chạy `pnpm run lint` và `pnpm run build`.
- [ ] **Gỡ bỏ debug**: Xóa sạch toàn bộ `console.log` và các imports dư thừa.
- [ ] **Thông báo lỗi**: Thay thế các câu "Something went wrong" bằng thông báo có nghĩa.
- [ ] **Tài liệu**: Cập nhật `.env.example` và viết `README.md` (Features, Setup, Env, Structure).

### 2. Auth & Session Hardening
- **State Machine**: Hiểu luồng `UNAUTHENTICATED -> AUTHENTICATED -> REFRESHING`.
- **Axios Interceptor**: Đảm bảo refresh token hoạt động tốt khi user reload hoặc token hết hạn.
- **Logout behavior**: Phải làm sạch cả LocalStorage (Token) lẫn Query Cache.

---

## ⚡ III. PERFORMANCE & UX SPEED WINS

### 1. Route-based Lazy Loading
Không tải toàn bộ app một lúc. Tải trang nào, code trang đó mới được fetch về.
```tsx
const ManageRitualList = lazy(() => import("@/features/rituals/pages/ManageRitualList"));

// Bọc trong Suspense
<Suspense fallback={<Skeleton className="h-20 w-full" />}>
  <ManageRitualList />
</Suspense>
```

### 2. UX Speed (Cảm giác tốc độ)
- **Submit Button**: Disable nút ngay khi click và hiện loading state.
- **Skeletons**: Hiển thị ngay lập tức khi fetch data.
- **Image optimization**: Không kéo ảnh gốc dung lượng lớn cho thumbnail.

---

## 🧭 IV. SIÊU CẤP API DEBUGGING CHEATSHEET
Tư duy chuẩn là debug theo pipeline: **FE Request -> Network -> Server -> Response -> FE Update**.

| Kịch bản | Dấu hiệu (Network Tab) | Nguyên nhân & Cách khắc phục |
| :--- | :--- | :--- |
| **1. Không thấy request** | Trống trơn trong Network | **Lỗi FE**: Quên gọi hàm (quên cặp ngoặc `()`) hoặc API logic chưa được kích hoạt. |
| **2. 404 Not Found** | Status `404` | **Lỗi Endpoint**: Sai URL, sai baseURL hoặc BE chưa map route đó vào hệ thống. |
| **3. 500 Server Error** | Status `500` | **Lỗi BE**: 99% lỗi backend. Đọc Response Body để biết BE đang gặp sự cố gì bên trong. |
| **4. 401 Unauthorized** | Status `401` | **Lỗi Auth**: Token không được gửi hoặc Token đã chết hẳn (check Authorization Header). |
| **5. 400 Bad Request** | Status `400` | **Lỗi Payload**: FE gửi sai tên field (VD: BE cần `email`, FE gửi `mail`) hoặc sai format. |
| **6. Mismatch Shape** | Status `200` | **Lỗi Mapping**: BE trả về `{data: []}` nhưng FE lại truy cập `res.items` -> UI hỏng. |
| **7. API OK - UI lặng im** | Status `200` | **Lỗi State**: Request thành công nhưng FE quên gọi hàm `setState` hoặc sai Dependency Array. |
| **8. Sai kiểu dữ liệu** | Status `400` | **Lỗi Format**: BE cần Number (`100`) nhưng FE gửi String (`"100"`) khiến BE từ chối. |
| **9. Query Param sai** | Data trả về sai/thiếu | **Lỗi Param**: BE cần `?pageNumber=1` nhưng FE gửi `?page=1`. |
| **10. Postman OK - FE FAIL** | Lỗi đỏ rực/Status 0 | **Lỗi CORS / Headers**: Postman thường tự thêm header, FE thì không.BE chưa whitelist domain của FE. |
| **11. Timeout/Network** | Pending mãi mãi | **Lỗi Mạng**: Server chết, mạng chập chờn hoặc timeout quá ngắn. |

---

## 🏗️ V. FRONTEND PATTERNS (12 MẪU THIẾT KẾ)

1.  **Container/Presentational**: Tách logic fetch data khỏi component hiển thị.
2.  **Custom Hook**: Gom logic dùng chung như `useAuth`, `usePagination`, `useDebounce`.
3.  **Feature-based Folders**: Quản lý folder theo domain nghiệp vụ (`features/auth`).
4.  **Guarded Route**: Kiểm soát quyền truy cập trang tập trung (Guest/User/Admin).
5.  **Error Boundary**: Bọc các vùng để tránh app bị trắng toàn trang khi crash nhỏ.
6.  **Service/Repository**: Tách lớp gọi API ra khỏi Component để dễ quản lý endpoint.
7.  **UI State Pattern**: Chuẩn hóa bộ 4 trạng thái: `loading / error / empty / success`.
8.  **Optimistic Update**: Cập nhật UI trước (như đã thành công), sau đó rollback nếu fail.
9.  **Retry + Backoff**: Tự động gọi lại API khi gặp lỗi mạng tạm thời.
10. **Pagination/Infinite Query**: Tải danh sách lớn từng phần (Lazy list).
11. **Debounce/Throttle**: Tối ưu hóa các hành động dày đặc như Search Input.
12. **Feature Flag**: Cho phép bật/tắt tính năng từ xa (A/B testing hoặc Safe rollback).

---

## 📚 VI. KIẾN THỨC CỐT LÕI (FE RECAP)

### 1. Rendering Strategies
- **CSR**: Render tại browser (Dashboard).
- **SSR**: Server trả về HTML hoàn chỉnh (Landing Page/SEO).
- **SSG**: Tạo file tĩnh lúc build (Blog/Documentation).
- **ISR**: Tĩnh nhưng tự cập nhật theo thời gian (E-commerce).

### 2. CDN & Caching
- **CDN**: Lưu cache gần user địa lý nhất để giảm trễ (Latency).
- **Caching**: Cần phân biệt cache ở Browser, CDN và Server. Cache giúp nhanh nhưng có nguy cơ data cũ (Stale).

### 3. Core Web Vitals
- **LCP**: Tốc độ hiển thị nội dung lớn nhất.
- **FID / INP**: Độ trễ khi người dùng tương tác với nút.
- **CLS**: Mức độ ổn định giao diện (tránh nhảy layout).

---

## ✅ VII. RELEASE READY CHECKLIST
- [ ] Build & Lint local pass 100%.
- [ ] Không còn debug logs hay console rác.
- [ ] `.env.example` chuẩn xác.
- [ ] README đã liệt kê đủ các bước Setup.
- [ ] SPA routes hoạt động (reload link sâu không 404).
