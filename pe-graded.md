# Chấm Điểm Phần Thực Hành React 
---

## Rubric gốc

| Tiêu chí | Tối đa |
|---|:---:|
| Cấu trúc & Setup chuẩn | 1.0 |
| Authentication Flow | 1.5 |
| Fetching Data (Query) | 1.0 |
| Form Management (Mutation + Zod) | 1.5 |
| UI/UX Aesthetics | 0.5 |
| Deploy & Bonus | 0.5 |
| **Tổng** | **6.0** |

---

## Bảng tổng hợp điểm

| Học viên | Setup (1.0) | Auth (1.5) | Query (1.0) | Form/Mutation/Zod (1.5) | UI/UX (0.5) | Deploy (0.5) | Tổng (6.0) | Điểm /10 |
| -------- | ----------: | ---------: | ----------: | ----------------------: | ----------: | -----------: | ---------: | --------: |
| cuong    |         0.4 |        0.3 |         0.0 |                     0.0 |         0.1 |          0.0 |    **0.8** |   **1.3** |
| duy      |         0.5 |        1.3 |         0.2 |                     0.5 |         0.3 |          0.0 |    **2.8** |   **4.7** |
| huy      |         0.3 |        0.4 |         0.0 |                     0.4 |         0.3 |          0.0 |    **1.4** |   **2.3** |
| khang    |         0.7 |        1.3 |         0.0 |                     0.5 |         0.3 |          0.0 |    **2.8** |   **4.7** |
| linh     |         0.7 |        0.9 |         0.3 |                     0.4 |         0.4 |          0.2 |    **2.9** |   **4.8** |
| mhoang   |         0.5 |        0.1 |         0.2 |                     0.0 |         0.2 |          0.0 |    **1.0** |   **1.7** |
| thinh    |         0.6 |        1.0 |         0.0 |                     0.2 |         0.3 |          0.1 |    **2.2** |   **3.7** |
| truong   |         0.3 |        0.2 |         0.0 |                     0.0 |         0.0 |          0.0 |    **0.5** |   **0.8** |

> 💡 **Quy đổi:** Điểm /10 = Tổng (6.0) ÷ 6 × 10, làm tròn 1 chữ số thập phân.


---

## Nhận xét chi tiết từng bài

---

### 1) cuong — Main-Test-Final-Project

**Tổng: 0.8 / 6.0** 

#### Cấu trúc & Setup (0.4 / 1.0)
- ✅ Có `src/features/auth/`, `src/stores/`, `src/provider/` — Feature-Based sơ khai
- ❌ Chỉ có auth feature, không có employees feature, không có shared
- ❌ Thiếu React Query, RHF/Zod trong package.json
- **Lý do trừ:** Thiếu 3/5 library bắt buộc → setup không đạt chuẩn đề

#### Authentication Flow (0.3 / 1.5)
- ✅ Zustand `persist()` middleware **có dùng đúng**: [`auth.store.ts`](cuong/Main-Test-Final-Project/src/stores/auth.store.ts)
- ❌ Router chỉ có `/login`: [`router.tsx`](cuong/Main-Test-Final-Project/src/provider/router.tsx) — không có dashboard, không có protected route
- ❌ LoginForm là HTML thuần, không có logic gọi API, không có action lưu token: [`LoginForm.tsx`](cuong/Main-Test-Final-Project/src/features/auth/components/LoginForm.tsx) — chỉ là `<form>` tĩnh không xử lý gì
- ❌ Không có logout
- **Chỉ có persist store → 0.3**

#### Fetching Data (0.0 / 1.0)
- ❌ Không có `@tanstack/react-query`, không có `useQuery`

#### Form Management (0.0 / 1.5)
- ❌ Không có RHF, Zod. LoginForm là HTML stub rỗng

#### UI/UX (0.1 / 0.5)
- ❌ Gần như không có UI thực sự, chỉ có LoginForm stub

#### Deploy (0.0 / 0.5)
- ❌ Không có gì

---

### 2) duy — Test-25-03

**Tổng: 2.8 / 6.0** 

#### Cấu trúc & Setup (0.5 / 1.0)
- ✅ Đủ bộ thư viện: react-query, RHF, Zod, Zustand, Axios: [`package.json`](duy/Test-25-03/package.json)
- ❌ **Cấu trúc KHÔNG phải Feature-Based** — dùng `src/components/`, `src/store/`, `src/app/` thay vì `src/features/...`
- ❌ Không tách Types, Services ra riêng theo chuẩn đề
- **Trừ 0.5 do cấu trúc sai chuẩn**

#### Authentication Flow (1.3 / 1.5)
- ✅ Zustand `persist()` + `createJSONStorage` chuẩn: [`useAppStore.ts`](duy/Test-25-03/src/store/useAppStore.ts)
- ✅ Login form dùng RHF + `zodResolver` + `formState.errors`: [`LoginPage.tsx`](duy/Test-25-03/src/components/LoginPage.tsx)
- ✅ GET `/login/1` đúng trick đề: `api.get("/login/1")`
- ✅ Protected Route với `location.state.from` redirect về URL cũ: [`ProtectedRoute.tsx`](duy/Test-25-03/src/components/ProtectedRoute.tsx)
- ✅ Logout (`clearAuth`) hoạt động
- ❌ Không có Sidebar, không có Layout với Outlet
- **Trừ 0.2 do thiếu Layout/Outlet**

#### Fetching Data (0.2 / 1.0)
- ✅ `useQuery` có dùng trong `HomePage.tsx` với `isLoading`, `isError` xử lý
- ❌ Query fetch **`/login/1`** (resource token), **không phải employees** — không đúng tính năng
- ❌ Không có employee list dùng `useQuery`
- **Rubric yêu cầu fetch employees → chỉ cho 0.2 vì biết dùng useQuery đúng cú pháp**

#### Form Management (0.5 / 1.5)
- ✅ Login form: RHF + Zod + hiển thị lỗi — đạt
- ❌ Không có employee feature → không có create/delete mutation
- ❌ Không có `useMutation`, `invalidateQueries` cho employees
- **Chỉ tính phần login form đạt**

#### UI/UX (0.3 / 0.5)
- ✅ Có CSS tổ chức, có auth-card, dashboard-card
- ❌ Không có sidebar, UI còn đơn giản

#### Deploy (0.0 / 0.5)
- ❌ Không có 

---

### 3) huy — final-test

**Tổng: 1.4 / 6.0**

#### Cấu trúc & Setup (0.3 / 1.0)
- ✅ Có `src/features/auth/` đúng Feature-Based
- ❌ Thiếu employees feature, không có shared
- ❌ Thiếu Zustand, React Query trong dependencies

#### Authentication Flow (0.4 / 1.5)
- ✅ LoginForm dùng **đúng chuẩn RHF + zodResolver**: `useForm`, `register`, `handleSubmit`, `formState.errors` — KỸ THUẬT TỐT: [`LoginForm.tsx`](huy/final-test/src/features/auth/components/LoginForm.tsx)
- ❌ `App.tsx` chỉ render `<LoginPage />` — **không có router, không có protected route, không có dashboard**
- ❌ Không có Zustand store (auth service trả token nhưng không lưu vào store)
- ❌ Logic login dùng `POST /login` — **sai yêu cầu đề** (phải GET `/login/1`): [`auth.service.ts`](huy/final-test/src/features/auth/services/auth.service.ts)
- ❌ Không có logout
- **LoginForm là điểm sáng duy nhất nhưng auth flow không hoàn chỉnh**

#### Fetching Data (0.0 / 1.0)
- ❌ Không có React Query

#### Form Management (0.4 / 1.5)
- ✅ Login Form RHF + Zod đạt chuẩn cao
- ❌ Không có employee form, không có useMutation
- **Chỉ tính login form**

#### UI/UX (0.3 / 0.5)
- ✅ Có CSS riêng cho login form, có styling

#### Deploy (0.0 / 0.5)
- ❌ Không có gì

---

### 4) khang — REACTtes1

**Tổng: 2.8 / 6.0** 

#### Cấu trúc & Setup (0.7 / 1.0)
- ✅ Cấu trúc `src/features/auth/`, `src/shared/` — Feature-Based đúng hướng
- ✅ Có RHF, Zod, Zustand, Axios, Router
- ❌ **Không có `@tanstack/react-query`** trong `package.json` — thiếu library bắt buộc: [`package.json`](khang/REACTtes1/package.json)
- ❌ Thiếu employees feature
- **Trừ 0.3**

#### Authentication Flow (1.3 / 1.5)
- ✅ Zustand `persist()` + `createJSONStorage` **chuẩn**: [`store.ts`](khang/REACTtes1/src/features/auth/store.ts)
- ✅ ProtectedRoute bọc MainLayout + Outlet: [`AppRouter.tsx`](khang/REACTtes1/src/app/AppRouter.tsx)
- ✅ Login form RHF + Zod: [`LoginForm.tsx`](khang/REACTtes1/src/features/auth/components/LoginForm.tsx)
- ✅ Logout `clearAuth()` có
- ❌ Dashboard và Employees là **`<div>Dashboard</div>` / `<div>Employees</div>` placeholder** — chưa có nội dung thực
- ❌ Không có redirect về URL cũ sau khi login
- **Trừ 0.2 do dashboard/employees là placeholder**

#### Fetching Data (0.0 / 1.0)
- ❌ Không có `@tanstack/react-query` → không thể dùng `useQuery`

#### Form Management (0.5 / 1.5)
- ✅ Login form RHF + Zod + `formState.errors` hiển thị — đạt
- ❌ Không có employee form CRUD, không có `useMutation`, `invalidateQueries`

#### UI/UX (0.3 / 0.5)
- ✅ Có layout cơ bản

#### Deploy (0.0 / 0.5)
- ❌ Không có gì

---

### 5) linh — final_test_react1

**Tổng: 2.9 / 6.0**

#### Cấu trúc & Setup (0.7 / 1.0)
- ✅ Có đủ thư viện cần thiết: react-query, Zod, Zustand, Axios: [`package.json`](linh/final_test_react1/package.json)
- ❌ **Không có `react-hook-form`** trong package.json — thiếu library bắt buộc
- ❌ Cấu trúc `src/pages/`, `src/hooks/` — **không phải Feature-Based** theo chuẩn `src/features/...`

#### Authentication Flow (0.9 / 1.5)
- ❌ **Zustand store KHÔNG dùng `persist()` middleware** — tự quản lý `localStorage` thủ công bằng `getStoredUser()`, `getStoredAccessToken()`, `saveSession()`, `clearSession()`: [`authStore.ts`](linh/final_test_react1/src/store/authStore.ts)
  > Đây là vi phạm **yêu cầu tường minh** của rubric: *"Zustand persist hoạt động"*
- ✅ ProtectedRoute redirect về `/login`: [`ProtectedRoute.tsx`](linh/final_test_react1/src/components/common/ProtectedRoute.tsx)
- ✅ Logout `clearSession()` + clear store hoạt động
- ✅ Layout + Outlet + Sidebar 2 tab (Dashboard & Employees) đủ
- ❌ Login không redirect về URL cũ (hardcode về `/`)
- ❌ Login form dùng `useState` + `loginSchema.safeParse()` thủ công — **không dùng RHF**: [`LoginPage.tsx`](linh/final_test_react1/src/pages/LoginPage.tsx)
- **Trừ 0.4 do thiếu persist + 0.2 do form không dùng RHF**

#### Fetching Data (0.3 / 1.0)
- ✅ `QueryClientProvider` setup đúng: [`main.tsx`](linh/final_test_react1/src/main.tsx)
- ❌ Employees **KHÔNG dùng `useQuery`** — dùng `useCrud` hook (custom hook với `useEffect` + `fetch`): [`useCrud.ts`](linh/final_test_react1/src/hooks/useCrud.ts)
- ✅ Có xử lý loading/error state
- ❌ `useMutation` trong `useEmployee` chỉ wrap `deleteItem` từ useCrud, không phải mutation đúng nghĩa theo đề
- **Chỉ cho 0.3 vì QueryClient setup đúng nhưng không dùng đúng API của React Query**

#### Form Management (0.4 / 1.5)
- ❌ **Không có `react-hook-form`** trong package.json → không thể dùng RHF
- ❌ Login form: `useState` + `safeParse()` thủ công — không có `useForm`, `register`, `handleSubmit`, `formState.errors`
- ❌ Employee form: `useState` thuần, validate bằng `!payload.fullName` — không có Zod validation cho employees
- ❌ Không có `useMutation` cho create/delete employees đúng chuẩn
- ❌ Không có `invalidateQueries`
- ✅ Có Zod schema cho login — điểm cộng nhỏ
- ✅ Có CRUD employees hoạt động qua useCrud
- **Thiếu RHF và useMutation/invalidateQueries → 0.4**

#### UI/UX (0.4 / 0.5)
- ✅ Có CSS, bảng employees đẹp, có loading/error state UI tốt
- ✅ Có `vercel.json` — chuẩn bị deploy

#### Deploy (0.2 / 0.5)
- ✅ Có `vercel.json` cấu hình đúng SPA rewrites: [`vercel.json`](linh/final_test_react1/vercel.json)
- ❌ Link Deploy không dùng được


---

### 6) mhoang — kiemTra

**Tổng: 1.0 / 6.0** 

#### Cấu trúc & Setup (0.5 / 1.0)
- ✅ Có React Query provider trong App
- ✅ Setup dự án, có UI libs
- ❌ Auth store **lỗi syntax nghiêm trọng** — sẽ crash khi build: [`store.ts`](mhoang/kiemTra/src/features/auth/store.ts)
  ```ts
  acccessToken: null;   // typo + dùng ; thay vì ,
  clearAuth: set({...}) // không bọc trong arrow function → crash
  ```
- ❌ Schema Zod để trống

#### Authentication Flow (0.1 / 1.5)
- ❌ Auth store có lỗi syntax → không thể chạy
- ❌ Router chỉ có login, không có protected route / dashboard / employees
- ❌ Không có login logic thực

#### Fetching Data (0.2 / 1.0)
- ✅ QueryClientProvider setup đúng trong App
- ❌ Không có `useQuery` thực sự dùng

#### Form Management (0.0 / 1.5)
- ❌ Schema Zod trống, không có RHF, không có mutation

#### UI/UX (0.2 / 0.5)
- ✅ Có UI libs setup

#### Deploy (0.0 / 0.5)
- ❌ Không có gì

---

### 7) thinh — pe-react

**Tổng: 2.2 / 6.0** 

> ⚠️ **Bài có feature CRUD hoạt động nhưng hoàn toàn không dùng đúng stack theo đề.**

> 🚨 **LƯU Ý QUAN TRỌNG:** `authStore.ts` và `EmployeesPage.tsx` của thinh **gần như copy y hệt** bài của linh. Cần xem xét vấn đề học thuật trung thực (academic integrity).

#### Cấu trúc & Setup (0.6 / 1.0)
- ✅ Cấu trúc `src/features/auth/`, `src/features/employee/`, `src/shared/` — **Feature-Based tốt nhất nhóm**
- ✅ Có Zustand, Axios, Router
- ❌ **Không có `@tanstack/react-query`, `react-hook-form`, `zod`** trong package.json: [`package.json`](thinh/pe-react/package.json)
- **Trừ 0.4 do thiếu 3/5 library bắt buộc**

#### Authentication Flow (1.0 / 1.5)
- ✅ Protected Route + Layout + Outlet đầy đủ: [`App.tsx`](thinh/pe-react/src/App.tsx)
- ✅ Login đúng trick GET `/login/1`: [`authService.ts`](thinh/pe-react/src/features/auth/services/authService.ts)
- ✅ Logout hoạt động
- ✅ Sidebar 2 tab (Home & Employees)
- ❌ **Auth store KHÔNG dùng `persist()` middleware** — tự quản lý localStorage thủ công: [`authStore.ts`](thinh/pe-react/src/shared/store/authStore.ts)
  > Code này **gần như copy y hệt** `authStore.ts` của linh
- ❌ Không redirect về URL cũ sau login
- **Trừ 0.5 do thiếu persist middleware**

#### Fetching Data (0.0 / 1.0)
- ❌ Không có `@tanstack/react-query` → không có `useQuery`
- ❌ Employees dùng custom `useCrud` hook (giống hệt linh)

#### Form Management (0.2 / 1.5)
- ❌ Không có `react-hook-form`, `zod` trong package.json
- ❌ Không có `useMutation`, `invalidateQueries`
- ✅ Employee CRUD có hoạt động (create, update, delete) qua useCrud
- ✅ Validation cơ bản bằng `!payload.fullName`
- **Chỉ cho 0.2 vì CRUD logic có hoạt động nhưng sai hoàn toàn stack yêu cầu**
  > `EmployeesPage.tsx` **gần như copy y hệt** bài linh

#### UI/UX (0.3 / 0.5)
- ✅ Có App.css, layout có header/sidebar

#### Deploy (0.1 / 0.5)
- ❌ vercel.json

---

### 8) truong — test1

**Tổng: 0.5 / 6.0** 

#### Cấu trúc & Setup (0.3 / 1.0)
- ✅ Có `src/features/auth/`, `src/store/`, `src/provider/`
- ❌ Thiếu hầu hết library
- ❌ **App.tsx trả về `<></>`** — không mount gì cả: [`App.tsx`](truong/test1/src/App.tsx)

#### Authentication Flow (0.2 / 1.5)
- ✅ Zustand `persist()` middleware có setup: [`authStore.ts`](truong/test1/src/store/authStore.ts)
- ❌ Route không được mount trong App
- ❌ LoginForm là `<h1>Login Form</h1>` placeholder
- ❌ Không có logic nào chạy được
- **Chỉ có persist store setup → 0.2**

#### Fetching Data (0.0 / 1.0)
- ❌ Không có React Query

#### Form Management (0.0 / 1.5)
- ❌ Không có gì

#### UI/UX (0.0 / 0.5)
- ❌ App trống, không có UI

#### Deploy (0.0 / 0.5)
- ❌ Không có gì

---


## Kết luận & Lỗi phổ biến

### Lỗi phổ biến:
1. **Zustand persist:** 5/8 bài không dùng `persist()` middleware đúng cách (linh, thinh tự quản lý localStorage; mhoang syntax error; huy/cuong không có)
2. **Employees feature thiếu hoàn toàn:** duy, khang, cuong, huy, truong
3. **Không dùng RHF:** linh, thinh, cuong, truong
4. **Không dùng React Query:** thinh, khang, cuong, truong, huy

### 🚨 Cảnh báo học thuật:
- `authStore.ts` của **thinh** và **linh** gần như **copy y hệt nhau từng dòng** (cùng hàm `getStoredUser`, `getStoredAccessToken`, `saveSession`, `clearSession`, cùng `useAuth` wrapper function)
- `EmployeesPage.tsx` của **thinh** và **linh** cũng **gần như giống nhau hoàn toàn**


---

