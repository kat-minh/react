# SHADCN/UI & TAILWIND V4 - STUDENT GUIDE

## 🎯 Mục tiêu học tập

Nâng cấp UI của app lên chuẩn professional với:

- **shadcn/ui**: Copy-paste component system (không phải npm package)
- **Tailwind v4**: CSS-first configuration với theme tokens
- **Sonner**: Toast notifications đẹp và mượt
- **NavigationMenu**: Professional navigation với accessibility built-in

**Outcome:** App UI nhìn như sản phẩm thật, không còn "mùi bài tập học sinh"

---

## 🧠 Khái niệm cốt lõi

### shadcn/ui vs Traditional UI Libraries

| Feature            | shadcn/ui                          | Material UI / Ant Design         |
| ------------------ | ---------------------------------- | -------------------------------- |
| **Install**        | Copy code vào project              | `npm install` package lớn        |
| **Customization**  | Edit trực tiếp file `.tsx`         | Override styles qua props/config |
| **Ownership**      | Code là của bạn 100%               | Code trong node_modules          |
| **Bundle Size**    | Chỉ import components dùng         | Cả library vào bundle            |
| **Learning Curve** | Đơn giản - chỉ là React components | Phải học API của library         |

### Tailwind v4 - CSS-first Approach

**Cũ (v3):**

```js
// tailwind.config.js
module.exports = {
  theme: {
    colors: {
      brand: "#3b82f6",
    },
  },
};
```

**Mới (v4):**

```css
/* index.css */
@theme {
  --color-brand: #3b82f6;
}
```

**Lợi ích:** Gần gũi hơn với CSS standards, faster compilation, zero runtime overhead

---

## 🛠️ PHASE 1: SETUP SHADCN/UI

### Bước 1: Init shadcn/ui

```bash
npx shadcn@latest init
```

**Questions:**

- **Style:** Default
- **Base Color:** Slate (hoặc màu bạn thích)
- **CSS Variables:** Yes ✅ (để dễ customize)
- **Import alias:** Mặc định `@/components` và `@/lib`

Lệnh này sẽ:

- Tạo `components.json` config
- Setup `src/lib/utils.ts` với `cn()` helper
- Cập nhật `tsconfig.json` với path aliases
- Inject theme vào `src/index.css`

### Bước 2: Verify Setup

Check `src/index.css` có đoạn này:

```css
@import "tailwindcss";

@theme {
  --color-background: #ffffff;
  --color-foreground: #0a0a0a;
  --color-primary: #0f172a;
  --color-destructive: #ef4444;
  /* ... nhiều tokens khác */
}
```

### Bước 3: Install Core Components

```bash
# Essential UI components
npx shadcn@latest add button input card label

# Dialogs & Notifications
npx shadcn@latest add alert-dialog sonner

# Navigation & Layout
npx shadcn@latest add navigation-menu badge separator
```

Các file sẽ được tạo trong `src/components/ui/`:

- `button.tsx`
- `input.tsx`
- `card.tsx`
- `label.tsx`
- `alert-dialog.tsx`
- `sonner.tsx`
- `navigation-menu.tsx`
- `badge.tsx`
- `separator.tsx`

---

## 🛠️ PHASE 2: UPGRADE LOGIN PAGE

### Before (HTML hardcoded):

```tsx
<div className="bg-gray-50 p-4">
  <div className="bg-white rounded-lg p-6 shadow">
    <h1 className="text-2xl mb-4">Login</h1>
    <input type="email" className="border p-2 w-full" />
    <button className="bg-blue-500 text-white p-2 rounded">Login</button>
  </div>
</div>
```

❌ Problems:

- Hardcoded colors (`bg-blue-500`)
- No dark mode support
- No loading states
- Inconsistent spacing

### After (shadcn components):

```tsx
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import {
  Card,
  CardContent,
  CardHeader,
  CardTitle,
  CardDescription,
} from "@/components/ui/card";
import { toast } from "sonner";
import { Loader2 } from "lucide-react";

export default function LoginPage() {
  const [email, setEmail] = useState("test@example.com");
  const [password, setPassword] = useState("password123");
  const [loading, setLoading] = useState(false);

  const handleLogin = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);

    try {
      const { tokens } = await authApi.login({ email, password });
      setTokens(tokens.accessToken, tokens.refreshToken);

      toast.success("Đăng nhập thành công!", {
        description: "Chào mừng bạn quay lại.",
      });

      navigate("/profile");
    } catch (err: any) {
      toast.error("Đăng nhập thất bại", {
        description: err.response?.data?.message || "Lỗi không xác định",
      });
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="flex items-center justify-center min-h-screen bg-linear-to-br from-slate-50 to-slate-100 dark:from-slate-900 dark:to-slate-800 p-4">
      <Card className="w-full max-w-md shadow-lg">
        <CardHeader className="text-center">
          <CardTitle className="text-3xl font-bold">Welcome Back</CardTitle>
          <CardDescription>Đăng nhập để tiếp tục</CardDescription>
        </CardHeader>
        <CardContent>
          <form onSubmit={handleLogin} className="space-y-4">
            <div className="space-y-2">
              <Label htmlFor="email">Email</Label>
              <Input
                id="email"
                type="email"
                placeholder="m@example.com"
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                required
              />
            </div>

            <div className="space-y-2">
              <Label htmlFor="password">Password</Label>
              <Input
                id="password"
                type="password"
                placeholder="••••••••"
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                required
              />
            </div>

            <Button type="submit" className="w-full" disabled={loading}>
              {loading && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
              {loading ? "Đang đăng nhập..." : "Login"}
            </Button>
          </form>
        </CardContent>
      </Card>
    </div>
  );
}
```

✅ Improvements:

- Theme tokens thay vì hardcoded colors
- Dark mode tự động
- Loading state với spinner
- Toast notifications
- Responsive layout
- Proper spacing với `space-y-*`

---

## 🛠️ PHASE 3: TOAST NOTIFICATIONS

### Setup Toaster

Thêm vào `src/main.tsx`:

```tsx
import { Toaster } from "@/components/ui/sonner";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <>
    <RouterProvider router={router} />
    <Toaster position="top-right" richColors />
  </>,
);
```

### Usage Examples

```tsx
import { toast } from "sonner";

// Success
toast.success("Thành công!", {
  description: "Hành động đã hoàn tất.",
});

// Error
toast.error("Lỗi!", {
  description: "Có gì đó không đúng.",
});

// Info
toast.info("Thông tin", {
  description: "Đây là thông báo.",
});

// Loading (Promise-based)
toast.promise(fetchData(), {
  loading: "Đang tải...",
  success: "Tải xong!",
  error: "Tải thất bại",
});

// With action button
toast("Có cập nhật mới", {
  action: {
    label: "Xem",
    onClick: () => console.log("Clicked"),
  },
});
```

---

## 🛠️ PHASE 4: NAVIGATION MENU & ALERT DIALOG

### Professional Navigation

Update `src/components/layouts/MainLayout.tsx`:

```tsx
import { Outlet, Link, useNavigate, useLocation } from "react-router-dom";
import { useAuthStore } from "../../stores/auth.store";
import { Button } from "@/components/ui/button";
import {
  NavigationMenu,
  NavigationMenuItem,
  NavigationMenuLink,
  NavigationMenuList,
  navigationMenuTriggerStyle,
} from "@/components/ui/navigation-menu";
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from "@/components/ui/alert-dialog";
import { toast } from "sonner";
import { cn } from "@/lib/utils";

export default function MainLayout() {
  const token = useAuthStore((state) => state.accessToken);
  const clearTokens = useAuthStore((state) => state.clearTokens);
  const navigate = useNavigate();
  const location = useLocation();

  const handleLogout = () => {
    clearTokens();
    toast.success("Đã đăng xuất", {
      description: "Hẹn gặp lại bạn!",
    });
    navigate("/login");
  };

  return (
    <div className="min-h-screen flex flex-col">
      {/* HEADER with glassmorphism */}
      <header className="border-b bg-background/95 backdrop-blur supports-[backdrop-filter]:bg-background/60 sticky top-0 z-50">
        <div className="max-w-4xl mx-auto flex h-16 items-center justify-between px-4">
          <Link
            to="/"
            className="text-xl font-bold tracking-tight hover:text-primary transition-colors"
          >
            ShopApp
          </Link>

          <div className="flex items-center gap-4">
            {/* Navigation Menu */}
            <NavigationMenu>
              <NavigationMenuList>
                <NavigationMenuItem>
                  <Link to="/">
                    <NavigationMenuLink
                      className={cn(
                        navigationMenuTriggerStyle(),
                        location.pathname === "/" &&
                          "bg-primary/10 text-primary font-medium",
                      )}
                    >
                      Home
                    </NavigationMenuLink>
                  </Link>
                </NavigationMenuItem>
                <NavigationMenuItem>
                  <Link to="/profile">
                    <NavigationMenuLink
                      className={cn(
                        navigationMenuTriggerStyle(),
                        location.pathname === "/profile" &&
                          "bg-primary/10 text-primary font-medium",
                      )}
                    >
                      Profile
                    </NavigationMenuLink>
                  </Link>
                </NavigationMenuItem>
              </NavigationMenuList>
            </NavigationMenu>

            {/* Logout Dialog */}
            {token ? (
              <AlertDialog>
                <AlertDialogTrigger asChild>
                  <Button variant="ghost" size="sm">
                    Logout
                  </Button>
                </AlertDialogTrigger>
                <AlertDialogContent>
                  <AlertDialogHeader>
                    <AlertDialogTitle>Bạn muốn đăng xuất?</AlertDialogTitle>
                    <AlertDialogDescription>
                      Phiên đăng nhập sẽ kết thúc và bạn cần login lại.
                    </AlertDialogDescription>
                  </AlertDialogHeader>
                  <AlertDialogFooter>
                    <AlertDialogCancel>Hủy</AlertDialogCancel>
                    <AlertDialogAction
                      onClick={handleLogout}
                      variant="destructive"
                    >
                      Đăng xuất
                    </AlertDialogAction>
                  </AlertDialogFooter>
                </AlertDialogContent>
              </AlertDialog>
            ) : (
              <Button variant="ghost" size="sm" asChild>
                <Link to="/login">Login</Link>
              </Button>
            )}
          </div>
        </div>
      </header>

      <main className="flex-1 max-w-4xl mx-auto w-full p-6">
        <Outlet />
      </main>

      <footer className="border-t bg-muted/40 py-6 text-center text-sm text-muted-foreground">
        <p>© 2024 ShopApp - ReactJS Course</p>
        <p className="mt-2">Built with shadcn/ui & Tailwind CSS</p>
      </footer>
    </div>
  );
}
```

**Key Concepts:**

1. **Active State Tracking:**

   ```tsx
   location.pathname === "/" && "bg-primary/10 text-primary font-medium";
   ```

2. **`cn()` Utility:** Merge Tailwind classes properly

   ```tsx
   cn(
     navigationMenuTriggerStyle(), // Base styles
     isActive && "bg-primary/10", // Conditional styles
   );
   ```

3. **`asChild` Prop:** Prevents nested buttons
   ```tsx
   <AlertDialogTrigger asChild>
     <Button>Click me</Button> {/* Merges into one button */}
   </AlertDialogTrigger>
   ```

---

## 🎨 THEME TOKENS (Quan trọng!)

### ❌ ĐỪNG hardcode colors:

```tsx
// BAD
<div className="bg-blue-500 text-white">
<Button className="bg-red-600 hover:bg-red-700">Delete</Button>
```

### ✅ Dùng theme tokens:

```tsx
// GOOD
<div className="bg-primary text-primary-foreground">
<Button variant="destructive">Delete</Button>
```

### Common Theme Tokens:

| Token                   | Usage             | Dark Mode   |
| ----------------------- | ----------------- | ----------- |
| `bg-background`         | Main background   | Auto switch |
| `text-foreground`       | Main text         | Auto switch |
| `bg-primary`            | Brand color       | Auto switch |
| `text-primary`          | Primary text      | Auto switch |
| `bg-destructive`        | Danger/Delete     | Auto switch |
| `bg-muted`              | Subtle background | Auto switch |
| `text-muted-foreground` | Secondary text    | Auto switch |
| `border`                | Border color      | Auto switch |

---

## 🛠️ PHASE 5: UPDATE OTHER PAGES

### RegisterPage

```tsx
import { User, Mail, Lock } from "lucide-react";

<Card className="w-full max-w-md shadow-lg">
  <CardHeader className="text-center">
    <CardTitle className="text-3xl font-bold">Tạo tài khoản</CardTitle>
    <CardDescription>Đăng ký để bắt đầu</CardDescription>
  </CardHeader>
  <CardContent>
    <form onSubmit={handleSubmit} className="space-y-4">
      <div className="space-y-2">
        <Label htmlFor="fullname">
          <User className="inline w-4 h-4 mr-1" />
          Full Name
        </Label>
        <Input
          id="fullname"
          value={formData.fullname}
          onChange={(e) => handleChange("fullname", e.target.value)}
          className={validationErrors.fullname ? "border-destructive" : ""}
        />
        {validationErrors.fullname && (
          <p className="text-destructive text-xs">
            {validationErrors.fullname}
          </p>
        )}
      </div>
      {/* More fields... */}
    </form>
  </CardContent>
</Card>;
```

### ProfilePage

```tsx
import { Badge } from "@/components/ui/badge";
import { Separator } from "@/components/ui/separator";

<Card className="shadow-lg">
  <CardHeader>
    <CardTitle className="text-3xl flex items-center gap-2">
      {user?.fullName}
      <Badge variant="secondary">
        <ShieldCheck className="w-3 h-3 mr-1" />
        Verified
      </Badge>
    </CardTitle>
    <CardDescription className="flex items-center gap-2">
      <Mail className="w-4 h-4" />
      {user?.email}
    </CardDescription>
  </CardHeader>

  <Separator />

  <CardContent className="pt-6">
    <div className="flex items-center justify-between p-4 bg-muted/50 rounded-lg">
      <span className="font-medium">User ID</span>
      <code className="text-sm font-mono bg-background px-2 py-1 rounded">
        {user?.id}
      </code>
    </div>
  </CardContent>
</Card>;
```

### HomePage - Cards Grid

```tsx
<div className="grid grid-cols-1 md:grid-cols-3 gap-8">
  <Card>
    <CardHeader>
      <div className="w-10 h-10 bg-primary/10 text-primary rounded-lg flex items-center justify-center mb-2 font-bold">
        1
      </div>
      <CardTitle>Clean Routing</CardTitle>
    </CardHeader>
    <CardContent>
      <p className="text-muted-foreground text-sm">
        Organized routes using createBrowserRouter...
      </p>
    </CardContent>
  </Card>
  {/* More cards... */}
</div>
```

---

## ⚠️ COMMON MISTAKES

### 1. Missing Toaster Component

**Symptom:** `toast.success()` không hiện gì

**Fix:** Thêm `<Toaster />` vào `main.tsx`:

```tsx
<Toaster position="top-right" richColors />
```

### 2. Hardcoded Colors

**Bad:**

```tsx
className = "bg-red-500 hover:bg-red-600";
```

**Good:**

```tsx
variant = "destructive";
// or
className = "bg-destructive hover:bg-destructive/90";
```

### 3. Nested Buttons

**Error:** `Warning: validateDOMNesting: <button> cannot appear as a descendant of <button>`

**Fix:** Use `asChild` prop:

```tsx
<AlertDialogTrigger asChild>
  <Button>Click me</Button>
</AlertDialogTrigger>
```

### 4. Tailwind v4 Gradient Syntax

**Old (v3):**

```tsx
bg - gradient - to - br;
```

**New (v4):**

```tsx
bg - linear - to - br;
```

### 5. NavigationMenu Active State Not Working

**Issue:** Using `asChild` with `NavLink` className function doesn't work

**Fix:** Use regular `Link` + `location.pathname` check:

```tsx
<Link to="/">
  <NavigationMenuLink
    className={cn(
      navigationMenuTriggerStyle(),
      location.pathname === "/" && "bg-primary/10",
    )}
  >
    Home
  </NavigationMenuLink>
</Link>
```

---

## 🎯 CHECKLIST

Sau khi hoàn thành session này, bạn phải đạt được:

- [ ] `src/components/ui/` có đầy đủ 9 components (button, input, card, label, alert-dialog, sonner, navigation-menu, badge, separator)
- [ ] Login/Register/Profile pages dùng shadcn components
- [ ] Header dùng NavigationMenu với active state
- [ ] Logout có AlertDialog confirmation
- [ ] Toast notifications hoạt động (success/error)
- [ ] KHÔNG còn hardcoded colors (`bg-blue-500`, `text-red-600`)
- [ ] Tất cả dùng theme tokens (`bg-primary`, `text-destructive`)
- [ ] Dark mode hoạt động (nếu có ThemeProvider)

---

## � BÀI TẬP VỀ NHÀ

### Mục tiêu: Hoàn thành 100% app với shadcn/ui

**Reference App:** https://learning-react-delta-opal.vercel.app/

Truy cập link trên để xem demo app hoàn chỉnh. Nhiệm vụ của bạn là làm cho app của mình giống y hệt về mặt UI.

### Requirements:

#### 1. **Loại bỏ toàn bộ hardcoded colors**

Kiểm tra bằng command:

```bash
# Tìm tất cả hardcoded Tailwind colors
grep -r "bg-\(red\|blue\|green\|yellow\|indigo\|purple\|pink\|cyan\)-[0-9]" src/

# Expected: Chỉ còn trong các demo components cũ (nếu có)
# LoginPage, RegisterPage, ProfilePage, MainLayout, HomePage PHẢI sạch
```

Thay thế:

- ❌ `bg-blue-500` → ✅ `bg-primary`
- ❌ `text-red-600` → ✅ `text-destructive`
- ❌ `bg-green-100` → ✅ `bg-primary/10` hoặc `bg-muted`
- ❌ `border-gray-300` → ✅ `border`

#### 2. **Upgrade tất cả pages**

**LoginPage:**

- [ ] Dùng `Card`, `CardHeader`, `CardTitle`, `CardDescription`, `CardContent`
- [ ] Labels có icons (từ `lucide-react`)
- [ ] Button có loading state với `Loader2` spinner
- [ ] Background gradient với Tailwind v4 syntax (`bg-linear-to-br`)
- [ ] Toast success/error khi login

**RegisterPage:**

- [ ] Giống LoginPage structure
- [ ] Validation errors hiển thị với `text-destructive`
- [ ] Icons cho từng field (User, Mail, Lock)
- [ ] Toast success khi register thành công
- [ ] Link "Đã có tài khoản?" dẫn về Login

**ProfilePage:**

- [ ] User avatar/icon trong `Card`
- [ ] Badge "Verified" với icon
- [ ] Separator giữa sections
- [ ] User ID hiển thị trong `code` tag với `bg-background`
- [ ] Icons cho email, ID (từ `lucide-react`)

**HomePage:**

- Làm xịn nhất có thể, homepage mà xấu -99d

**Toast:** Hiển thị thông báo

- tham khảo code ở docs (phía trên cũng có chỉ)

### Resources:

- **Demo App:** https://learning-react-delta-opal.vercel.app/
- **shadcn/ui Docs:** https://ui.shadcn.com/
- **Tailwind v4 Docs:** https://tailwindcss.com/docs
- **lucide-react Icons:** https://lucide.dev/

---

## �🚀 NEXT STEPS

**Session tiếp theo: Forms & Validation**

- React Hook Form
- Zod schema validation
- Form error handling
- Controlled inputs advanced patterns

UI hôm nay chưa có validation UX tốt - buổi sau ta sẽ nâng cấp form experience lên professional level! 🎯
