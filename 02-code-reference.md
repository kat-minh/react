# 💻 FULL CODE REFERENCE: SESSION ROUTING

## 1. Cài đặt React Router v6
```bash
npm install react-router-dom
```

---

## 2. PART 2: ROUTER SETUP & LAYOUT PATTERN

### 2.1 Cấu hình Router (src/router.tsx)
```tsx
import { createBrowserRouter } from "react-router-dom";
import HomePage from "./pages/HomePage";
import LoginPage from "./pages/LoginPage";
import MainLayout from "./components/layouts/MainLayout"; 

// 1. Sử dụng createBrowserRouter - API mới nhất của v6
export const router = createBrowserRouter([
  {
    path: "/", // Đường dẫn gốc
    element: <MainLayout />, // Layout bọc ngoài (Cái nhà)
    children: [
      // Các trang con (Nội thất)
      {
        index: true, // Trang mặc định khi vào "/"
        element: <HomePage />,
      },
      {
        path: "login", // Đường dẫn "/login"
        element: <LoginPage />,
      },
    ],
  },
]);
```

### 2.2 Tạo MainLayout & Outlet (src/components/layouts/MainLayout.tsx)
```tsx
import { Outlet, Link } from "react-router-dom";

export default function MainLayout() {
  return (
    <div className="min-h-screen flex flex-col">
      {/* ===== HEADER - Tường nhà (Cố định) ===== */}
      <header className="bg-blue-600 text-white p-4 sticky top-0 z-50">
        <nav className="max-w-4xl mx-auto flex justify-between items-center">
          <Link to="/" className="text-xl font-bold">
            ShopApp
          </Link>
          <div className="flex gap-4">
            <Link to="/" className="hover:text-blue-200">
              Home
            </Link>
            <Link to="/login" className="hover:text-blue-200">
              Login
            </Link>
          </div>
        </nav>
      </header>

      {/* ===== MAIN CONTENT - Outlet (Thay đổi theo URL) ===== */}
      <main className="flex-1 max-w-4xl mx-auto w-full p-6">
        <Outlet /> {/* 👈 LỖ HỔNG THẦN THÁNH: Nơi render các trang con */}
      </main>

      {/* ===== FOOTER - Nền nhà (Cố định) ===== */}
      <footer className="bg-gray-200 p-6 text-center text-sm text-gray-600">
        © 2024 ShopApp - Piedteam ReactJS Course
      </footer>
    </div>
  );
}
```

### 2.3 Tạo các trang mẫu (Mock Pages)
**HomePage (src/pages/HomePage.tsx):**
```tsx
export default function HomePage() {
  return (
    <div className="text-center py-20">
      <h1 className="text-4xl font-bold text-gray-800">Welcome to ShopApp</h1>
      <p className="text-gray-500 mt-4">This is the Home Page</p>
      <div className="mt-10">
        <img
          src="https://via.placeholder.com/400x200?text=Shop+Banner"
          alt="banner"
          className="mx-auto rounded-lg shadow-md"
        />
      </div>
    </div>
  );
}
```

**LoginPage (src/pages/LoginPage.tsx - Bản đơn giản ban đầu):**
```tsx
export default function LoginPage() {
  return (
    <div className="max-w-sm mx-auto mt-20">
      <div className="bg-white p-8 rounded-xl shadow-lg border">
        <h2 className="text-2xl font-bold mb-6 text-center">Login</h2>
        <form>
          <input type="email" placeholder="Email" className="w-full p-3 border rounded mb-4" />
          <input type="password" placeholder="Password" className="w-full p-3 border rounded mb-4" />
          <button className="w-full bg-blue-600 text-white p-3 rounded font-bold hover:bg-blue-700">
            Sign In
          </button>
        </form>
      </div>
    </div>
  );
}
```

### 2.4 Mount Router vào App (src/main.tsx)
```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { RouterProvider } from "react-router-dom";
import { router } from "./router"; // Import router đã cấu hình
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    {/* Thay App bằng RouterProvider */}
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

---

## 3. PART 3: NAVIGATION & LINKS

### 3.1 Thử nghiệm Link (SPA) vs Anchor Tag (Reload)
```tsx
// ❌ SAI - Bị reload toàn bộ trang, mất State
<a href="/login">Login (Reload trang)</a>

// ✅ ĐÚNG - Chỉ thay đổi Component, không reload
<Link to="/login">Login (SPA)</Link>
```

### 3.2 NavLink - Xử lý Active State
```tsx
import { NavLink } from "react-router-dom";

<nav className="flex gap-4">
  <NavLink
    to="/"
    className={({ isActive }) =>
      isActive ? "text-yellow-300 font-bold underline" : "hover:text-blue-200"
    }
  >
    Home
  </NavLink>

  <NavLink
    to="/login"
    className={({ isActive }) =>
      isActive ? "text-yellow-300 font-bold underline" : "hover:text-blue-200"
    }
  >
    Login
  </NavLink>
</nav>
```

### 3.3 useNavigate - Điều hướng bằng Code logic
```tsx
import { useNavigate } from "react-router-dom";

function OrderComponent() {
  const navigate = useNavigate();

  const handleOrder = () => {
    // 1. Xử lý API đặt hàng...
    console.log("Đặt hàng thành công!");

    // 2. Tự động chuyển trang sau khi xử lý xong
    navigate("/profile");
  };

  return <button onClick={handleOrder}>Đặt hàng ngay</button>;
}
```

---

## 4. PART 4: PROTECTED ROUTES (THE GUARD)

### 4.1 Tạo Guard Component (src/components/guards/RequireAuth.tsx)
```tsx
import { Navigate, Outlet, useLocation } from "react-router-dom";

export default function RequireAuth() {
  // 1. Check Token trong thiết bị (Mock bằng localStorage)
  const token = localStorage.getItem("accessToken");
  const location = useLocation();

  if (!token) {
    // 2. Không có token -> Đá về /login
    // replace: đè lịch sử, state: lưu địa chỉ cũ để quay lại sau
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  // 3. Có token -> Cho đi tiếp vào các tầng bên trong
  return <Outlet />;
}
```

### 4.2 Áp dụng Guard vào Router (Cập nhật src/router.tsx)
```tsx
import RequireAuth from "./components/guards/RequireAuth";
import ProfilePage from "./pages/ProfilePage";

export const router = createBrowserRouter([
  {
    path: "/",
    element: <MainLayout />,
    children: [
      // === PUBLIC ROUTES (Ai cũng vào được) ===
      { index: true, element: <HomePage /> },
      { path: "login", element: <LoginPage /> },

      // === PROTECTED ROUTES (Cần đăng nhập) ===
      {
        element: <RequireAuth />, // Guard bọc ở đây
        children: [
          { path: "profile", element: <ProfilePage /> },
          { path: "settings", element: <SettingsPage /> },
        ],
      },
    ],
  },
]);
```

### 4.3 Trang cá nhân (src/pages/ProfilePage.tsx)
```tsx
export default function ProfilePage() {
  return (
    <div className="p-10 bg-white rounded-lg shadow-md">
      <h1 className="text-3xl font-bold mb-4">My Profile</h1>
      <p className="text-gray-600">Chào mừng bạn đã trở lại!</p>
    </div>
  );
}
```

### 4.4 Xử lý Hậu Đăng Nhập (Redirect Back logic)
```tsx
import { useNavigate, useLocation } from "react-router-dom";

export default function LoginPage() {
  const navigate = useNavigate();
  const location = useLocation();

  // 1. Lấy địa chỉ user muốn đến dự định ban đầu
  const from = location.state?.from?.pathname || "/";

  const handleLogin = () => {
    // Mock lưu token
    localStorage.setItem("accessToken", "fake-token-123456");

    // 2. Login xong quay lại đúng trang cũ
    navigate(from, { replace: true });
  };

  return (
    <div className="p-8">
      {location.state?.from && (
        <p className="text-orange-600 mb-4">
          ⚠️ Bạn cần đăng nhập để truy cập {location.state.from.pathname}
        </p>
      )}
      <button onClick={handleLogin} className="bg-blue-600 text-white p-3 rounded">
        Đăng nhập & Tiếp tục
      </button>
    </div>
  );
}
```
