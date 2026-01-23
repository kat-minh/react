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
