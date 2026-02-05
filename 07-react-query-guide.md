# REACT QUERY (TANSTACK QUERY) - STUDENT GUIDE

## 🎯 Mục tiêu học tập

Sử dụng TanStack Query (React Query) để quản lý Server State hiệu quả:

- Hiểu sự khác biệt giữa Client State và Server State
- Fetch data với `useQuery` thay vì `useEffect`
- Tận dụng caching và stale-while-revalidate
- Xử lý loading, error states một cách chuẩn mực
- Sử dụng React Query DevTools để debug

---

## 🧠 Khái niệm cốt lõi

### Client State vs Server State

#### 🟢 Client State (Local State - Ta kiểm soát 100%)

```tsx
// Client State - Dữ liệu của App, do ta tạo ra
const [isModalOpen, setIsModalOpen] = useState(false);
const [theme, setTheme] = useState("dark");
const [sidebarCollapsed, setSidebarCollapsed] = useState(false);

// ✅ Đặc điểm:
// - Ta tạo ra, ta kiểm soát
// - Không có delay, không có lỗi network
// - Luôn "tươi mới" 100%
// - Lưu trong useState, Zustand, Redux
```

#### 🔴 Server State (Remote State - Ta chỉ "mượn" từ Server)

```tsx
// Server State - Dữ liệu từ Server, ta chỉ cache nó
const user = { id: 1, name: "John" }; // Lấy từ API /users/me

// ⚠️ Đặc điểm:
// - Ta KHÔNG sở hữu (Server mới là chủ)
// - Có thể CŨ bất kỳ lúc nào (ai đó đổi trên Server)
// - Có thể LỖI (network fail, server crash)
// - Có DELAY (phải chờ network)
// - Cần CACHE để app nhanh
// - Cần SYNC để data không lỗi thời
```

**Bảng So Sánh:**

| Aspect        | Client State      | Server State                    |
| ------------- | ----------------- | ------------------------------- |
| **Nguồn gốc** | App tạo ra        | Server cung cấp                 |
| **Ownership** | App sở hữu 100%   | Chỉ "mượn" tạm thời             |
| **Freshness** | Luôn mới          | Có thể cũ bất kỳ lúc nào        |
| **Network**   | Không liên quan   | Phụ thuộc hoàn toàn             |
| **Error**     | Không có          | Có thể fail (401, 500, timeout) |
| **Công cụ**   | useState, Zustand | **React Query**                 |

> **💡 Rule of Thumb:**  
> Nếu data **TỪ SERVER** → dùng React Query.  
> Nếu data **TỪ USER** (click, type, toggle) → dùng useState/Zustand.

---

### 🔄 Stale-While-Revalidate Strategy

**Ví dụ thực tế:**  
Tưởng tượng bạn vào Shopee xem giá iPhone.

**Cách cũ (useEffect):**  
Màn hình trắng xóa 2 giây → Loading spinner → Hiện giá.

**Cách React Query:**  
Hiện ngay giá cũ (cache) → Ngầm fetch giá mới → Cập nhật lặng lẽ nếu có thay đổi.

**Flow:**

```
User vào trang
    ↓
[1] Query check cache
    ↓
[2] Có cache? → Render NGAY (stale data)
    ↓
[3] Fetch mới ở background (revalidate)
    ↓
[4] So sánh data mới vs data cũ
    ↓
[5] Khác? → Re-render với data mới
    ↓
[6] Giống? → Không làm gì (tiết kiệm render)
```

**Demo Code:**

```tsx
// Lần 1: User vào trang Profile
useUser(); // → Fetch API → Cache vào ['me'] → Render

// User chuyển sang trang khác rồi quay lại Profile
useUser();
// → [NGAY LẬP TỨC] Render data từ cache (stale)
// → [BACKGROUND] Fetch API kiểm tra có data mới không
// → [NẾU CÓ MỚI] Re-render với data mới
// → [NẾU KHÔNG] Không làm gì cả
```

---

### ⏱️ StaleTime vs GcTime (Cache Time)

#### StaleTime (Thời gian "tươi")

```tsx
useQuery({
  queryKey: ["products"],
  queryFn: fetchProducts,
  staleTime: 5 * 60 * 1000, // 5 phút
});

// Nghĩa là: Data được coi là "TƯƠI" trong 5 phút
// Trong 5 phút đó:
// - Không fetch lại (dù refocus window, remount component)
// - Data được tin tưởng tuyệt đối
// - Cache hit 100%

// Sau 5 phút:
// - Data thành "STALE" (cũ)
// - Lần tới mount/refocus → Fetch lại để kiểm tra
```

**Default:** `staleTime: 0` (data ngay lập tức cũ)

#### GcTime (Garbage Collection Time - Thời gian "tồn tại")

```tsx
useQuery({
  queryKey: ["products"],
  queryFn: fetchProducts,
  gcTime: 10 * 60 * 1000, // 10 phút
});

// Nghĩa là: Cache TỒN TẠI trong bộ nhớ 10 phút
// Nếu không component nào dùng trong 10 phút → XÓA khỏi RAM

// Ví dụ:
// - User vào trang Products → Cache tạo ra
// - User rời khỏi trang → Cache vẫn còn (inactive)
// - 10 phút sau không ai dùng → Query xóa cache để tiết kiệm RAM
```

**Default:** `gcTime: 5 * 60 * 1000` (5 phút)

**Timeline Scenario:**

```tsx
staleTime: 1 phút    // Data "tươi" trong 1 phút
gcTime: 5 phút       // Cache tồn tại 5 phút

// Timeline:
[0s] User vào trang → Fetch → Cache ['products']
[30s] User F5 → KHÔNG fetch (còn tươi)
[70s] User F5 → FETCH (đã cũ > 1 phút)
[5 phút] User không dùng → Cache bị xóa → Lần sau phải fetch lại từ đầu
```

> **❗ Rule of Thumb:**
>
> - `staleTime` ngắn (0-30s): Data thay đổi liên tục (chứng khoán, chat)
> - `staleTime` dài (5-10 phút): Data ổn định (profile, settings)
> - `gcTime` luôn > `staleTime` để tận dụng cache
> - Production: `staleTime: 30s, gcTime: 5 phút` là balance tốt

---

## 🛠️ PHASE 1: SETUP TANSTACK QUERY

### Bước 1: Cài đặt Library

```bash
# Với pnpm
pnpm add @tanstack/react-query @tanstack/react-query-devtools

# Với npm
npm install @tanstack/react-query @tanstack/react-query-devtools
```

> **Lưu ý:** 2 packages riêng biệt:
>
> - `@tanstack/react-query`: Core library (production)
> - `@tanstack/react-query-devtools`: Debug tool (dev only)

---

### Bước 2: Configure Global Provider

Cập nhật `src/main.tsx`:

```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import App from "./App";
import "./index.css";

// ✅ BƯỚC 1: Tạo QueryClient instance
// QueryClient = "Người quản lý" tất cả queries trong app
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // 🎛️ Config mặc định cho TẤT CẢ queries

      // 1. Refetch on Window Focus
      refetchOnWindowFocus: false,
      // ⚠️ Mặc định: true (tự fetch lại khi user quay lại tab)
      // 💡 Học: Tắt để dễ debug (log đỡ nhảy loạn)
      // 💡 Production: Bật lại để data luôn tươi

      // 2. Retry Failed Requests
      retry: 1,
      // ⚠️ Mặc định: 3 lần
      // 💡 Học: Giảm xuống 1 để nhanh thấy lỗi
      // 💡 Production: 2-3 là hợp lý (network chập chờn)

      // 3. Stale Time
      staleTime: 0,
      // ⚠️ Mặc định: 0 (data ngay lập tức "cũ")
      // 💡 Production: 30s - 5 phút tùy data

      // 4. Cache Time (GC Time)
      gcTime: 5 * 60 * 1000,
      // ⚠️ Mặc định: 5 phút
      // 💡 Cache tồn tại 5 phút kể từ khi không còn component nào dùng
    },
  },
});

// ✅ BƯỚC 2: Wrap App với Provider
ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    {/* 
      QueryClientProvider:
      - Giống Context.Provider
      - Cung cấp queryClient cho toàn bộ component tree
      - BẮT BUỘC phải wrap ngoài cùng
    */}
    <QueryClientProvider client={queryClient}>
      <App />

      {/* 
        ReactQueryDevtools:
        - Debug tool ONLY cho development
        - Production: Tự động bị tree-shaking (không vào bundle)
        - Hiện icon hoa đỏ góc dưới màn hình
      */}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  </React.StrictMode>,
);
```

> **🚦 CHECKPOINT:**  
> Run `npm run dev`. Mở trình duyệt.  
> Góc dưới cùng bên trái thấy icon hoa đỏ chưa? 🌸  
> Nếu chưa thấy → Check lại import và Provider.

---

### Bước 3: Hiểu React Query DevTools

**Mở DevTools:** Click vào icon hoa đỏ 🌸

**Query States (màu sắc quan trọng):**

- 🟢 **Green (fresh):** Data còn "tươi" (trong `staleTime`)
- 🟡 **Yellow (stale):** Data đã "cũ" (ngoài `staleTime`) nhưng còn trong cache
- 🔵 **Blue (fetching):** Đang fetch data
- 🔴 **Red (error):** Fetch bị lỗi
- ⚪ **Gray (inactive):** Không có component nào đang dùng query này

**Thông tin hiển thị:**

```tsx
{
  queryKey: ['me'],           // Key để identify
  queryHash: '"me"',          // Hash của key
  state: {
    data: { ... },            // Data hiện tại
    error: null,              // Error nếu có
    status: 'success',        // loading | error | success
    fetchStatus: 'idle',      // fetching | paused | idle
  },
  observers: 1,               // Số component đang subscribe
  updatedAt: 1703123456789,   // Timestamp
}
```

---

## 🛠️ PHASE 2: TẠO CUSTOM HOOK

### Bước 1: Create useUser Hook

Tạo file `src/hooks/useUser.ts`:

> **Lý do dùng Custom Hook:**
>
> 1. Tái sử dụng logic (DRY principle)
> 2. Dễ test (mock hook, không mock axios)
> 3. Dễ maintain (đổi API chỉ sửa 1 chỗ)
> 4. Semantic naming (đọc code dễ hiểu)

```tsx
import { useQuery } from "@tanstack/react-query";
import { authApi } from "@/lib/api/auth.api";

/**
 * Custom Hook: Fetch current user profile
 *
 * @returns {UseQueryResult} Query result object
 *
 * Usage:
 * const { data, isLoading, error } = useUser();
 */
export const useUser = () => {
  return useQuery({
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // 📌 QUERY KEY (BẮT BUỘC)
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    queryKey: ["me"],

    /**
     * 🧠 Query Key Concept:
     *
     * - Key = ID của Cache
     * - Cùng Key = Chung Cache
     * - Khác Key = Khác Cache
     *
     * Ví dụ:
     * ['me'] ≠ ['user'] ≠ ['user', 1] ≠ ['user', 2]
     *
     * Key là Array vì:
     * - Dễ nest: ['user', userId, 'posts', postId]
     * - Dễ invalidate theo pattern
     * - React Query so sánh array theo giá trị (deep equal)
     */

    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // 🔧 QUERY FUNCTION (BẮT BUỘC)
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    queryFn: async () => {
      /**
       * queryFn PHẢI:
       * 1. Return Promise
       * 2. Throw error nếu fail (đừng catch!)
       * 3. Return data cần cache (không phải raw response)
       */

      const user = await authApi.getMe();
      // authApi.getMe() đã normalize response, trả về User object trực tiếp
      // response shape: { id, email, fullName, ... }

      // ✅ ĐÚNG: authApi đã return đúng data cần thiết
      return user;

      // ❌ SAI: Return cả response (cache thừa thãi)
      // return response;

      /**
       * 🧠 Cache Shape:
       * Query CHỈ cache cái mình return
       * Key ['me'] → Cache: { id: '123', email: 'user@example.com', fullName: 'John Doe' }
       *
       * Nếu return response → Cache: { data: { data: { result: ... } } }
       * → Rối khi dùng: data.data.data.result (WTF?)
       */
    },

    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // ⚙️ QUERY OPTIONS (TÙY CHỌN)
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

    // staleTime: 1000 * 60 * 5, // 5 phút
    /**
     * 💡 StaleTime Usage:
     *
     * 0 (default): Data ngay lập tức "cũ"
     * → Mỗi lần mount component = fetch lại
     * → Tốt cho: Real-time data (chat, stock price)
     *
     * 30s - 5 phút: Data "tươi" trong khoảng này
     * → Mount component = không fetch (dùng cache)
     * → Tốt cho: User profile, settings
     *
     * Infinity: Data KHÔNG BAO GIỜ cũ
     * → Chỉ fetch 1 lần duy nhất
     * → Tốt cho: Static data (country list, categories)
     */

    // enabled: !!accessToken,
    /**
     * 💡 Enabled Option:
     *
     * Conditional fetching:
     * enabled: false → Query không chạy
     * enabled: true → Query chạy
     *
     * Use case:
     * - Chỉ fetch khi có token
     * - Chỉ fetch khi user click button
     * - Dependent queries (fetch B sau khi có data từ A)
     */
  });
};
```

---

## 🛠️ PHASE 3: SỬ DỤNG TRONG COMPONENT

### Cập nhật ProfilePage.tsx

```tsx
import { useUser } from "@/hooks/useUser";
import { Button } from "@/components/ui/button";
import { LoadingState, ErrorState } from "@/components/ui/StatusStates";

export default function ProfilePage() {
  // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  // 🎣 CALL THE HOOK
  // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  const {
    data: user, // ✅ Rename 'data' thành 'user' cho semantic
    isLoading, // true = Lần đầu fetch, chưa có data
    isError, // true = Fetch bị lỗi
    error, // Error object nếu có
    refetch, // Function để fetch lại manually
    isFetching, // true = Đang fetch (dù có data hay không)
  } = useUser();

  /**
   * 🧠 States Breakdown:
   *
   * [FIRST LOAD]
   * isLoading: true   → Show skeleton
   * isFetching: true
   * data: undefined
   *
   * [SUCCESS]
   * isLoading: false
   * isFetching: false
   * data: { user object }
   *
   * [REFETCH (có data cũ)]
   * isLoading: false  → Không show skeleton (có data rồi!)
   * isFetching: true  → Show spinner nhỏ ở góc
   * data: { old data } → UI vẫn hiện data cũ
   *
   * [ERROR]
   * isLoading: false
   * isError: true
   * error: { message, status }
   */

  // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  // 🎨 RENDER STATES
  // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  // STATE 1: LOADING (Lần đầu, chưa có data)
  if (isLoading) {
    return <LoadingState />;
  }

  // STATE 2: ERROR
  if (isError) {
    return (
      <ErrorState
        message={error?.message || "Failed to load profile"}
        onRetry={() => refetch()}
      />
    );
  }

  // STATE 3: EMPTY (API success nhưng không có data - edge case)
  if (!user) {
    return (
      <div className="text-center p-10">
        <p className="text-gray-500">No user data available</p>
      </div>
    );
  }

  // STATE 4: SUCCESS (Có data)
  return (
    <div className="max-w-4xl mx-auto p-6">
      {/* Header với Refresh Button */}
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold">My Profile</h1>

        <Button
          onClick={() => refetch()}
          disabled={isFetching}
          className="relative"
        >
          {/* Show spinner khi đang refetch (có data cũ) */}
          {isFetching && (
            <span className="absolute inset-0 flex items-center justify-center">
              <span className="animate-spin">🔄</span>
            </span>
          )}
          <span className={isFetching ? "opacity-0" : ""}>Refresh</span>
        </Button>
      </div>

      {/* Profile Content */}
      <div className="bg-white shadow rounded-lg p-6">
        <div className="space-y-4">
          <div>
            <label className="text-sm text-gray-500">Name</label>
            <p className="text-lg font-medium">{user.fullName}</p>
          </div>

          <div>
            <label className="text-sm text-gray-500">Email</label>
            <p className="text-lg">{user.email}</p>
          </div>

          <div>
            <label className="text-sm text-gray-500">User ID</label>
            <p className="text-sm text-gray-600 font-mono">{user.id}</p>
          </div>
        </div>
      </div>

      {/* Debug Info (Optional - để học) */}
      <details className="mt-6">
        <summary className="cursor-pointer text-sm text-gray-500">
          Show Raw Data (Debug)
        </summary>
        <pre className="mt-2 p-4 bg-gray-100 rounded text-xs overflow-auto">
          {JSON.stringify(user, null, 2)}
        </pre>
      </details>
    </div>
  );
}
```

---

## 🎓 BEST PRACTICES

### Rule 1: LUÔN dùng Custom Hook

```tsx
// ❌ BAD - Inline trong component
function Profile() {
  const { data } = useQuery({ queryKey: ["me"], queryFn: fetchMe });
}

// ✅ GOOD - Custom hook
function Profile() {
  const { data } = useUser(); // Reusable, testable, maintainable
}
```

**Lý do:**

- Tái sử dụng logic (DRY principle)
- Dễ test (mock hook, không mock axios)
- Dễ maintain (đổi API chỉ sửa 1 chỗ)
- Semantic naming (đọc code dễ hiểu)

---

### Rule 2: KHÔNG dùng useEffect với query data

```tsx
// ❌ BAD - Duplicate state + Extra render
const { data } = useUser();
const [user, setUser] = useState(null);

useEffect(() => {
  if (data) setUser(data);
}, [data]);

// ✅ GOOD - Dùng trực tiếp
const { data: user } = useUser();

// ✅ GOOD - Transform bằng select
const { data: user } = useQuery({
  queryKey: ["me"],
  queryFn: fetchMe,
  select: (data) => ({
    ...data,
    displayName: data.fullName.toUpperCase(),
  }),
});
```

**Lý do:** Query data ĐÃ là state rồi. Thêm useState = duplicate và khó maintain.

---

### Rule 3: StaleTime theo loại data

```tsx
{
  // Real-time (giá chứng khoán, chat)
  staleTime: 0,

  // Frequent updates (newsfeed, notifications)
  staleTime: 30 * 1000, // 30 giây

  // Semi-stable (product list)
  staleTime: 2 * 60 * 1000, // 2 phút

  // Stable (user profile, settings)
  staleTime: 5 * 60 * 1000, // 5 phút

  // Static (country list, categories)
  staleTime: Infinity,
}
```

---

### Rule 4: KHÔNG duplicate server data vào Zustand

```tsx
// ❌ BAD - Duplicate cache
const { data: user } = useUser();
useEffect(() => {
  userStore.setUser(user); // ❌ Duplicate! React Query đã cache rồi
}, [user]);

// ✅ GOOD - React Query là single source of truth
const { data: user } = useUser(); // Use directly
```

**Nguyên nhân:** React Query ĐÃ LÀ cache store. Duplicate = waste memory + out of sync bugs.

---

## ⚠️ COMMON MISTAKES

### 1. Quên return trong queryFn

**Symptom:** Data luôn `undefined` dù API status 200.

**Cause:** Arrow function có `{}` nhưng không có `return`.

```tsx
// ❌ SAI
queryFn: async () => {
  await api.getUser(); // Quên return!
};

// ✅ ĐÚNG
queryFn: async () => {
  return await api.getUser();
};
```

---

### 2. Key duplicate hoặc quá chung chung

**Issue:** Dùng chung key `['user']` cho cả `getMe` và `getUserById`.

**Symptom:** Data bị lẫn lộn giữa các queries.

```tsx
// ❌ SAI - Quá chung
useQuery({ queryKey: ['user'], ... })

// ✅ ĐÚNG - Specific
useQuery({ queryKey: ['user', 'me'], ... })
useQuery({ queryKey: ['user', userId], ... })
```

---

### 3. Lạm dụng useEffect để set state từ data

**Code:** `useEffect(() => { if (data) setState(data) }, [data])`.

**Issue:** **Anti-pattern lớn nhất!** Tạo ra duplicate state và extra renders.

```tsx
// ❌ SAI
const { data } = useUser();
const [user, setUser] = useState();
useEffect(() => setUser(data), [data]);

// ✅ ĐÚNG
const { data: user } = useUser();
```

---

### 4. Query Key không stable (object mới mỗi render)

**Symptom:** Query fetch liên tục, không bao giờ dừng.

```tsx
// ❌ SAI - filters là object mới mỗi render
const filters = { category };
useQuery({ queryKey: ['products', filters], ... });

// ✅ ĐÚNG - useMemo
const filters = useMemo(() => ({ category }), [category]);

// ✅ BETTER - Destructure trực tiếp
useQuery({ queryKey: ['products', { category }], ... });
```

---

### 5. Catch error trong queryFn

**Issue:** React Query không biết có lỗi, không set `isError = true`.

```tsx
// ❌ SAI - Catch error
queryFn: async () => {
  try {
    return await api.get();
  } catch (error) {
    console.error(error);
    return null; // React Query nghĩ thành công!
  }
};

// ✅ ĐÚNG - Let it throw
queryFn: async () => {
  return await api.get(); // Throw tự nhiên
};
```

---

### 6. Quên Provider

**Symptom:** Error: `No QueryClient set, use QueryClientProvider`.

**Fix:** Bọc `<QueryClientProvider>` ở root của app (main.tsx).

---

### 7. Không kiểm tra DevTools

**Symptom:** Query không chạy nhưng không biết tại sao.

**Fix:** MỞ DEVTOOLS! Xem query state (fresh/stale/fetching/error).

---

### 8. Enabled = false nhưng không biết

**Symptom:** Query không chạy dù component mount.

**Cause:** `enabled: !!undefined` → `false`

```tsx
enabled: (!!accessToken,
  // Log để check:
  console.log("accessToken:", accessToken, "enabled:", !!accessToken));
```

---

## 🐛 DEBUGGING CHECKLIST

### Query không fetch?

- [ ] Check QueryClientProvider có wrap app không
- [ ] Check queryKey và queryFn có đúng không
- [ ] Check `enabled` option (mặc định = true)
- [ ] Mở DevTools xem query có xuất hiện không

### Data undefined?

- [ ] Check queryFn có `return` không
- [ ] Check API response structure
- [ ] Check network tab xem response
- [ ] Log data trong queryFn

### Query fetch liên tục?

- [ ] Check queryKey có stable không (object mới?)
- [ ] Check component có re-render liên tục không
- [ ] Set `refetchOnWindowFocus: false` để test

### Cache không hit?

- [ ] Check queryKey có GIỐNG NHAU không
- [ ] Check gcTime (có bị xóa chưa?)
- [ ] Xem DevTools → Cache tab

---

## 📚 TÀI LIỆU THAM KHẢO

- [TanStack Query Docs](https://tanstack.com/query/latest)
- [React Query Best Practices](https://tkdodo.eu/blog/practical-react-query)
- [Caching Explained](https://tanstack.com/query/latest/docs/react/guides/caching)

---

Happy coding! 🎉
