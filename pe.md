## THỰC HÀNH (6 Điểm)

### 💻 BÀI TOÁN: XÂY DỰNG "DASHBOARD QUẢN LÝ NHÂN SỰ" (HRM LITE)

**Yêu cầu kỹ thuật bắt buộc:**
- Khởi tạo app bằng Vite + React + TypeScript.
- Setup các library: React Router, Zustand, React Query, React Hook Form, Zod, Axios, UI Library tự chọn (TailwindCSS/Shadcn/Bootstrap).
- Cấu trúc thư mục mô hình Feature-Based (`src/features/...`, `src/shared/...`).
- Toàn bộ dữ liệu Fetch/Mutate bắt buộc tương tác với MockAPI giả (Mỗi học viên tạo 1 link MockAPI (`mockapi.io`) cho riêng mình).

#### 1. Yêu cầu tính năng (Feature Spec):

🔹 **Tính năng 1: Xác thực hệ thống (Authentication Flow)**
- **Giao diện:** Xây dựng màn hình `/login`. Form yêu cầu `email` (chuẩn email) và `password` (tối thiểu 6 kí tự). Phải có thông báo lỗi nếu nhập sai. Dùng RHF + Zod.
- **Logic (Hướng dẫn API Fake Login):** Vì `mockapi.io` thuần tuý chỉ là RESTful API cho CRUD (Create, Read, Update, Delete) và không hỗ trợ cơ chế Auth (cấp JWT, verify password phức tạp). Các bạn xử lý bằng "Trick" như sau:
  - Tạo 1 Resource trên MockAPI tên là `/login` với **Data cấp sẵn** (1 record có id=1 chứa chuỗi `"accessToken": "fake-jwt-token-123456"`, role, message success...).
  - Front-end điền Form Login xong, không cần gửi thông tin lên bằng `POST` mà chỉ cần dùng Axios tạo 1 request `GET` thông thường xuống đúng đường dẫn `/login/1` đó để lấy JSON token về.
  - Sau đó lưu giá trị token kia vào **Zustand store (với persist middleware)**.
  - Lúc này, ứng dụng React coi như đã có Token và cho phép đăng nhập thành công. Code sẽ chạy tương tự đồ án thật trên lớp.
- **Bảo vệ:** Lập trình component `<ProtectedRoute>` để ngăn những ai chưa login không vào được các trang nội bộ của Dashboard.

🔹 **Tính năng 2: Giao diện Layout Toàn hệ thống**
- Áp dụng cấu trúc `<Outlet/>`. Hiển thị Sidebar tĩnh với 2 tab: DashBoard & Quản lý nhân viên. Navbar ở trên cùng hiện tên tài khoản giả định và nút Đăng xuất (Gọi action clear auth của Zustand và đẩy về login).

🔹 **Tính năng 3: Quản lý danh sách Nhân viên (Employees Feature)**
- Giao diện dạng Bảng (Bảng có UI chuyên nghiệp, đổ bóng, viền hiện đại).
- **Fetch Data:** Dùng `useQuery` để call api `/employees` lấy danh sách. Xử lý các trạng thái `isLoading` (hiện spinner/skeleton) và `isError`.
- **Thêm nhân viên:** Nút "Create New" mở Modal Form hoặc trang `/employees/create` sử dụng RHF + Zod (Họ tên, tuổi, vị trí phòng ban). Call `useMutation` để submit. Thành công hiện Toast Thông báo.
- **Auto Refetch:** Sau khi tạo mới thành công, bảng danh sách phải tự sinh mục mới mà không cần thao tác F5 trên trình duyệt (sử dụng `invalidateQueries`).
- **Xóa nhân viên:** Nút xoá trên mỗi dòng gọi mutation xóa item trên mockapi, sau đó invalidate list.

#### 2. Tiêu chí chấm điểm (Rubric):

| Tiêu chí | Trọng số | Ràng buộc |
| :------- | :------: | :------- |
| **Cấu trúc & Setup chuẩn** | 1.0 đ | Đúng Feature Based, tách Types, Services, components rành mạch. |
| **Authentication Flow** | 1.5 đ | Zustand persist hoạt động. Protected route cấu hình đúng URL redirect, Logout đúng logic. |
| **Fetching Data (Query)** | 1.0 đ | Render loading UI, handle báo lỗi UI tốt, không memory leak. |
| **Form Management (Mutation + Zod)**| 1.5 đ | Validation đầy đủ (ko rỗng, độ dài hợp lệ), hiển thị báo đỏ form đúng. Trigger Mutation, handle Invalidating. |
| **UI/UX Aesthetics** | 0.5 đ | Phối màu hiện đại, button có hiệu ứng, toast message (ví dụ dùng Sonner) mượt mà. Đề bài không gò bó css. |
| **Deploy & Bonus (Tùy chọn)** | 0.5 đ | Setup Github repository rõ ràng. Có chạy thành công Github Action Linting, Build deploy hoàn chỉnh lên Vercel ra tên miền thực. |

*Lưu ý: Bất kì dấu hiệu vi phạm gọi API trực tiếp trong useEffect gây Loop vô hạn, hoặc thiết lập form không kiểm soát (quên defaultValues) tùy độ nghiêm trọng trừ 0.5 tới 1 điểm.*
