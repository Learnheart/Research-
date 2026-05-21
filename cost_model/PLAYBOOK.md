# PLAYBOOK — Cost Model Research

[CẬP NHẬT: 21/05/2026]

> **Mục đích:** Tập hợp các lệnh + workflow vận hành Cost Model. Mỗi workflow self-contained, có thể chạy độc lập, có ví dụ PowerShell cụ thể (môi trường Windows). Mọi quy ước tham chiếu về [[CLAUDE]].

---

## ⚡ Navigation

| Bạn cần làm | Đi tới |
|---|---|
| Cập nhật ngày update ở 1 file | §1 |
| Thêm 1 dòng vào rate card | §2 |
| Refresh rate card hàng tháng (top-5 provider) | §3 |
| Audit compliance với `CLAUDE.md` | §4 |
| Tạo file `.md` mới đúng template | §5 |
| Khởi tạo + dùng git | §6 |
| Deploy methodology sang Databricks workspace | §7 |

---

## §1. Cập nhật ngày update ở file

Khi sửa nội dung file `.md`, **bắt buộc** cập nhật dòng `[CẬP NHẬT: DD/MM/YYYY]` ở đầu file (quy tắc [[CLAUDE]] §2.3).

### PowerShell snippet

```powershell
# Thay $file bằng đường dẫn thật
$file  = 'C:\Projects\Databricks\cost_model\01_rate_card.md'
$today = (Get-Date -Format 'dd/MM/yyyy')
(Get-Content $file -Raw -Encoding UTF8) `
    -replace '\[CẬP NHẬT: \d{2}/\d{2}/\d{4}\]', "[CẬP NHẬT: $today]" `
    | Set-Content $file -Encoding UTF8 -NoNewline:$false
```

### Bulk update (toàn bộ file `.md` trong `cost_model/`)

```powershell
$today = (Get-Date -Format 'dd/MM/yyyy')
Get-ChildItem -Recurse 'C:\Projects\Databricks\cost_model\*.md' | ForEach-Object {
    (Get-Content $_.FullName -Raw -Encoding UTF8) `
        -replace '\[CẬP NHẬT: \d{2}/\d{2}/\d{4}\]', "[CẬP NHẬT: $today]" `
        | Set-Content $_.FullName -Encoding UTF8
}
```

> **Cẩn thận:** Bulk update chỉ dùng khi **toàn bộ** file đã được sửa cùng ngày. Bình thường update từng file riêng.

### Citations

- Microsoft Docs. *about_Regular_Expressions (PowerShell).* learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_regular_expressions (truy cập 21/05/2026)

---

## §2. Quy trình thêm 1 dòng vào Rate Card

### Bước thực hiện

| Bước | Hành động |
|---|---|
| 1 | Mở `01_rate_card.md` section tương ứng: §1 text / §2 image / §3 audio / §4 video-embed-finetune |
| 2 | Thêm 1 dòng đúng format bảng của section đó (xem template bên dưới) |
| 3 | Đảm bảo cột `Verify-by` có URL chính thức của provider |
| 4 | Cập nhật `[CẬP NHẬT]` ở đầu file (xem §1) |
| 5 | Update `PROGRESS.md` Phase 3 hoặc Backlog tương ứng |
| 6 | Commit (nếu đã enable git): `git commit -m "rate-card: add <provider>/<model>"` |

### Template hàng cho text section

```markdown
| <Provider> | <Model> | <input_$/1M> | <output_$/1M> | <cache_read_$/1M> | <Notes> | <verify_URL> |
```

### Template hàng cho image-output section

```markdown
| <Provider> | <Model> | <Resolution/Quality> | <$/image> | <verify_URL> |
```

### Citations

- Anthropic. *Pricing.* anthropic.com/pricing (truy cập 21/05/2026)
- OpenAI. *API Pricing.* openai.com/api/pricing (truy cập 21/05/2026)

---

## §3. Quy trình monthly rate card refresh

### Cadence

| Khi nào | Hành động |
|---|---|
| Ngày đầu mỗi tháng | Chạy refresh checklist bên dưới |
| Khi vendor công bố đổi giá | Update trong vòng 14 ngày ([[03_forecasting]] §5) |

### Checklist refresh

| Bước | Hành động | Tool |
|---|---|---|
| 1 | Scrape pricing page top-5 provider | curl / browser; tương lai dùng WebFetch tool |
| 2 | Diff với rate card hiện tại | `git diff 01_rate_card.md` |
| 3 | Nếu delta ≥ 5% cho dòng nào → flag trong commit message với prefix `BREAKING:` | Manual review |
| 4 | Update date stamp đầu file (xem §1) | PowerShell snippet §1 |
| 5 | Trigger forecast re-baseline | Theo [[03_forecasting]] §5 |
| 6 | Update `PROGRESS.md` Phase 3 task tương ứng | Manual edit |

### Top-5 provider pricing pages

| Provider | URL | Refresh frequency |
|---|---|---|
| Anthropic | https://anthropic.com/pricing | Hàng tháng |
| OpenAI | https://openai.com/api/pricing | Hàng tháng |
| Google | https://ai.google.dev/pricing | Hàng tháng |
| AWS Bedrock | https://aws.amazon.com/bedrock/pricing/ | Hàng tháng |
| Azure OpenAI | https://azure.microsoft.com/.../pricing/details/cognitive-services/openai-service | Hàng tháng |

### Citations

- FinOps Foundation. *Effect of Optimization on AI Forecasting.* finops.org/wg/effect-of-optimization-on-ai-forecasting/ (truy cập 21/05/2026)

---

## §4. Audit compliance với `CLAUDE.md`

### Manual checklist (mỗi file)

```
[ ] Heading H1 ở dòng 1
[ ] Dòng `[CẬP NHẬT: DD/MM/YYYY]` ngay sau H1 (CLAUDE §2.3)
[ ] Tiếng Việt có dấu, không pha không dấu (CLAUDE §1.1)
[ ] Mỗi section có số liệu / claim kỹ thuật → có `### Citations` ở cuối section (CLAUDE §3.1)
[ ] Citation đúng format: `- <Tên>. *<Title>.* <URL> (truy cập DD/MM/YYYY)` (CLAUDE §3.2)
[ ] Link `[[name]]` trỏ tới file tồn tại (CLAUDE §4.4)
[ ] Bảng > bullet list ở chỗ phù hợp (CLAUDE §4.3)
[ ] Heading dạng `## §N. <Tên>` (CLAUDE §4.5)
```

### Auto-check (PowerShell + grep)

#### Tìm file thiếu date stamp

```powershell
Get-ChildItem -Recurse 'C:\Projects\Databricks\cost_model\*.md' | ForEach-Object {
    $content = Get-Content $_.FullName -Raw -Encoding UTF8
    if ($content -notmatch '\[CẬP NHẬT: \d{2}/\d{2}/\d{4}\]') {
        Write-Host "MISSING date stamp: $($_.FullName)" -ForegroundColor Red
    }
}
```

#### Tìm link `[[name]]` broken

```powershell
$baseNames = (Get-ChildItem -Recurse 'C:\Projects\Databricks\cost_model\*.md').BaseName
$pattern   = '\[\[([^\]]+)\]\]'
Get-ChildItem -Recurse 'C:\Projects\Databricks\cost_model\*.md' | ForEach-Object {
    $matches = [regex]::Matches((Get-Content $_.FullName -Raw -Encoding UTF8), $pattern)
    foreach ($m in $matches) {
        $target = $m.Groups[1].Value -split '/' | Select-Object -Last 1
        if ($baseNames -notcontains $target) {
            Write-Host "BROKEN link [[$($m.Groups[1].Value)]] in $($_.Name)" -ForegroundColor Yellow
        }
    }
}
```

#### Tìm section có số mà không có Citations

```powershell
# Heuristic: section có "%" hoặc "$" hoặc "×" hoặc "M tokens" mà không kết thúc bằng "Citations"
# (cần kiểm tra thêm bằng mắt — đây chỉ là gợi ý)
Get-ChildItem -Recurse 'C:\Projects\Databricks\cost_model\*.md' | ForEach-Object {
    $content = Get-Content $_.FullName -Raw -Encoding UTF8
    # Tách theo H2
    $sections = $content -split '(?m)^## '
    for ($i = 1; $i -lt $sections.Count; $i++) {
        $sec = $sections[$i]
        $hasNumber = $sec -match '(%|\$\d|×\d|M tokens|\d+\s*USD)'
        $hasCitation = $sec -match '### Citations'
        if ($hasNumber -and -not $hasCitation) {
            $title = ($sec -split "`n")[0]
            Write-Host "MISSING citation in '$($_.Name)' §$title" -ForegroundColor Yellow
        }
    }
}
```

### Citations

- Microsoft Docs. *PowerShell -match operator.* learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_comparison_operators (truy cập 21/05/2026)

---

## §5. Template tạo file `.md` mới

Khi tạo file mới (top-level hoặc trong `_appendix/`), dùng template sau:

```markdown
# <Title>

[CẬP NHẬT: DD/MM/YYYY]

> **Insight cốt lõi:** <Một câu kết luận theo Pyramid principle — đặt trước mọi nội dung khác>

---

## ⚡ 30-sec card

<Tuỳ file:>
- Sơ đồ ASCII (cho file conceptual như taxonomy)
- Công thức (cho file methodology)
- Bảng navigation (cho file index)

| Bạn cần | Đi tới |
|---|---|
| ... | §1 |
| ... | §2 |

---

## §1. <Section name>

<Nội dung. Ưu tiên bảng so sánh.>

### Citations

- <Tên tổ chức>. *Title.* URL (truy cập DD/MM/YYYY)

---

## §2. <Section name>

<Nội dung>

### Citations

- <Tên>. *Title.* URL (truy cập DD/MM/YYYY)

---

## Liên kết

- <Related concept>: [[name]]
- <Related concept>: [[name]]
```

### Citations

- (không có claim số liệu cần citation cho section này)

---

## §6. Git workflow

Hiện tại `C:\Projects\Databricks` chưa phải git repo (theo environment). Khi enable git cho riêng `cost_model/`:

### Init

```powershell
Set-Location 'C:\Projects\Databricks\cost_model'
git init
git branch -M main

# Tạo .gitignore
@'
# Backup & temp
*.bak
*.tmp
*~

# OS
.DS_Store
Thumbs.db

# Editor
.vscode/
.idea/

# Future data artifacts (nếu sinh ra)
*.parquet
*.duckdb
'@ | Set-Content '.gitignore' -Encoding UTF8

git add .
git commit -m "cost-model: initial baseline v1 (Phase 0-1 complete)"
```

### Commit message convention (Conventional Commits)

```
<scope>: <short description>

scope ∈ {
  rate-card    → 01_rate_card.md
  formula      → 02_cost_formula.md
  forecast     → 03_forecasting.md
  monitoring   → 04_monitoring.md
  appendix     → _appendix/*
  rules        → CLAUDE.md
  progress     → PROGRESS.md
  playbook     → PLAYBOOK.md
  readme       → README.md
}
```

### Ví dụ commit messages

```
rate-card: add Gemini 2.5 Flash-Lite pricing
rate-card: BREAKING — Anthropic Opus giảm 30% từ 2026-06-01
formula: add modality term cho video output
forecast: calibrate retry_multiplier với data pilot UC-RM-001
rules: enforce citations cuối section (CLAUDE §3.1)
```

### Branching strategy

| Branch | Use case |
|---|---|
| `main` | Stable; chỉ merge sau review |
| `feat/<topic>` | Feature mới (vd. `feat/audio-modality`) |
| `fix/<topic>` | Sửa lỗi (vd. `fix/elevenlabs-price`) |
| `chore/<topic>` | Housekeeping (vd. `chore/audit-q2`) |

### Citations

- Conventional Commits. *Conventional Commits 1.0.0.* conventionalcommits.org (truy cập 21/05/2026)
- Vincent Driessen. *A successful Git branching model.* nvie.com/posts/a-successful-git-branching-model/ (truy cập 21/05/2026)

---

## §7. Triển khai methodology sang Databricks workspace (Phase 4+)

Khi sẵn sàng deploy methodology lên Databricks production:

| Bước | Mô tả | Output |
|---|---|---|
| 1 | Convert `01_rate_card.md` → `rate_card.yaml` (machine-readable) | `data/rate_card.yaml` |
| 2 | Build allocation notebook đọc `system.serving.endpoint_usage` + join với `rate_card.yaml` | Databricks notebook |
| 3 | Schedule daily job → bảng `gold.cost_per_invocation` | Delta table |
| 4 | Build Databricks SQL dashboard layered theo 6 lớp ([[02_cost_formula]] §1) | Dashboard URL |
| 5 | Cấu hình Lakehouse Monitoring trên metric ở [[04_monitoring]] §3.1 | Monitor config |
| 6 | Setup AI Gateway service policies cho budget + rate limit ([[04_monitoring]] §2) | Policy YAML |

### Citations

- Databricks Docs. *Monitor usage for Unity AI Gateway endpoints.* docs.databricks.com/aws/en/ai-gateway/usage-tracking-beta (truy cập 21/05/2026)
- Databricks Docs. *Introduction to Databricks Lakehouse Monitoring.* docs.databricks.com/aws/en/lakehouse-monitoring/ (truy cập 21/05/2026)
- FinOps Foundation. *How to Build a Generative AI Cost and Usage Tracker.* finops.org/wg/how-to-build-a-generative-ai-cost-and-usage-tracker/ (truy cập 21/05/2026)

---

## §8. Cập nhật `PLAYBOOK.md` này

Khi thêm workflow mới:

| Bước | Hành động |
|---|---|
| 1 | Thêm section `## §N. <Tên workflow>` |
| 2 | Update bảng `⚡ Navigation` ở đầu file |
| 3 | Thêm citations ở cuối section nếu có số liệu / claim kỹ thuật |
| 4 | Update date stamp đầu file (§1) |
| 5 | Update `PROGRESS.md` Phase 2 nếu workflow này là core, hoặc Backlog nếu nice-to-have |

### Citations

- (không có claim cần citation)

---

## Liên kết

- Quy tắc tác giả: [[CLAUDE]]
- Checklist phase + task: [[PROGRESS]]
- Master index: [[README]]
- File methodology dùng workflow này: [[01_rate_card]], [[02_cost_formula]], [[03_forecasting]], [[04_monitoring]]
