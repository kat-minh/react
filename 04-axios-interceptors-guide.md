# AXIOS LAYER & INTERCEPTORS - STUDENT GUIDE

## 🎯 Mục tiêu học tập

Xây dựng một API Layer tập trung (centralized) với Axios interceptors để:

- Tự động attach JWT token vào mọi request
- Xử lý lỗi tập trung
- Tự động refresh token khi hết hạn
- Normalize response từ backend

## 🧠 Khái niệm cốt lõi

### Interceptors (Trạm kiểm soát)

**Request Interceptor:** Chặn request TRƯỚC khi bay ra khỏi App

- Dùng để nhét Token vào Header
- Modify request config

**Response Interceptor:** Chặn response TRƯỚC khi về tới Component

- Xử lý lỗi chung (401, 500)
- Convert/normalize dữ liệu

### Refresh Token Flow

1. App gọi API A → Backend trả về **401 Unauthorized**
2. Interceptor bắt được 401
3. App âm thầm gọi API **Refresh Token**
4. Nếu thành công → Lưu token mới → Tự động gọi lại API A
5. Nếu thất bại → Logout

---

## 🛠️ PHASE 1: SETUP AXIOS INSTANCE

### Bước 1: Cài đặt Axios

```bash
npm install axios
```

### Bước 2: Tạo Base Client

Tạo file `src/lib/http/apiClient.ts`:

```ts
import axios from "axios";

// Create instance
const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL, // Nhớ config .env
  headers: {
    "Content-Type": "application/json",
  },
  timeout: 10000, // 10s timeout
});

export default apiClient;
```

### Bước 3: Service Layer Pattern

⚠️ **QUAN TRỌNG: Backend Inconsistency**

Backend thường không nhất quán trong response format:

- Một số API trả `result`, một số trả `data`
- Một số dùng `message`, một số dùng `msg`
- Field naming: `snake_case` vs `camelCase`

**Frontend PHẢI normalize response trong service layer!**

#### Ví dụ Backend không nhất quán:

```ts
// ❌ VẤN ĐỀ: Backend Inconsistent Responses

// API Login trả về:
{
  "message": "Login successful",
  "result": {
    "access_token": "eyJ...",
    "refresh_token": "eyJ..."
  }
}

// API Register trả về (KHÁC format!):
{
  "msg": "Register successful", // ❌ msg thay vì message
  "data": {                      // ❌ data thay vì result
    "access_token": "eyJ...",
    "refresh_token": "eyJ..."
  }
}
```

#### ✅ Giải pháp: Normalize trong Service Layer

Tạo file `src/lib/api/users.api.ts`:

```ts
import apiClient from "@/lib/http/apiClient";

// Define interface Frontend cần (CHUẨN HÓA)
interface AuthTokens {
  accessToken: string;
  refreshToken: string;
}

interface User {
  _id: string;
  email: string;
  name: string;
}

export const usersApi = {
  // ✅ Login: Normalize result → accessToken/refreshToken
  async login(credentials: {
    email: string;
    password: string;
  }): Promise<AuthTokens> {
    const { data } = await apiClient.post("/users/login", credentials);

    // Backend trả: { message, result: { access_token, refresh_token } }
    // Frontend nhận: { accessToken, refreshToken }
    return {
      accessToken: data.result.access_token, // ⚡ Convert snake_case → camelCase
      refreshToken: data.result.refresh_token,
    };
  },

  // ✅ Register: Normalize data → accessToken/refreshToken
  async register(userData: RegisterDto): Promise<AuthTokens> {
    const { data } = await apiClient.post("/users/register", userData);

    // ⚠️ Backend dùng 'data' thay vì 'result'
    return {
      accessToken: data.data.access_token, // ⚡ data.data (không phải data.result)
      refreshToken: data.data.refresh_token,
    };
  },

  // ✅ Get Me: Normalize result.user → user
  async getMe(): Promise<User> {
    const { data } = await apiClient.post("/users/me");

    // Backend trả: { message, result: { user: {...} } }
    // Frontend nhận: User object trực tiếp
    return data.result.user;
  },

  // ✅ Refresh Token: Normalize
  async refreshToken(refreshToken: string): Promise<AuthTokens> {
    const { data } = await apiClient.post("/users/refresh-token", {
      refresh_token: refreshToken, // Backend yêu cầu snake_case
    });

    return {
      accessToken: data.result.access_token,
      refreshToken: data.result.refresh_token,
    };
  },
};
```

**Lợi ích:**

- Components chỉ cần biết interface Frontend (camelCase)
- Mọi inconsistency được xử lý ở 1 chỗ duy nhất
- Dễ maintain khi backend thay đổi

---

## 🛠️ PHASE 2: REQUEST INTERCEPTOR (Attach Token)

Cập nhật `src/lib/http/apiClient.ts`:

```ts
import axios from "axios";
import { useAuthStore } from "@/stores/auth.store";

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  headers: {
    "Content-Type": "application/json",
  },
  timeout: 10000,
});

// ⚡ Add Request Interceptor
apiClient.interceptors.request.use(
  (config) => {
    // 1. Lấy token từ Zustand (không dùng hook, lấy trực tiếp)
    const accessToken = useAuthStore.getState().accessToken;

    // 2. Nếu có token, attach vào Header
    if (accessToken) {
      config.headers.Authorization = `Bearer ${accessToken}`;
    }

    return config;
  },
  (error) => Promise.reject(error),
);

export default apiClient;
```

**Lưu ý:** Dùng `useAuthStore.getState()` vì file `.ts` không thể dùng React hooks.

---

## 🛠️ PHASE 3: RESPONSE INTERCEPTOR (Auto Refresh Token)

### Bước 1: Handle Normal Errors

Cập nhật `src/lib/http/apiClient.ts`:

```ts
// Add Response Interceptor
apiClient.interceptors.response.use(
  (response) => response.data, // ⚠️ Return data trực tiếp
  async (error) => {
    const originalRequest = error.config;
    const status = error.response?.status;

    // TODO: Handle 401 here

    return Promise.reject(error);
  },
);
```

### Bước 2: Implement Auto-Refresh Logic

```ts
apiClient.interceptors.response.use(
  (response) => response.data,
  async (error) => {
    const originalRequest = error.config;
    const status = error.response?.status;

    // Nếu lỗi 401 và chưa từng retry (tránh lặp vô tận)
    if (status === 401 && !originalRequest._retry) {
      (originalRequest as any)._retry = true; // ⚠️ Cast any để tránh lỗi TS

      try {
        const refreshToken = useAuthStore.getState().refreshToken;
        if (!refreshToken) throw new Error("No refresh token");

        // 1. Gọi API xin token mới
        // ⚠️ Dùng axios thường để tránh dính interceptor của apiClient
        const response = await axios.post(
          `${import.meta.env.VITE_API_URL}/users/refresh-token`,
          {
            refresh_token: refreshToken,
          },
        );

        const { access_token, refresh_token } = response.data.result;

        // 2. Lưu token mới vào store
        useAuthStore.getState().setTokens(access_token, refresh_token);

        // 3. Update header của request cũ
        originalRequest.headers.Authorization = `Bearer ${access_token}`;

        // 4. Gọi lại request cũ
        return apiClient(originalRequest);
      } catch (refreshError) {
        // Nếu refresh cũng fail → Logout luôn
        useAuthStore.getState().clearTokens();
        window.location.href = "/login"; // Redirect cứng
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  },
);
```

**Giải thích logic:**

1. Bắt lỗi 401 (Unauthorized)
2. Check flag `_retry` để tránh vòng lặp vô tận
3. Gọi API refresh token (dùng axios gốc, không qua interceptor)
4. Lưu token mới vào store
5. Gọi lại request cũ với token mới
6. Nếu refresh fail → Logout

---

## ⚠️ Common Mistakes

### 1. Circular Dependency

**Bug:** Import `apiClient` vào `auth.store` rồi lại import `auth.store` vào `apiClient`

**Fix:** Cấu trúc file cẩn thận. `apiClient` import store, store KHÔNG import lại client.

### 2. Infinite Loop 401

**Bug:** API Refresh Token cũng bị 401 → Interceptor lại bắt → Lại gọi refresh → Vòng lặp

**Fix:** Dùng `axios` gốc (không interceptor) để gọi API refresh token.

### 3. Quên `_retry` flag

**Bug:** Vòng lặp vô tận nếu token mới vẫn sai

**Fix:** Luôn check `!originalRequest._retry` trước khi refresh.

---

## 📝 Best Practices

1. **KHÔNG tin Backend 100%** - Luôn normalize response trong service layer
2. **KHÔNG dùng raw API response** trực tiếp trong component
3. **TypeScript interface cho Frontend** khác với Backend (camelCase vs snake_case)
4. **Document inconsistencies** trong code comments
5. **Centralize normalization** - 1 chỗ duy nhất (service layer)

### Common Backend Inconsistencies:

- Field names: `result` vs `data` vs `payload`
- Message field: `message` vs `msg` vs `error`
- Case convention: `snake_case` vs `camelCase`
- Error shape: `errors` (object) vs `error` (string)
- Date format: ISO string vs timestamp vs "DD/MM/YYYY"

---

## 🎯 Mini Task

**Test Interceptor:**

1. Vào `LoginPage`, sửa nút Login để gọi `usersApi.getMe()`
2. Set token giả trong `localStorage` thành chuỗi `abc` (token sai)
3. Bấm nút → Quan sát Network Tab
4. **Kỳ vọng:**
   - Request `/me` fail 401
   - Ngay lập tức thấy request `/refresh` chạy
   - Rồi lại thấy `/me` chạy lại

---

## 🏠 Homework

**Xây dựng trang đăng ký tài khoản:**

### Yêu cầu:

1. **Call API Register:**

   - Endpoint: `POST /authentication/register`
   - API Docs: https://thich-cung-kieng-server.onrender.com/docs#/Authentication/AuthController_register
   - Request body:
     ```json
     {
       "username": "string",
       "password": "string",
       "email": "string"
     }
     ```
   - Response thành công (201):
     ```json
     log ra xem
     ```

2. **Tạo Service Layer:**

   - Tạo file `src/lib/api/auth.api.ts`
   - Tạo interface `RegisterDto` với các field: username, email, password (cái mình sẽ gửi đi)
   - Tạo interface `RegisterResponse` để normalize dữ liệu từ backend (cái mình muốn nhận)
   - Implement function `register()` trong service layer
   - Nhớ normalize response

3. **Tạo trang Register:**

   - Tạo file `src/pages/RegisterPage.tsx`
   - Form gồm 3 field: Username, Email, Password
   - Khi submit:
     - Xử lí loading
     - Redirect về trang login khi thành công
     - Nếu lỗi -> Hiển thị "Đăng ký thất bại, vui lòng thử lại"

4. **Cập nhật Router:**

   - Thêm route `/register` vào `router.tsx`
   - Thêm link "Chưa có tài khoản? Đăng ký ngay" ở trang Login


### Bonus (tùy chọn):

- Thêm confirm password field (phải match với password)
- Hiển thị password strength indicator
- Disable nút submit khi đang loading
