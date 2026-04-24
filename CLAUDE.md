# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> Tiếp tục build/sửa WebCV portfolio của Nguyễn Nhật Trường — AI Automation Engineer đang chuyển ngành từ Video Editor.

---

## 🧱 REPO LAYOUT & CODE ARCHITECTURE

Repo này **không phải Node/Next/Python project** — không có `package.json`, không có build tool, không có test suite. Các lệnh `npm run build / test / lint / dev` từ global CLAUDE.md **không áp dụng** ở đây. Đừng chạy chúng.

### Files cần biết
- [CV Portfolio.html](CV%20Portfolio.html) — **artifact chính**, single-file standalone (~1,600 dòng). Inline CSS trong `<style>` + inline JS trong `<script>`, không có asset ngoài trừ Google Fonts (Space Grotesk / Inter / JetBrains Mono).
- [CV_Vietnamese_Update.md](CV_Vietnamese_Update.md) — changelog Việt hoá text, quy định rõ phần nào dịch / phần nào giữ tiếng Anh. Tham chiếu khi sửa copy.
- [uploads/CV_Design_Brief.md](uploads/CV_Design_Brief.md) — design brief ban đầu, palette/typography/section order. Tham chiếu khi sửa design.
- [CLAUDE.md](CLAUDE.md) — file này. Là **single source of truth** cho toàn bộ data/positioning/design decisions.

### Cấu trúc bên trong `CV Portfolio.html`
```
<head>
  ├─ Google Fonts (Space Grotesk / Inter / JetBrains Mono)
  └─ <style>  — toàn bộ CSS inline, dùng CSS variables (--g, --o, --ink, ...)
              — responsive via @media (max-width: 1024px / 768px)
              — dark mode via body.dark { ... } override CSS vars

<body>
  ├─ <nav class="topbar">               — sticky, có trạng thái + Download CV
  ├─ <div class="page">
  │    ├─ #hero      — Node Canvas (desktop) + .hero-mobile fallback
  │    ├─ #stats     — stats-bar với data-target cho count-up
  │    ├─ #projects  — .proj-grid (4 card 2-col + 1 card full-width)
  │    ├─ #stack     — 4 .stack-row (AI/LLMs · Automation · Dev · Media)
  │    ├─ #learning  — WIP box dashed
  │    ├─ #experience — vertical timeline
  │    ├─ #about     — profile.json style
  │    └─ #contact   — .env style key=value
  ├─ <div class="tweaks">  — panel runtime: theme / lines on-off / accent picker
  └─ <script>
       ├─ DEFAULTS trong `/*EDITMODE-BEGIN*/ { ... } /*EDITMODE-END*/` — edit-mode marker, ĐỪNG đổi tên/xoá comment này
       ├─ draw()         — vẽ SVG bezier dashed giữa các #n-* node trong hero; tính toạ độ bằng getBoundingClientRect → re-run on load + resize
       ├─ line-in animation — stroke-dashoffset keyframe inject runtime
       ├─ IntersectionObserver — count-up cho [data-target]
       ├─ apply(S) / update() / hexAlpha — áp CSS vars từ tweaks panel
       └─ postMessage({type:'__edit_mode_available'}, '*') — handshake với parent window (claude.ai artifact host)
```

### Những điểm quan trọng không thể hiện trên bề mặt
- **Hero canvas = absolute-positioned nodes + SVG overlay.** Các node có id `n-trigger / n-input / n-center / n-out-g / n-out-o / n-links`. Hàm `draw()` đọc bounding box từng node để vẽ bezier giữa chúng. Nếu thêm/xoá/đổi id node, **phải sửa cả danh sách edge trong `draw()`** (các dòng `bez(edge('n-xxx','r'), ...)` và `canvas · 6 nodes · 5 edges` trong `.canvas-bar`).
- **Edit-mode integration (claude.ai artifact host).** Khối `/*EDITMODE-BEGIN*/ { ... } /*EDITMODE-END*/` là contract với host — host parse JSON trong đó và bơm edit. Listener `__activate_edit_mode` / `__deactivate_edit_mode` bật/tắt `.tweaks` panel. Giữ nguyên marker comments và cấu trúc `postMessage` khi refactor.
- **Dark mode bằng CSS vars override**, không phải selector lặp. Muốn thêm màu tuỳ biến dark, thêm override trong `body.dark { --xxx: ... }`.
- **Responsive bẻ ở 1024px và 768px.** Dưới 768px hero canvas bị `display:none`, `.hero-mobile` hiện lên. Nếu đổi breakpoint phải đổi cả hai media query.
- **Không có framework / build step.** Mọi thứ phải viết vanilla, tránh import module hoặc syntax cần transpile. File phải mở trực tiếp bằng `file://` được.
- **Print-friendly là yêu cầu bắt buộc** (để export PDF). Khi thêm section mới, test print preview — đặc biệt animation SVG + count-up có thể vỡ; cân nhắc `@media print { animation: none; }` nếu cần.

## 🛠 WORKFLOW KHI SỬA FILE

- **Preview:** mở trực tiếp `CV Portfolio.html` trong trình duyệt (double-click hoặc `start "" "CV Portfolio.html"` trên Windows). Không cần dev server.
- **Không có lint/test/build command.** Thay vào đó, sau khi sửa:
  1. Reload trình duyệt, kiểm tra 3 breakpoint (DevTools → resize xuống ~1200 / ~900 / ~400).
  2. Toggle dark mode qua tweaks panel.
  3. Verify SVG lines vẫn vẽ đúng giữa các node hero.
  4. Test scroll → count-up ở stats bar chạy đúng.
  5. Test print preview (Ctrl+P) nếu đụng tới layout.
- **Edit chứ không rewrite.** File lớn (~1,600 dòng) nhưng cấu trúc rõ ràng — dùng `Edit` tool với context đủ để unique-match. Không dùng `Write` overwrite toàn file trừ khi user yêu cầu refactor toàn bộ.
- **Khi đổi data/metric:** phải đồng bộ cả 3 chỗ có thể cùng nhắc con số đó — hero nodes, stats bar (`data-target`), và project cards. Ví dụ đổi "2,000+" thì search cả file.
- **Khi thêm/xoá node hero:** sửa (a) HTML node, (b) danh sách edges trong `draw()`, (c) label `canvas · N nodes · M edges`.

---

## 🎯 MỤC TIÊU DỰ ÁN

Build WebCV portfolio (HTML/CSS/JS standalone) để apply vị trí **AI Automation Engineer fulltime** tại các công ty tại TP.HCM.

**Yêu cầu kỹ thuật:**
- Standalone HTML file (có thể mở bằng trình duyệt, không cần build tool)
- Responsive 3 breakpoints: desktop (>1024px), tablet (768-1024px), mobile (<768px)
- Ngôn ngữ: tiếng Việt cho nội dung kể chuyện/mô tả, tiếng Anh cho thuật ngữ kỹ thuật (tên tool, framework, thuật ngữ chuyên ngành)
- Không dùng framework nặng — CSS thuần + vanilla JS là đủ
- Có thể export PDF dễ dàng (print-friendly)

---

## 👤 PROFILE ỨNG VIÊN

### Thông tin cá nhân
- **Họ tên:** Nguyễn Nhật Trường
- **Sinh:** 29/05/2000
- **Địa điểm:** TP. Hồ Chí Minh, Việt Nam
- **Email:** truongnhat2905@gmail.com
- **Facebook:** https://www.facebook.com/nguyentruong2905
- **Phone:** 0379137483
- **LinkedIn:** [chưa có — placeholder]
- **GitHub:** [chưa có — placeholder]
- **Mức lương kỳ vọng:** 18-25 triệu VND (đang test market)

### Học vấn
- **Trường:** ĐH Tài nguyên và Môi trường TP.HCM
- **Ngành:** Công nghệ Thông tin
- **Tốt nghiệp:** 2022
- **Luận văn:** Ứng dụng AR vào bán hàng — đạt 9.0/10, nhận học bổng cuối kỳ

### Hành trình nghề nghiệp
**Narrative chủ đạo:** IT foundation → Video Editor (3+ năm) → AI Video Specialist (2025) → AI Automation Engineer (2026)

Đây là hành trình **chuyển đổi có chủ đích**, không phải nhảy việc — tận dụng nền IT để mang AI vào workflow content production thực tế tại doanh nghiệp.

**Các giai đoạn:**
1. **Đại học (2018-2022):** Học CNTT, hoạt động media ở CLB thiện nguyện, làm AR Filter (Meta Spark AR) freelance
2. **2022 - nay:** Video Editor fulltime tại công ty mỹ phẩm (HCMC) — chủ yếu video quảng cáo ngành beauty
3. **2025:** Chủ động chuyển hướng sang AI Video tại cùng công ty (self-initiated)
4. **2026:** Mở rộng sang AI Automation tại cùng công ty (self-initiated role)
5. **10/2025 - nay:** Chạy affiliate solo trên Shopee + TikTok Shop (ngành gia dụng)

### Thành tựu nổi bật không thuộc automation (chỉ đưa nếu relevant)
- Video triệu view (mảng giáo dục, side project)
- Video BĐS chốt 2 biệt thự Q1 trong 4 tháng (freelance)

### Tính cách & phong cách làm việc
- Ít nói, đa nhiệm, mong cầu công nhận về năng lực
- Điểm mạnh: tự học nhanh, production mindset, bắt kịp thị trường nhanh khi chuyển hướng
- Điểm yếu: thiếu kiến thức nền chuyên sâu ở từng ngành, hay lan man
- **About section (working_style):** "Tự học nhanh · Làm việc độc lập · Làm việc nhóm" (đã sửa từ "Thích mày mò các thứ mới mẻ")

---

## 📊 DATA KỸ THUẬT ĐÃ CONFIRM

Các con số dưới đây là **thật từ ứng viên**, KHÔNG được chế thêm hoặc phóng đại:

### Hệ thống Auto-Post (2024 - nay)
- **Nền tảng automation:** Make.com (2024 - giữa 2025) → n8n (giữa 2025 - nay)
- **Tổng bài đã post:** 2,000+ (ước lượng thấp, thực tế "vài nghìn")
- **Đang quản lý:** 2 Facebook Page + 2 TikTok Channel
- **Đang build thêm:** 3 Facebook Page mới
- **Tần suất:** 3 bài/ngày/platform, chạy 24/7
- **Platforms target:** Facebook, TikTok, TikTok Shop

### AI Video Translation Pipeline (2025 - nay)
- **Quy mô:** 1,000+ video đã dịch production-scale ("vài ngàn video")
- **Throughput:** 80+ video/đêm (auto-run mode)
- **Ngành hàng:** Makeup/mỹ phẩm
- **Tech stack:**
  - Transcribe: Whisper (giai đoạn đầu)
  - Dịch: Gemma 3 (local via Ollama) — hiện tại
  - Output: subtitle + voice-over
- **Đang build:** Kho thuật ngữ ngành makeup để QC tự động
- **Cost:** 100% local LLM → tiết kiệm toàn bộ chi phí API

### Multi-Agent Content System với CrewAI (2025 - nay)
- **Framework:** CrewAI
- **Số agent:** 3 (Research + Script + Review)
- **LLMs sử dụng:**
  - Research Agent: Qwen 3 4B (local) — scrape insight
  - Script Agent: Gemini 2.5 Pro (cloud) — viết hook + body
  - Review Agent: QC tone + policy
- **Use case:** Sản xuất content cho affiliate Shopee + TikTok Shop
- **Model viết content hiện tại:** Gemini 2.5 Pro / Gemini Flash Lite (preview: 3.1-flash-lite-preview)

### Ads Policy Compliance Agent (2026 - đang triển khai)
- **Tech:** Gemini Gem
- **Data source:** Đưa thẳng file policy vào knowledge của Gem (chưa RAG vì data ít)
- **Output:** Duyệt/Không duyệt + chỉ ra chỗ sai + đề xuất sửa
- **Status:** Đang build, sắp deploy production
- **Impact:** Giảm thời gian review script ads thủ công trước khi giao editor

### Knowledge Base Obsidian → Agent (đang học)
- **Status:** Đang tìm hiểu, chưa RAG
- **Tool:** Obsidian làm KB nguồn

### Local LLM experience
- **Đang chạy:** Gemma 3 của Google (local via Ollama)
- **Từng test:** Qwen 3 4B cho viết content
- **Đã thử:** Xây workflow multi-agent bằng CrewAI (mỗi agent 1 LLM riêng)
- **Tool stack:** Ollama, LM Studio

### Affiliate Solo Business (10/2025 - nay)
- **Shopee Affiliate:** 10 triệu VND hoa hồng (cả năm 2025)
- **TikTok Shop:** 4+ triệu VND hoa hồng (từ 01/2026, ~3 tháng)
- **Facebook Reels pages:** 2 page, tổng 120K+ followers
- **TikTok Shop channels:** 2 kênh active
- **Niche chính:** Gia dụng (kệ chén)
- **Đang test:** Ngành rửa xe
- **Số đơn / tỷ lệ chuyển đổi:** Không rõ (dashboard Shopee không cho xem lại)

### Coding & DevOps
- **Viết code:** Không trực tiếp viết — dùng Claude/Gemini để viết theo yêu cầu mình đưa ra (AI-assisted development)
- **Gọi API:** Đã gọi API Claude/OpenAI/Gemini trực tiếp qua chương trình Python local
- **FFmpeg/Remotion:** Có dùng nhưng không tự viết script trực tiếp
- **Deploy:** Chưa deploy chính thức, chỉ dùng Docker + Ngrok để public link tạm thời

---

## 🛠️ TECH STACK CHÍNH THỨC

Chia 4 nhóm (dùng nguyên trong CV):

### 🤖 AI & LLMs
`Claude` · `Gemini` · `LM Studio` · `Gemma 4` · `Qwen 3` · `Ollama`

### ⚙️ Automation & Orchestration
`n8n` · `CrewAI` · `Gemini Gem` · `Make.com` · `Zapier`

### 💻 Development & DevOps (hiện đang ẩn trong HTML — comment out)
`Python` · `Docker` · `Ngrok` · `FFmpeg` · `Remotion` · `REST API`

### 🎬 Media & Creative
`Premiere Pro` · `After Effects` · `CapCut`
> Note: `Meta Spark AR` đang bị comment out trong HTML

### 📚 Đang học (work in progress)
`RAG Pipelines` · `Vector Databases` · `Agentic AI` · `Obsidian` · `Claude Code` · `Cowork`
> Note: `MCP Protocol` và `Agentic Coding` đã được thay bằng `Agentic AI` và `Obsidian` (không có arrow)

**⚠️ KHÔNG ĐƯỢC thêm các tool/framework này** (ứng viên chưa dùng — nếu thêm sẽ là nói láo):
- Veo, Midjourney, DALL-E
- LangChain, LlamaIndex (chưa dùng, chỉ đang học)
- Pinecone, Chroma, Qdrant, Supabase (chưa dùng)
- Các cloud platform: AWS, GCP, Azure (chưa deploy)
- "Master Prompts", "Hyper-seg", "TVC Workflow" (các cụm không rõ nghĩa từ wireframe cũ)

---

## 🎨 DESIGN DIRECTION ĐÃ CHỐT

### Direction: **Node Canvas Hero**
- Hero section = canvas với các node kết nối bằng dashed line (giống workflow UI)
- Bên dưới hero là bento-style grid cho các section còn lại
- **Lý do chọn:** Metaphor workflow/node ăn khớp hoàn hảo với vị trí AI Automation Engineer

### Palette màu
```
Primary (xanh automation):     #10B981
Accent (cam content/business): #F97316
Background:                    #FAFAFA / #FFFFFF
Text primary:                  #1A1A1A
Text secondary:                #666666
Border dashed:                 #CCCCCC / #D0D0D0
```

**Tỷ lệ:** Xanh = chủ đạo (70%, cho automation/tech/success), Cam = accent (30%, cho highlight business/revenue).

### Typography
```
Heading:        Space Grotesk (Sans-serif, 600-700 weight)
Body:           Inter (Sans-serif, 400-500)
Monospace:      JetBrains Mono (cho code/metric/label technical)
```

**Lưu ý:** Tránh handwritten font — làm trông "artistic" hơn "technical", lệch vibe.

### Dark mode
Có support dark mode (đã implement trong version hiện tại của file WebCV).

### Responsive breakpoints
```
Desktop: >1024px   — full node canvas, projects 2-col
Tablet:  768-1024  — simplified canvas, projects 2-col
Mobile:  <768px    — stack vertical, canvas → mobile card, projects 1-col
```

### Interaction/Animation
- Hero SVG dashed lines: animate flow khi page load (stroke-dashoffset)
- Stats bar numbers: count-up animation khi scroll tới
- Project cards: hover nhẹ nhấc lên + shadow
- Badges: static, không animation

---

## 📑 CẤU TRÚC SECTIONS (scroll order)

```
1. TOPBAR (sticky)                 — status + download CV button
2. HERO (Node Canvas)              — impact 3 giây đầu
3. STATS BAR                       — proof points dạng số (count-up)
4. FEATURED PROJECTS               — 4 cards chính + 1 card wide (Affiliate)
5. TECH STACK                      — 4 nhóm công nghệ
6. LEARNING (work in progress)     — dashed box, growth mindset
7. EXPERIENCE TIMELINE             — vertical timeline với node dots
8. ABOUT (profile.json style)      — 3 block với code-comment header
9. CONTACT (.env style)            — key=value format
```

### Section labels (technical metaphor — GIỮ nguyên tiếng Anh phần trigger/input/output)
```
01 · trigger · giới-thiệu
02 · metrics · số-liệu
03 · execute · dự-án
04 · imports · công-nghệ
05 · wip · đang-học
06 · history · kinh-nghiệm
07 · context · giới-thiệu
08 · output · liên-hệ
```

---

## 🌐 QUY TẮC NGÔN NGỮ (QUAN TRỌNG)

### ✅ Phần TIẾNG VIỆT
- Section headings (H2): "Dự án nổi bật", "Kinh nghiệm", "Giới thiệu", "Liên hệ"...
- Mô tả dự án, tagline, bullet points trong timeline
- Labels dễ hiểu: "Bài đăng tự động", "Doanh thu Affiliate", "Người theo dõi"
- CTAs: "Tải CV", "Liên hệ"
- Period: "Hiện tại" thay cho "Present"
- Status: "Sẵn sàng cho cơ hội mới" thay "Open to opportunities"
- Tweaks panel: "Tuỳ chỉnh", "Giao diện", "Sáng/Tối", "Bật/Tắt"

### 🚫 Phần GIỮ NGUYÊN TIẾNG ANH (KHÔNG dịch)

**Lý do:** Dịch ra làm CV trông thiếu chuyên môn và mất vibe technical.

1. **Tên công nghệ/tool:** Claude, Gemini, Ollama, n8n, CrewAI, Whisper, Python, Docker... (tất cả tên riêng)
2. **Thuật ngữ chuyên ngành:** AI, LLM, RAG, MCP, Multi-agent, Pipeline, Workflow, API, Local LLM, DevOps, Production, Automation, Orchestration
3. **Job titles:** `AI Automation Engineer`, `AI Video Specialist`, `Video Editor`, `Affiliate Solo Operator`
4. **Tên dự án (project titles):** Multi-Platform Auto-Post System, AI Video Translation Pipeline, Multi-Agent Content System, Ads Policy Compliance Agent, Affiliate Solo Business
5. **Project tags:** `automation`, `scale`, `local-llm`, `production`, `multi-agent`, `framework`, `llm-app`, `compliance`, `business`, `revenue`
6. **Technical metaphor section labels:** `trigger`, `input`, `output`, `function · main()`, `imports`, `execute`, `wip`, `history`, `context`
7. **Canvas status bar:** `canvas · 6 nodes · 5 edges · workflow v2026`
8. **Contact keys (mô phỏng .env):** `LOCATION`, `EMAIL`, `FACEBOOK`, `LINKEDIN`, `GITHUB`, `STATUS`
9. **Code-style comments trong About:** `// background`, `// transition`, `// working_style`

### Quy tắc vàng
> **Tiếng Việt** cho phần kể chuyện, mô tả, label dễ hiểu với HR Việt Nam.
> **Tiếng Anh** cho phần kỹ thuật, tên riêng, thuật ngữ chuyên ngành, vibe code-style.

---

## 🎯 POSITIONING & TONE

### Positioning cho CV
**"Content production veteran chuyển ngành sang AI Automation — có production-ready experience, không phải beginner demo."**

3 điểm bán (selling points) chính:
1. **Production scale thật:** 2,000+ bài post, 1,000+ video dịch — không phải toy project
2. **Full automation stack:** Từ workflow (n8n) đến multi-agent (CrewAI) đến local LLM (Ollama)
3. **Business outcome mindset:** Có revenue thật (14M affiliate GMV), không chỉ tech-for-tech

### Tone of voice
- Confident nhưng không khoe khoang
- Đưa số liệu, không đưa hình dung
- Ngôn ngữ production (scale, pipeline, throughput, deployment) thay vì ngôn ngữ học thuật (research, experiment)
- Thân thiện với HR Việt Nam: có emoji vừa phải, không quá formal

---

## 📋 FEATURED PROJECTS — DATA CHI TIẾT

### Project 1 — Auto-Post Workflow
- **Period:** 2024 — Hiện tại
- **Tagline:** Hệ thống đăng bài tự động cross-platform
- **Tech:** n8n, Make.com, Zapier, Facebook API, TikTok Node
- **Tags:** automation, scale
- **Metrics:**
  - 2,000+ bài post tự động
  - 2+2 FB Pages + TikTok Channels
  - 3 bài/ngày (label ẩn)
  - 0 thao tác thủ công sau khi setup

### Project 2 — AI Video Translation Pipeline
- **Period:** 2025 — Hiện tại
- **Tagline:** Dịch video sản phẩm cho ngành mỹ phẩm
- **Tech:** Whisper, LM Studio, Gemma 4, FFmpeg, Python
- **Tags:** local-llm, production
- **Stack flow:** Whisper (transcribe) → Gemma 4 local via LM Studio (dịch) → FFmpeg (post-processing)
- **Desc:** Chương trình tự động transcribe + dịch + tạo voice-over cho video ngắn về nội dung makeup tutorial, chạy trên local LLM
- **Metrics:**
  - 1,000+ video đã dịch production-scale
  - 80+ video/đêm throughput auto-run
  - 100% tiết kiệm chi phí API (chạy local)
  - Output 2-in-1: subtitle + voice-over

### Project 3 — Multi-Agent Content System
- **Period:** 2025 — Hiện tại
- **Tagline:** Hệ thống agent phối hợp viết content tự động
- **Tech:** CrewAI, Gemini 2.5 Pro, Qwen 3, Python
- **Tags:** multi-agent, framework
- **Architecture:** Agent Nghiên cứu (Gemini 2.5 Pro) → Agent Kịch bản (Gemini 2.5) → Agent Kiểm duyệt (Qwen 3 local)
- **Metrics:**
  - 3 agents: Nghiên cứu · Viết kịch bản · Kiểm duyệt
  - 2 LLMs: Qwen 3 + Gemini 2.5 Pro

### Project 4 — Ads Policy Compliance Agent
- **Period:** 2026 — Hiện tại
- **Tagline:** AI kiểm tra các kịch bản quảng cáo
- **Tech:** n8n, Gemini/GPT model, Policy Knowledge Base
- **Tags:** Workflow, compliance
- **Output format:** Duyệt/Không duyệt + Liệt kê lỗi + Đề xuất sửa
- **Impact:** Giảm thời gian review kịch bản ads trước khi giao editor

### Project 5 — Cô Giấy System (full-width Flagship card)
- **Period:** 2025 — Hiện tại
- **Tagline:** Quy trình xây dựng nội dung.
- **Tech:** n8n, Google Sheets, Gemini 2.5 Pro, Veo 3, Remotion, Prompt Engineering
- **Tags:** system-design, multi-agent, production
- **Desc:** Thiết kế và xây dựng hệ thống sản xuất video dạy ngôn ngữ cho trẻ em 6-14 tuổi bằng AI.
- **Metrics (4 bước quy trình):**
  - 1 — Tạo nội dung và chia phân cảnh
  - 2 — Tạo prompt image assets / story board / video motion
  - 3 — Tạo image và video
  - 4 — Dùng Remotion ghép thành video final và thêm subtitle

### Project 6 — Affiliate Solo (full-width card)
- **Period:** 10/2025 — Hiện tại
- **Tagline:** (tagline ẩn — bạn đã comment out)
- **Tech:** TikTok Shop, Shopee Affiliate, Self-built AI system
- **Tags:** affiliate, revenue
- **Metrics:**
  - 14M VND tổng hoa hồng (10M Shopee 2025 + 4M TikTok Shop Q1/2026)
  - 120K+ followers 2 Facebook Reels page
  - 4 channels: 2 FB + 2 TikTok Shop
  - Solo operator, 100% automation-driven

---

## 💼 EXPERIENCE TIMELINE — DATA CHI TIẾT

### 1. AI Video/ Automation — 08/2025 — Hiện tại
- HV HOLDINGS GLOBAL - BU Faca · TP.HCM
- Nghiên cứu các AI về video/ hình ảnh
- Sản xuất các video AI về sản phẩm để chạy quảng cáo
- Xây dựng các workflow automation cho team

### 2. Video Editor — 05/2023 — 08/2025
- HV HOLDINGS GLOBAL - BU Faca · TP.HCM
- Sản xuất video chạy quảng cáo cho các sản phẩm mỹ phẩm
- Lên kế hoạch, đi tiền trạm, chuẩn bị thiết bị để quay diễn viên

### 3. Affiliate Solo — 10/2025 — Hiện tại
- Shopee Affiliate + TikTok Shop · Ngành gia dụng
- Xây dựng hệ thống nội dung + tự động hóa đăng bài
- 14M VND GMV · 120K+ followers · 4 channels active

> **Note:** Entry "AI Automation Engineer — 2026 — Hiện tại" tại Công ty Mỹ phẩm hiện đang bị comment out trong HTML (chưa hiển thị). Tên công ty chính xác là HV HOLDINGS GLOBAL.

---

## 🚦 HƯỚNG DẪN CHO CLAUDE CLI

### Khi sửa/build WebCV, cần giữ
- Direction **Node Canvas Hero** (đã chốt)
- 4 nhóm tech stack như đã liệt kê
- 5 projects (4 cards đều + 1 wide full-width cho Affiliate)
- Quy tắc ngôn ngữ: Việt cho kể chuyện, Anh cho thuật ngữ kỹ thuật
- Palette Xanh #10B981 + Cam #F97316 (70/30)
- Responsive 3 breakpoints
- Support dark mode

### Khi cần bổ sung thông tin mới
- Ưu tiên hỏi lại ứng viên, **KHÔNG chế số liệu**
- Data trong file này đã được confirm từ ứng viên → nguồn sự thật duy nhất
- Nếu cần thêm metric mà chưa có, để placeholder `[bổ sung sau]` thay vì bịa

### Khi HR/nhà tuyển dụng feedback đòi chỉnh
- Giữ core data, chỉ điều chỉnh framing
- Có thể bổ sung thêm project mới nếu ứng viên build thêm — nhưng cần giữ cấu trúc 4+1

### Tránh các lỗi đã gặp trước đây
- ❌ Đừng để "Mobile Dev" (luận văn là AR, không phải Mobile)
- ❌ Đừng dùng cụm "Self-host · secure · scale" (không đúng profile)
- ❌ Đừng thêm Veo, Midjourney, Master Prompts vào tech stack
- ❌ Đừng dùng handwritten font cho project titles
- ❌ Đừng đưa "Alphabet Builders" (placeholder cũ từ wireframe, không phải dự án thật)

---

## 📝 TODO / OPEN ITEMS

Các mục sau đang **chờ ứng viên bổ sung**, CV hiện đang dùng placeholder:
- [ ] LinkedIn URL
- [ ] GitHub URL (nếu ứng viên lập)
- [ ] CV PDF file để download
- [ ] Chốt ngày "Available from"
- [ ] Cân nhắc bổ sung portfolio cụ thể: link 1-2 video demo hoặc case study chi tiết

---

**END OF CONTEXT**

> File này là **single source of truth** cho dự án WebCV của Nguyễn Nhật Trường. Mọi quyết định về content, design, data đều tham chiếu về đây. Nếu có mâu thuẫn với file/instruction khác, ưu tiên file này.
