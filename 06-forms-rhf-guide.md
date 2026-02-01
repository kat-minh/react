# REACT HOOK FORM & ZOD - STUDENT GUIDE

## 🎯 Mục tiêu học tập

Xây dựng form xử lý đăng ký (Register) sử dụng React Hook Form và Zod để:

- Hiểu được sự khác biệt giữa Controlled và Uncontrolled forms
- Quản lý form hiệu quả với React Hook Form (zero re-renders khi typing)
- Validate dữ liệu với Zod schema
- Xử lý custom validation (password match, date format)
- Kết nối form với backend API

---

## 🧠 Khái niệm cốt lõi

### Controlled vs Uncontrolled Components

#### 🔴 Controlled (React Way - Traditional)

React quản lý value của input qua state:

```tsx
const [email, setEmail] = useState("");

<input
  value={email} // ← React kiểm soát giá trị
  onChange={(e) => setEmail(e.target.value)} // ← Mỗi keystroke → setState
/>;
```

**Flow:**

1. User gõ "a" → onChange trigger
2. `setEmail("a")` → Component re-render
3. Input nhận `value="a"` mới
4. Lặp lại với mỗi ký tự

**Vấn đề:**

- ❌ Re-render toàn bộ component MỖI KEYSTROKE
- ❌ Performance tệ với form lớn (10+ fields)
- ❌ Boilerplate code: `useState` + `onChange` cho TỪNG field

#### 🟢 Uncontrolled (DOM Way - Performance)

DOM tự quản lý value, React chỉ "đọc" khi cần:

```tsx
const emailRef = useRef<HTMLInputElement>(null);

<input ref={emailRef} />; // ← Không có value, không có onChange

// Đọc giá trị khi submit
const handleSubmit = () => {
  console.log(emailRef.current?.value);
};
```

**Flow:**

1. User gõ "a" → DOM tự update (không qua React)
2. Component KHÔNG re-render
3. Khi submit → Đọc value từ ref

**Ưu điểm:**

- ✅ Zero re-renders khi typing
- ✅ Performance tốt hơn nhiều
- ✅ Ít boilerplate code

**Nhược điểm:**

- ⚠️ Không real-time validation (phải đợi submit)
- ⚠️ Khó quản lý complex logic
- ⚠️ Phải tự viết validation logic
- ⚠️ Không có error state management

### ⚠️ Tại sao KHÔNG dùng useRef cho forms?

Tưởng tượng form 10 fields với useRef:

```tsx
// ❌ BAD APPROACH
function RegisterFormWithRef() {
  const emailRef = useRef<HTMLInputElement>(null);
  const passwordRef = useRef<HTMLInputElement>(null);
  const confirmPasswordRef = useRef<HTMLInputElement>(null);
  // ... 7 refs nữa

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();

    const email = emailRef.current?.value || "";
    const password = passwordRef.current?.value || "";

    // 😱 Manual validation hell
    if (!email.includes("@")) {
      alert("Email invalid");
      return;
    }

    if (password.length < 6) {
      alert("Password too short");
      return;
    }

    // Lặp lại cho 10 fields... 🤦‍♂️
  };
}
```

**Vấn đề:**

1. ❌ Quá nhiều boilerplate: 10 refs, 10 dòng đọc value
2. ❌ Manual validation: Tự viết logic if/else cho TỪNG field
3. ❌ Không có error state: Làm sao hiển thị lỗi dưới mỗi input?
4. ❌ Không có touched state: Làm sao biết user đã tương tác với field?
5. ❌ Khó maintain: Thêm 1 field mới = thêm 5-10 dòng code
6. ❌ Không type-safe: TypeScript không giúp được gì

### ✨ React Hook Form - Best of Both Worlds

RHF dùng **Uncontrolled** nhưng cung cấp API giống **Controlled**:

```tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const schema = z.object({
  email: z.string().email("Email không hợp lệ"),
  password: z.string().min(6, "Password tối thiểu 6 ký tự"),
});

type FormData = z.infer<typeof schema>;

function RegisterFormWithRHF() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = (data: FormData) => {
    // Data đã được validate 100% ✅
    console.log(data); // Type-safe ✅
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("email")} />
      {errors.email && <p>{errors.email.message}</p>}

      <input {...register("password")} type="password" />
      {errors.password && <p>{errors.password.message}</p>}
    </form>
  );
}
```

**So sánh:**

| Feature                  | useRef         | React Hook Form        |
| ------------------------ | -------------- | ---------------------- |
| Boilerplate code         | ❌ Nhiều       | ✅ Ít                  |
| Validation               | ❌ Manual      | ✅ Auto (Zod)          |
| Error handling           | ❌ Tự code     | ✅ Built-in            |
| TypeScript support       | ❌ Weak        | ✅ Strong              |
| Performance              | ✅ Tốt         | ✅ Tốt (cũng dùng ref) |
| Developer Experience     | ❌ Tệ          | ✅ Tuyệt vời           |
| Scalability (10+ fields) | ❌ Không scale | ✅ Scale tốt           |

**KẾT LUẬN:**

- **useRef phù hợp cho:** Focus input, scroll to element, lưu timer IDs
- **React Hook Form phù hợp cho:** BẤT KỲ form nào từ 2 fields trở lên
- **Quy tắc vàng:** _"Nếu bạn đang nghĩ dùng useRef cho form → DÙNG React Hook Form"_

---

## 📋 ZOD CHEAT SHEET

### 1️⃣ String Validations

```ts
import { z } from "zod";

// Basic string
z.string(); // Bất kỳ string nào

// Required (không được empty string)
z.string().min(1, "Không được để trống");
// ⚠️ LƯU Ý: z.string() CHẤP NHẬN "" (empty string)

// Email validation
z.string().email("Email không hợp lệ");

// Min/Max length
z.string().min(6, "Tối thiểu 6 ký tự");
z.string().max(100, "Tối đa 100 ký tự");
z.string().min(3).max(20); // Kết hợp

// Regex pattern
z.string().regex(/^[A-Z]/, "Phải bắt đầu bằng chữ hoa");
z.string().regex(/^\d{10}$/, "Phone phải 10 số");

// URL validation
z.string().url("URL không hợp lệ");

// Trim whitespace
z.string().trim(); // Tự động xóa khoảng trắng đầu/cuối

// Transform
z.string().toLowerCase(); // Chuyển về chữ thường
z.string().toUpperCase(); // Chuyển về chữ hoa

// Optional
z.string().optional(); // Cho phép undefined
z.string().nullable(); // Cho phép null
```

**Ví dụ thực tế:**

```ts
const userSchema = z.object({
  username: z
    .string()
    .min(3, "Username tối thiểu 3 ký tự")
    .max(20, "Username tối đa 20 ký tự")
    .regex(/^[a-zA-Z0-9_]+$/, "Chỉ chứa chữ, số, gạch dưới"),

  email: z.string().email("Email không hợp lệ").toLowerCase().trim(),

  website: z.string().url("URL không hợp lệ").optional(),
});
```

### 2️⃣ Number Validations

```ts
// Basic number
z.number();

// Integer (số nguyên)
z.number().int("Phải là số nguyên");

// Positive/Negative
z.number().positive("Phải là số dương");
z.number().negative("Phải là số âm");
z.number().nonnegative("Phải >= 0");

// Min/Max value
z.number().min(0, "Tối thiểu 0");
z.number().max(100, "Tối đa 100");
z.number().min(18).max(65); // Độ tuổi hợp lệ

// Multiple of (chia hết cho)
z.number().multipleOf(5, "Phải chia hết cho 5");

// Finite (không phải Infinity)
z.number().finite("Không được là Infinity");
```

**Ví dụ thực tế:**

```ts
const productSchema = z.object({
  price: z.number().positive("Giá phải > 0").max(1000000),

  quantity: z
    .number()
    .int("Số lượng phải là số nguyên")
    .nonnegative("Số lượng >= 0"),

  discount: z.number().min(0).max(100).optional(),
});
```

### 3️⃣ Date Validations

```ts
// Basic date
z.date(); // JavaScript Date object

// Min/Max date
z.date().min(new Date("2020-01-01"), "Ngày phải sau 2020");
z.date().max(new Date(), "Không được chọn ngày tương lai");

// Coerce from string (HTML input type="date" trả về string)
z.coerce.date(); // "2024-01-30" → Date object
```

**Ví dụ thực tế:**

```ts
const eventSchema = z
  .object({
    startDate: z.coerce.date(),
    endDate: z.coerce.date(),
  })
  .refine((data) => data.endDate > data.startDate, {
    message: "Ngày kết thúc phải sau ngày bắt đầu",
    path: ["endDate"],
  });
```

### 4️⃣ Custom Validations

```ts
// refine (custom logic - single field)
z.string().refine((val) => val.includes("@"), { message: "Phải chứa ký tự @" });

// transform (chuyển đổi dữ liệu)
z.string().transform((val) => val.toUpperCase());
// "hello" → "HELLO"

// superRefine (custom logic - multiple fields)
z.object({
  password: z.string(),
  confirm: z.string(),
}).superRefine((data, ctx) => {
  if (data.password !== data.confirm) {
    ctx.addIssue({
      code: "custom",
      message: "Mật khẩu không khớp",
      path: ["confirm"],
    });
  }
});
```

### 5️⃣ Type Inference

```ts
const userSchema = z.object({
  name: z.string(),
  age: z.number(),
  email: z.string().email(),
});

// Lấy type từ schema
type User = z.infer<typeof userSchema>;
// Type: { name: string; age: number; email: string }
```

### 📌 Quick Reference Table

| Method         | Dùng cho                    | Ví dụ                                  |
| -------------- | --------------------------- | -------------------------------------- |
| `.min()`       | String length, Number value | `z.string().min(3)`                    |
| `.max()`       | String length, Number value | `z.number().max(100)`                  |
| `.email()`     | Email validation            | `z.string().email()`                   |
| `.regex()`     | Custom pattern              | `z.string().regex(/^\d+$/)`            |
| `.optional()`  | Optional fields             | `z.string().optional()`                |
| `.default()`   | Default value               | `z.string().default("N/A")`            |
| `.refine()`    | Custom validation           | `z.string().refine(v => v.length > 0)` |
| `.transform()` | Transform data              | `z.string().transform(v => v.trim())`  |
| `z.coerce`     | Type coercion               | `z.coerce.number()`                    |

---

## 🛠️ IMPLEMENTATION GUIDE

### PHASE 1: SETUP & INSTALLATION

#### Bước 1: Cài đặt packages

```bash
npm install react-hook-form zod @hookform/resolvers
```

**Giải thích:**

- `react-hook-form`: Quản lý form state và submission
- `zod`: Schema validation library
- `@hookform/resolvers`: Adapter để kết nối Zod với React Hook Form

#### Bước 2: Tạo Validation Schema

Tạo file `src/utils/rules.ts`:

```ts
import * as z from "zod";

export const registerSchema = z
  .object({
    email: z
      .string()
      .min(1, { message: "Email không được để trống" })
      .email({ message: "Email không hợp lệ" }),

    name: z.string().min(1, { message: "Tên không được để trống" }),

    password: z.string().min(6, { message: "Password phải ít nhất 6 ký tự" }),

    confirm_password: z.string(),

    date_of_birth: z.string(), // HTML input type="date" trả về string
  })
  .superRefine(({ password, confirm_password }, ctx) => {
    // Custom validation cho password match
    if (confirm_password !== password) {
      ctx.addIssue({
        code: "custom",
        message: "Nhập lại mật khẩu không khớp",
        path: ["confirm_password"], // Gán lỗi vào field này
      });
    }
  });

// Export type để dùng cho RHF
export type RegisterSchemaType = z.infer<typeof registerSchema>;
```

**⚠️ LƯU Ý QUAN TRỌNG:**

1. **Luôn tách schema ra file riêng** (`rules.ts`) để:
   - Dễ tái sử dụng cho nhiều components
   - Dễ test
   - Clean code

2. **superRefine vs refine:**
   - `refine`: Validate cả object, nhưng lỗi gán cho object
   - `superRefine`: Validate cả object, nhưng có thể gán lỗi cho field cụ thể qua `path`

---

### PHASE 2: IMPLEMENT REGISTER FORM

Tạo/Update file `src/pages/RegisterPage.tsx`:

```tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { registerSchema, RegisterSchemaType } from "@/utils/rules";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";

export default function RegisterPage() {
  // 🔍 Setup useForm với Zod resolver
  const {
    register, // Function để đăng ký input với RHF
    handleSubmit, // Wrapper cho submit handler
    formState: {
      errors, // Object chứa validation errors
      isSubmitting, // Boolean - đang submit hay không
    },
  } = useForm<RegisterSchemaType>({
    mode: "onSubmit", // Validate khi submit
    resolver: zodResolver(registerSchema),
    defaultValues: {
      email: "",
      name: "",
      password: "",
      confirm_password: "",
      date_of_birth: "",
    },
  });

  // 🔍 onSubmit chỉ chạy KHI validation PASS
  const onSubmit = (data: RegisterSchemaType) => {
    console.log("✅ Data đã validate:", data);
    // TODO: Gọi API ở đây
  };

  return (
    <form
      onSubmit={handleSubmit(onSubmit)}
      className="space-y-4 max-w-md mx-auto mt-10 p-6 border rounded-lg"
    >
      <h2 className="text-2xl font-bold">Đăng ký</h2>

      {/* EMAIL FIELD */}
      <div className="space-y-2">
        <Label htmlFor="email">Email</Label>
        <Input
          id="email"
          type="email"
          placeholder="email@example.com"
          {...register("email")}
        />
        {errors.email && (
          <p className="text-red-500 text-xs">{errors.email.message}</p>
        )}
      </div>

      {/* NAME FIELD */}
      <div className="space-y-2">
        <Label htmlFor="name">Họ tên</Label>
        <Input id="name" placeholder="Nguyễn Văn A" {...register("name")} />
        {errors.name && (
          <p className="text-red-500 text-xs">{errors.name.message}</p>
        )}
      </div>

      {/* PASSWORD FIELD */}
      <div className="space-y-2">
        <Label htmlFor="password">Mật khẩu</Label>
        <Input
          id="password"
          type="password"
          placeholder="••••••••"
          {...register("password")}
        />
        {errors.password && (
          <p className="text-red-500 text-xs">{errors.password.message}</p>
        )}
      </div>

      {/* CONFIRM PASSWORD FIELD */}
      <div className="space-y-2">
        <Label htmlFor="confirm_password">Nhập lại mật khẩu</Label>
        <Input
          id="confirm_password"
          type="password"
          placeholder="••••••••"
          {...register("confirm_password")}
        />
        {errors.confirm_password && (
          <p className="text-red-500 text-xs">
            {errors.confirm_password.message}
          </p>
        )}
      </div>

      {/* DATE OF BIRTH */}
      <div className="space-y-2">
        <Label htmlFor="date_of_birth">Ngày sinh</Label>
        <Input id="date_of_birth" type="date" {...register("date_of_birth")} />
        {errors.date_of_birth && (
          <p className="text-red-500 text-xs">{errors.date_of_birth.message}</p>
        )}
      </div>

      {/* SUBMIT BUTTON */}
      <Button type="submit" className="w-full" disabled={isSubmitting}>
        {isSubmitting ? "Đang xử lý..." : "Đăng ký"}
      </Button>
    </form>
  );
}
```

---

### PHASE 3: HIỂU SÂU register() FUNCTION

Khi bạn viết `{...register("email")}`, RHF thực chất trả về object:

```tsx
// register("email") returns:
{
  name: "email",           // Field name để RHF tracking
  ref: (el) => {...},      // Callback ref để RHF "móc" vào DOM
  onChange: (e) => {...},  // Track changes nhưng KHÔNG trigger re-render
  onBlur: (e) => {...}     // Dùng cho validation mode "onBlur"
}
```

**Magic của RHF:**

- Dùng **ref** internally → Uncontrolled → Zero re-renders khi typing
- Cung cấp API giống Controlled → Developer experience tốt
- Chỉ re-render khi:
  1. Có errors (validation fail)
  2. Bắt đầu submit (`isSubmitting = true`)
  3. Submit xong (`isSubmitting = false`)

---

### PHASE 4: FLOW KHI SUBMIT

```
User bấm Submit Button
    ↓
handleSubmit() được gọi
    ↓
RHF đọc tất cả values từ DOM (qua refs)
    ↓
Pass values → Zod Schema validate
    ↓
    ├─ Validation FAIL?
    │   ↓
    │  Update errors object
    │  Component re-render (hiển thị errors)
    │  STOP (không gọi onSubmit)
    │
    └─ Validation PASS?
        ↓
       Call onSubmit(cleanData)
       Set isSubmitting = true
       Component re-render (button disabled)
           ↓
          onSubmit chạy xong
          Set isSubmitting = false
          Component re-render (button enabled)
```

---

### PHASE 5: CONNECT TO API

Update `onSubmit` function:

```tsx
import { usersApi } from "@/lib/api/users.api";
import { toast } from "sonner";
import { useNavigate } from "react-router-dom";

export default function RegisterPage() {
  const navigate = useNavigate();

  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<RegisterSchemaType>({
    resolver: zodResolver(registerSchema),
  });

  const onSubmit = async (data: RegisterSchemaType) => {
    try {
      // Data transformation nếu cần
      const apiBody = {
        ...data,
        date_of_birth: new Date(data.date_of_birth).toISOString(),
      };

      const res = await usersApi.register(apiBody);

      toast.success("Đăng ký thành công!");
      navigate("/login");
    } catch (error: any) {
      // Xử lý lỗi 422 từ backend
      if (error.response?.status === 422) {
        toast.error("Lỗi dữ liệu đầu vào");
        // TODO: Map backend errors to form fields (Advanced)
      } else {
        toast.error("Lỗi hệ thống");
      }
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>{/* ... form fields ... */}</form>
  );
}
```

---

## 🎓 VALIDATION MODES

React Hook Form hỗ trợ nhiều validation modes:

```tsx
useForm({
  mode: "onSubmit", // ← Validate khi submit (recommended)
  // mode: "onChange", // Validate realtime khi gõ
  // mode: "onBlur",   // Validate khi rời khỏi field
  // mode: "onTouched", // Validate sau khi đã touch field
});
```

**So sánh:**

| Mode        | Khi nào validate    | Re-renders | UX       | Khi nào dùng           |
| ----------- | ------------------- | ---------- | -------- | ---------------------- |
| `onSubmit`  | Khi bấm Submit      | Ít nhất    | Tốt      | Forms đơn giản         |
| `onChange`  | Mỗi keystroke       | Nhiều nhất | Tốt nhất | Forms phức tạp         |
| `onBlur`    | Khi rời field       | Trung bình | Tốt      | Balance performance/UX |
| `onTouched` | Sau khi touch field | Trung bình | OK       | Ít dùng                |

**Khuyến nghị:**

- Bắt đầu với `onSubmit`
- Chuyển sang `onBlur` khi cần UX tốt hơn
- Chỉ dùng `onChange` khi thực sự cần real-time validation

---

## ⚠️ COMMON MISTAKES & DEBUGGING

### 1️⃣ Quên `type="submit"` cho Button

**Symptom:** Bấm nút không submit, hoặc trang reload

**Fix:**

```tsx
// ❌ WRONG
<button onClick={handleSubmit(onSubmit)}>Submit</button>

// ✅ CORRECT
<button type="submit">Submit</button>
```

### 2️⃣ Quên `{...register}`

**Symptom:** Gõ dữ liệu nhưng submit nhận `undefined`

**Fix:**

```tsx
// ❌ WRONG
<input name="email" />

// ✅ CORRECT
<input {...register("email")} />
```

### 3️⃣ Lỗi import Resolver

**Symptom:** Validation không chạy

**Fix:**

```bash
# Cài đúng package
npm install @hookform/resolvers
```

```tsx
// Import đúng
import { zodResolver } from "@hookform/resolvers/zod";
```

### 4️⃣ Quên truyền Resolver

**Symptom:** Form submit dù data rỗng/sai

**Fix:**

```tsx
// ❌ WRONG
useForm<FormData>();

// ✅ CORRECT
useForm<FormData>({
  resolver: zodResolver(schema),
});
```

### 5️⃣ Schema không match với field names

**Symptom:** Validation không chạy cho field cụ thể

**Fix:**

```tsx
// Schema field names phải khớp với register() names
const schema = z.object({
  email: z.string().email(), // ← "email"
});

<input {...register("email")} /> {/* ← "email" */}
```

---

## 🚀 ADVANCED TOPICS

### 1️⃣ Async Validation (Check email tồn tại)

```ts
const schema = z.object({
  email: z
    .string()
    .email()
    .refine(
      async (email) => {
        // Call API check email exist
        const res = await usersApi.checkEmailExists(email);
        return !res.data.exists;
      },
      { message: "Email đã tồn tại" },
    ),
});
```

⚠️ **Cẩn thận:** Async validation có thể làm chậm form. Nên dùng debounce hoặc validate `onBlur`.

### 2️⃣ Dynamic Fields (Array of inputs)

```tsx
import { useFieldArray } from "react-hook-form";

const { fields, append, remove } = useFieldArray({
  control,
  name: "items",
});

// Thêm/xóa fields động
```

### 3️⃣ Backend Error Mapping

Khi backend trả về lỗi 422:

```tsx
import { useForm } from "react-hook-form";

const { setError } = useForm();

const onSubmit = async (data) => {
  try {
    await api.register(data);
  } catch (error) {
    if (error.response?.status === 422) {
      const backendErrors = error.response.data.errors;

      // Map backend errors to form fields
      Object.keys(backendErrors).forEach((field) => {
        setError(field as any, {
          type: "manual",
          message: backendErrors[field][0],
        });
      });
    }
  }
};
```

---

## 📝 PRACTICE EXERCISES

### Exercise 1: Basic Form

Tạo Login Form với:

- Email field (validate email format)
- Password field (min 6 chars)
- Remember me checkbox
- Submit button

### Exercise 2: Password Strength

Thêm validation cho password phải có:

- Ít nhất 1 chữ hoa
- Ít nhất 1 số
- Ít nhất 1 ký tự đặc biệt

```ts
z.string()
  .min(8, "Tối thiểu 8 ký tự")
  .refine((val) => /[A-Z]/.test(val), {
    message: "Phải có ít nhất 1 chữ hoa",
  })
  .refine((val) => /[0-9]/.test(val), {
    message: "Phải có ít nhất 1 số",
  })
  .refine((val) => /[!@#$%^&*]/.test(val), {
    message: "Phải có ít nhất 1 ký tự đặc biệt",
  });
```

### Exercise 3: Date Validation

Validate `date_of_birth`:

- Phải >= 18 tuổi
- Không được chọn ngày tương lai

```ts
z.coerce
  .date()
  .max(new Date(), "Không được chọn ngày tương lai")
  .refine(
    (date) => {
      const age = new Date().getFullYear() - date.getFullYear();
      return age >= 18;
    },
    { message: "Phải đủ 18 tuổi" },
  );
```

---

## 🔗 USEFUL RESOURCES

- [React Hook Form Docs](https://react-hook-form.com/)
- [Zod Documentation](https://zod.dev/)
- [shadcn/ui Form Examples](https://ui.shadcn.com/docs/components/form)

---

## ✅ CHECKLIST

Sau buổi học này, bạn cần nắm được:

- [ ] Hiểu sự khác biệt giữa Controlled và Uncontrolled forms
- [ ] Hiểu tại sao không nên dùng useRef cho forms
- [ ] Setup React Hook Form với Zod resolver
- [ ] Tạo validation schema với Zod
- [ ] Implement form với register(), handleSubmit(), errors
- [ ] Hiểu validation modes (onSubmit, onChange, onBlur)
- [ ] Custom validation với superRefine
- [ ] Kết nối form với API
- [ ] Debug common errors

---

## 💡 TIPS & BEST PRACTICES

1. **Luôn tách schema ra file riêng** (`rules.ts`)
2. **Dùng `mode: "onSubmit"`** khi bắt đầu, chuyển sang `onBlur` khi cần UX tốt hơn
3. **Validate ở client để improve UX**, nhưng LUÔN validate lại ở server
4. **Dùng TypeScript** để type-safe: `z.infer<typeof schema>`
5. **Error messages phải rõ ràng** cho user: "Email không hợp lệ" thay vì "Invalid"
6. **Test form validation** trước khi connect API
7. **Handle loading state** với `isSubmitting` để prevent double submit
8. **Transform data trước khi gửi API** nếu cần (date format, trim, etc.)
