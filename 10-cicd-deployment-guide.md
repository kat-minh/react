# BÀI 10: FE CI/CD & VERCEL DEPLOYMENT

## 🎯 Mục tiêu học tập
Nâng cấp từ việc deploy thủ công sang xây dựng một hệ thống **CI/CD chuyên nghiệp**:
- **CI (Continuous Integration)**: Xây dựng "Bốt kiểm soát chất lượng" (Quality Gate) tự động trên GitHub.
- **CD (Continuous Deployment)**: Tự động hóa quy trình Release lên Vercel.
- **Fail-safe Logic**: Xử lý thông báo lỗi (Discord) và tư duy Rollback.
- **Graduation**: Chốt khóa với live URL và minh chứng năng lực trên Portfolio/CV.

---

## 🏗️ I. TRIẾT LÝ CI/CD: TẠI SAO LOCAL PASS LÀ CHƯA ĐỦ?
CI/CD giúp chúng ta thoát khỏi hội chứng "Works on my machine":
- **CI (Kiểm tra)**: Tự động chạy ngay khi Push/PR. Nếu Lint đỏ hoặc Build vỡ -> Cấm Merge vào `main`.
- **CD (Triển khai)**: Tự động đưa code "sạch" lên Vercel ngay sau khi qua cổng kiểm dịch.

---

## 🛠️ II. GITHUB ACTIONS: XÂY DỰNG QUALITY GATE
Tạo file `.github/workflows/ci.yml` tại thư mục gốc của dự án:

```yaml
name: CI
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  frontend-ci:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup pnpm
        uses: pnpm/action-setup@v4
        with: { version: 9 }
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'pnpm' }
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      - name: Lint & Build
        run: |
          pnpm run lint
          pnpm run build
```

### 🛡️ Branch Protection (Kỷ luật thép)
Để CI thực sự có ý nghĩa, bạn cần bật **Branch Protection** trên GitHub:
1. Vào Repo -> Settings -> Branches -> Add branch protection rule.
2. Branch name pattern: `main`.
3. Tích chọn: **Require a pull request before merging** và **Require status checks to pass before merging**.
4. Chọn check `frontend-ci`.
=> Kết quả: Từ nay không ai có thể "Push ẩu" vào main nếu CI chưa báo xanh.

---

## 🤖 III. TUTORIAL: TÍCH HỢP DISCORD NOTIFICATION (BONUS)
Mục tiêu: Khi pipeline bị FAIL, hệ thống sẽ tự động nhắn tin thông báo vào Discord của team để xử lý ngay.

**Bước 1: Lấy Webhook URL**
- Vào Discord Server -> Server Settings -> Integrations -> Webhooks -> New Webhook -> Copy Webhook URL.

**Bước 2: Cài đặt GitHub Secret**
- Vào GitHub Repo -> Settings -> Secrets and variables -> Actions -> New repository secret.
- Name: `DISCORD_WEBHOOK_URL` | Value: Dán link vừa copy.

**Bước 3: Tạo Workflow thông báo**
- Tạo file `.github/workflows/notify-fail.yml`:
```yaml
name: Notify Pipeline Failure
on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]

jobs:
  notify-discord:
    if: ${{ github.event.workflow_run.conclusion == 'failure' }}
    runs-on: ubuntu-latest
    steps:
      - name: Send Discord notification
        uses: Ilshidur/action-discord@master
        env:
          DISCORD_WEBHOOK: ${{ secrets.DISCORD_WEBHOOK_URL }}
        with:
          args: |
            ❌ CI workflow failed!
            Repo: ${{ github.repository }}
            Branch: ${{ github.event.workflow_run.head_branch }}
            Triggered by: ${{ github.event.workflow_run.actor.login }}
            Logs: ${{ github.event.workflow_run.html_url }}
```

**🔥 Mẹo Test thử:** 
Để kiểm tra bot có chạy không, hãy cố tình tạo ra một lỗi nhỏ (VD: thêm một biến thừa hoặc `console.log` để bị Lint bắt lỗi), sau đó Push lên. Bạn sẽ thấy bot bắn tin nhắn vào Discord ngay lập tức!

---

## 🚀 IV. VERCEL AUTO CI/CD & SPA ROUTING

### 1. Vercel Git Integration
Vercel tự động hóa phần lớn quy trình CD:
- **Preview Deployments**: Mỗi Pull Request (PR) sẽ có một link "xem trước" riêng để review UI.
- **Env Vars**: Nhập các Secret (VD: `VITE_API_URL`) vào mục **Project Settings -> Environment Variables** trên Vercel.

### 2. Fix lỗi 404 (SPA Routing)
Tạo file `vercel.json` để xử lý việc reload trang tại route sâu:
```json
{
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```

