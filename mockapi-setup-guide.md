# MOCKAPI SETUP GUIDE

## 🎯 Mục tiêu

Sau tài liệu này, bạn sẽ làm trước ở nhà để vô kiểm tra dùng:

- Tạo project trên mockapi.io
- Tạo resource `/login` và `/employees`
- Seed dữ liệu mẫu để demo luồng đăng nhập và CRUD nhân viên

---

## 1. Setup trên web mockapi.io (Step by Step)

## Bước 1: Tạo tài khoản và project

1. Truy cập: https://mockapi.io
2. Bấm `Sign Up` (hoặc `Sign In` nếu đã có tài khoản).
3. Bấm `Create new project`. (nút dấu + màu xành kế bên nút chữ Projects)
4. Điền thông tin:
   - Project name: `hrm-lite-yourname`
   - API Prefix: `{yourname}`
5. Bấm `Create`.

Kết quả: bạn có Base URL dạng:

```txt
https://<project-id>.mockapi.io/yourname
```

Ghi lại Base URL này, sẽ dùng trong code.

---

## Bước 2: Tạo resource `login`

1. Trong project vừa tạo, bấm `New Resource`.
2. Resource name: `login`
3. Bấm `Create`.
4. chọn Data ở Resource login mới tạo
5. Tạo 1 record duy nhất (id = 1) với dữ liệu:

```json
[{
  "id": "1",
  "email": "test@gmail.com",
  "role": "admin",
  "name": "Minh",
  "accessToken": "fake-jwt-token-123456",
  "message": "Login successfully"
}]
```

5. Lưu lại và test endpoint:

```txt
GET /login/1
```

URL đầy đủ:

```txt
https://<project-id>.mockapi.io/yourname/login/1
```

Nếu trình duyệt trả JSON đúng như trên là đạt.

---

## Bước 3: Tạo resource `employees`

1. Bấm `New Resource`.
2. Resource name: `employees`
3. Tạo các field khuyên dùng:

   - `id` (Object ID)
   - `fullName` (string)
   - `department` (string)
   - `gender`(string)
4. Seed 3-5 nhân viên mẫu.

Ví dụ records:

```json
[
  {
    "fullName": "Delores Langosh",
    "department": "Tools",
    "gender": "Male to female transsexual woman",
    "id": "1"
  },
  {
    "fullName": "Henry Conn",
    "department": "Home",
    "gender": "Transsexual",
    "id": "2"
  },
  {
    "fullName": "Sabrina Stanton",
    "department": "Beauty",
    "gender": "Demi-boy",
    "id": "3"
  },
  {
    "fullName": "Gordon Erdman",
    "department": "Automotive",
    "gender": "Cis",
    "id": "4"
  },
  {
    "fullName": "Miss Robyn Lubowitz",
    "department": "Tools",
    "gender": "Non-binary",
    "id": "5"
  },
  {
    "fullName": "Agnes Kerluke",
    "department": "Toys",
    "gender": "Transsexual female",
    "id": "6"
  }
]
```

- Đăng nhập giả lập: luôn gọi `GET /login/1`
- Danh sách nhân viên: `GET /employees`
- Thêm nhân viên: `POST /employees`
- Xóa nhân viên: `DELETE /employees/:id`
- Cập nhật nhân viên: `PUT /employees/:id`

---

## 2. Setup code React (Feature-Based)
---

## Bước 1: Tạo biến môi trường

Tạo file `.env` ở root project:

```env
VITE_API_BASE_URL=https://<project-id>.mockapi.io/yourname
```

Lưu ý:

- Với Vite, biến phải có prefix `VITE_`
- Sau khi sửa `.env`, cần restart dev server

---
## Bước 2: Viết API login giả lập

```ts
type LoginResponse = {
  accessToken: string;
  message: string;
  user: {
    id: string;
    name: string;
    email: string;
    role: string;
  };
};

export const authApi = {
  fakeLogin: async (): Promise<LoginResponse> => {
    const { data } = await api.get("/login/1");

    return {
      accessToken: data.accessToken,
      message: data.message,
      user: {
        id: data.id,
        name: data.name,
        email: data.email,
        role: data.role,
      },
    };
  },
};
```
