# Phân tích bố cục trang chủ tlu.edu.vn

> Phân tích theo tư duy: **Nhìn giao diện → khoanh hộp → đặt tên → xác định cha/con → chọn Flex/Grid/Block/Position → nhận diện nội dung → chọn HTML semantic.**

**Phạm vi.** Bài phân tích dựa trên nội dung và thứ tự phần tử của trang, chưa dựa trên giao diện đã render. Hai loại thông tin được đánh dấu:

- **[Quan sát]**: thấy trực tiếp trong nội dung trang (tiêu đề, liên kết, ảnh, thứ tự các khối).
- **[Suy luận]**: đoán từ cấu trúc, cần kiểm tra bằng DevTools (F12 → Elements) để xác nhận.

Website dùng Next.js (ảnh có đường dẫn `/_next/image`), nên class thật sẽ khác tên đặt ở đây. Tên dưới đây là tên theo vai trò để học.

---

## 1. Bản đồ bố cục toàn trang (chỉ dùng div)

```text
div.page
├── div.header
│   └── div.header-container
│       ├── div.brand                       (logo)
│       ├── div.nav
│       │   └── div.nav-container
│       │       └── div.nav-item  ×8
│       │           └── div.submenu
│       │               └── div.submenu-item
│       └── div.header-tools
│           ├── div.search
│           └── div.lang-switch
│
├── div.main
│   ├── div.hero
│   │   └── div.hero-slides → div.hero-slide ×(ảnh + 2 video)
│   │
│   ├── div.news-section
│   │   ├── div.news-carousel   → div.carousel-track → div.news-card ×7
│   │   ├── div.notice-list     → div.notice-item ×6
│   │   └── div.event-carousel  → div.carousel-track → div.event-card ×8
│   │
│   ├── div.stats
│   │   ├── div.section-title
│   │   └── div.stats-list → div.stat-item ×5 (stat-number + stat-label)
│   │
│   ├── div.feature-grid
│   │   ├── div.leader-photo
│   │   ├── div.teaser.admission   (Tuyển sinh)
│   │   ├── div.teaser             (Đào tạo)
│   │   ├── div.teaser             (Sinh viên)
│   │   └── div.teaser             (Cựu sinh viên)
│   │
│   ├── div.units
│   │   ├── div.section-title
│   │   ├── div.unit-list   → div.unit-item ×11
│   │   └── div.unit-detail
│   │
│   ├── div.topic-section  (Hợp tác quốc tế)
│   ├── div.topic-section  (Nghiên cứu)
│   ├── div.partners
│   ├── div.topic-section  (Vì cộng đồng)
│   └── div.gallery → div.gallery-item ×14
│
└── div.footer
    ├── div.footer-main
    │   ├── div.footer-brand  (logo + địa chỉ/email/điện thoại + social)
    │   ├── div.footer-column ×3
    │   └── div.footer-map
    └── div.footer-bottom (copyright)
```

**Lưu ý [Quan sát]:** các mục "Hợp tác quốc tế", "Nghiên cứu", "Vì cộng đồng" có cùng cấu trúc (3 bài có ảnh + 3 bài chỉ có chữ). Đó là **một component dùng lại 3 lần** (`topic-section`), không phải 3 component khác nhau.

---

## 2. Phân tích từng component

Mỗi component gồm 3 lớp: **Layout box → CSS → Nội dung + HTML**. Cây div ở mục 1 chính là "phiên bản chỉ dùng div". Phần *Semantic* viết lại bằng thẻ đúng nghĩa.

### 2.1. Header + Navigation

**Layout [Quan sát + Suy luận]**

```text
div.header
└── div.header-container
    ├── div.brand
    ├── div.nav → div.nav-container → div.nav-item ×8 → div.submenu
    └── div.header-tools → div.search + div.lang-switch
```

Thứ tự trong mã nguồn là logo → tìm kiếm → ngôn ngữ → menu. Phía sau còn một **bản menu thứ hai** có thêm mục "Tất cả …" và tiêu đề "Trường Đại học Thủy lợi". Rất có khả năng đây là **menu trượt (drawer) cho mobile**, cần kiểm tra bằng cách thu nhỏ cửa sổ trình duyệt.

**CSS**

```css
.header-container { display: flex; align-items: center; justify-content: space-between; gap: 24px; }
.nav-container    { display: flex; gap: 8px; }          /* 8 mục xếp ngang */
.header-tools     { display: flex; align-items: center; gap: 12px; }
.lang-switch      { display: flex; gap: 8px; }

.nav-item  { position: relative; }                      /* làm mốc cho submenu */
.submenu   { position: absolute; top: 100%; left: 0; display: none; }
.nav-item:hover .submenu { display: block; }
```

- **Flex** ở 3 chỗ: `header-container`, `nav-container`, `header-tools`. Đây là flex lồng nhau.
- **Position** cho `submenu`: nổi lên trên nội dung bên dưới, không đẩy trang xuống.
- **Không cần Flex** cho bên trong `submenu`: các link con xếp dọc theo Normal Flow.

| Nội dung | Vai trò | HTML |
|---|---|---|
| Logo | thương hiệu, bấm về trang chủ | `a` bọc `img` |
| 8 mục menu | điều hướng chính | `nav > ul > li > a` |
| Submenu | điều hướng cấp 2 | `ul > li > a` lồng trong `li` cha |
| Nút "Tìm kiếm" | mở ô tìm kiếm (hành động) | `button` |
| VN / EN + cờ | đổi ngôn ngữ | `button` nếu chỉ đổi trạng thái, `a` nếu chuyển URL |

```html
<header class="header">
  <div class="header-container">
    <a class="brand" href="/"><img src="logo.webp" alt="Trường Đại học Thủy lợi"></a>
    <nav class="nav" aria-label="Menu chính">
      <ul class="nav-container">
        <li class="nav-item">
          <a href="/gioi-thieu">GIỚI THIỆU</a>
          <ul class="submenu">
            <li><a href="/co-cau-to-chuc">Cơ cấu tổ chức</a></li>
          </ul>
        </li>
      </ul>
    </nav>
    <div class="header-tools">
      <button type="button">Tìm kiếm</button>
      <div class="lang-switch">
        <button type="button">VN</button><button type="button">EN</button>
      </div>
    </div>
  </div>
</header>
```

**Kết luận:** Flex lồng 3 tầng + Position cho submenu; HTML là `header > nav > ul > li > a`. Mục "eTLU" không có link riêng (chỉ có submenu), nên nên làm `button` mở submenu thay vì `a`.

---

### 2.2. Hero (banner)

**Layout [Suy luận]:** `div.hero > div.hero-slides > div.hero-slide`. Nội dung có 5 ảnh banner và 2 video `.mp4`, nên có khả năng là slider/carousel, cần kiểm tra tương tác để xác nhận.

**CSS (hai cách phổ biến)**

```css
.hero        { position: relative; overflow: hidden; }
/* Cách A: trượt ngang */
.hero-slides { display: flex; }
.hero-slide  { flex: 0 0 100%; }      /* mỗi slide chiếm đúng 100% bề rộng */
/* Cách B: chồng lớp, đổi bằng fade */
.hero-slide  { position: absolute; inset: 0; }
```

`flex: 0 0 100%` nghĩa là không co, không giãn, rộng 100%. Nhiều slide nối nhau thành một dải dài và JS dịch dải đó bằng `transform`.

| Nội dung | HTML |
|---|---|
| Ảnh banner | `img` (bọc `a` nếu bấm được) |
| Video | `video autoplay muted loop playsinline` |
| Tiêu đề trang chủ | `h1` (ẩn bằng CSS, vì nội dung hiện tại không có `h1` rõ ràng) |

```html
<section class="hero" aria-label="Banner nổi bật">
  <h1 class="visually-hidden">Trường Đại học Thủy lợi</h1>
  <div class="hero-slides">
    <div class="hero-slide"><img src="20thang11.webp" alt="Kỷ niệm 20/11"></div>
    <div class="hero-slide"><video src="1.mp4" autoplay muted loop playsinline></video></div>
  </div>
</section>
```

---

### 2.3. News card (Tin tức)

Đây là **component quan trọng nhất** vì lặp lại nhiều lần trên trang.

**Layout [Quan sát]:** mỗi card gồm ảnh (là link) → nhãn chuyên mục ("Tin tức chung", "Hoạt động chung") → tiêu đề là link → mô tả (chỉ card đầu có) → ngày. Card đầu có thêm nhãn **"Hot"**.

```text
div.news-carousel
└── div.carousel-track
    └── div.news-card
        ├── div.news-card-image
        └── div.news-card-body
            ├── div.news-card-category
            ├── div.news-card-title
            ├── div.news-card-excerpt
            └── div.news-card-date
```

Cặp nút `‹ ›` cho thấy đây là carousel. Danh sách được lặp 2 lần trong nội dung, có thể là bản sao để chạy vòng (loop) hoặc hai bản cho desktop/mobile. Cần kiểm tra.

**CSS**

```css
.carousel        { position: relative; overflow: hidden; }
.carousel-track  { display: flex; gap: 20px; }            /* KHÔNG wrap: để trượt ngang */
.news-card       { flex: 0 0 calc(33.333% - 14px);        /* 3 card/hàng trên desktop */
                   display: flex; flex-direction: column; position: relative; }
.news-card-body  { flex: 1; }                              /* thân card chiếm phần còn lại, các card cao đều nhau */
.news-badge      { position: absolute; top: 8px; left: 8px; }  /* nhãn "Hot" nổi trên ảnh */
.carousel-arrow  { position: absolute; top: 50%; }
```

Flex xuất hiện ở **hai tầng**: `carousel-track` (ngang) và `news-card` (dọc). Hướng `row` hay `column` quyết định trục chính. Đây là điểm sinh viên hay nhầm.

| Nội dung | Vai trò | HTML |
|---|---|---|
| Ảnh | minh họa bài | `a > img` |
| Nhãn chuyên mục | phân loại | `span` |
| Nhãn "Hot" | huy hiệu | `span` |
| Tiêu đề | tên bài | `h3 > a` |
| Mô tả | tóm tắt | `p` |
| Ngày | thời gian | `time datetime="2026-10-19"` |
| Mũi tên ‹ › | điều khiển | `button` |

```html
<article class="news-card">
  <a class="news-card-image" href="/bai-viet"><img src="..." alt=""></a>
  <div class="news-card-body">
    <span class="news-card-category">Tin tức chung</span>
    <h3><a href="/bai-viet">Thông báo tuyển dụng giảng viên</a></h3>
    <p>Căn cứ nhu cầu giảng dạy…</p>
    <time datetime="2026-10-19">19/10/2026</time>
  </div>
</article>
```

Lưu ý: ảnh và tiêu đề trỏ cùng một URL nên người dùng đọc màn hình sẽ nghe trùng link. Cách xử lý là cho ảnh `alt=""` như trên, hoặc bọc cả card trong một link.

**Kết luận:** `Layout: image + body → CSS: Flex column (card) trong Flex row (track) → HTML: article > a/img + span + h3/a + p + time`.

---

### 2.4. Thông báo (Notice list)

**Layout [Quan sát]:** 6 dòng, mỗi dòng gồm tiêu đề (link) rồi ngày ở cuối.

```text
div.notice-list
└── div.notice-item
    ├── div.notice-title
    └── div.notice-date
```

```css
.notice-item  { display: flex; justify-content: space-between; align-items: baseline; gap: 16px; }
.notice-title { flex: 1; }          /* tiêu đề dài chiếm hết chỗ */
.notice-date  { flex-shrink: 0; }   /* ngày không bị bóp */
```

Nếu ngày nằm dưới tiêu đề thì **không cần Flex**, vì hai phần tử tự xếp dọc.

| Nội dung | HTML |
|---|---|
| Tiêu đề khu "Thông Báo" | `h2 > a` |
| Danh sách | `ul > li` |
| Tiêu đề thông báo | `a` |
| Ngày | `time` |

```html
<section class="notice-list">
  <h2><a href="/thong-bao">Thông Báo</a></h2>
  <ul>
    <li class="notice-item">
      <a href="/thong-bao-thu-hoc-phi">Thông báo thu học phí học kỳ 1…</a>
      <time datetime="2026-11-11">11/11/2026</time>
    </li>
  </ul>
</section>
```

---

### 2.5. Sự kiện (Event carousel)

**Layout [Quan sát]:** mỗi card có ảnh, **ô ngày/tháng** ("19" + "Tháng 11") và tiêu đề. Danh sách lặp lại nhiều lần, nhiều khả năng do carousel chạy vòng hoặc marquee.

```text
div.event-card
├── div.event-image
├── div.event-date → div.event-day + div.event-month
└── div.event-title
```

```css
.event-card { position: relative; }
.event-date { position: absolute; top: 0; left: 0;           /* ô ngày đè lên góc ảnh */
              display: flex; flex-direction: column; align-items: center; }
```

| Nội dung | HTML |
|---|---|
| Ô ngày | `time` chứa 2 `span` (ngày, tháng) |
| Ảnh | `img` |
| Tiêu đề | `h3 > a` |

```html
<article class="event-card">
  <a href="/su-kien/...">
    <img src="..." alt="">
    <time class="event-date" datetime="2026-11-19">
      <span class="event-day">19</span><span class="event-month">Tháng 11</span>
    </time>
  </a>
  <h3><a href="/su-kien/...">Trực tiếp Lễ kỷ niệm Ngày Nhà giáo Việt Nam 20/11</a></h3>
</article>
```

---

### 2.6. Những con số ấn tượng (Statistics)

**Layout [Quan sát]:** `h2` + 5 nhãn: *Người học, Đối tác trong & ngoài nước, Sinh viên tốt nghiệp có việc làm, Ngành học, Giảng viên Tiến sĩ*. Phần số không có trong nội dung lấy được, nên rất có khả năng là **bộ đếm chạy bằng JS**, cần kiểm tra.

```text
div.stats
├── div.section-title
└── div.stats-list
    └── div.stat-item
        ├── div.stat-number
        └── div.stat-label
```

**Flex hay Grid?** Cả hai đều dùng được nhưng **Grid hợp hơn**: 5 mục cần cột đều nhau, và khi thu hẹp Flex wrap sẽ ra kiểu 3+2 lệch.

```css
.stats-list { display: grid; grid-template-columns: repeat(auto-fit, minmax(160px, 1fr)); gap: 24px; }
.stat-item  { display: flex; flex-direction: column; align-items: center; }  /* số trên, nhãn dưới */
```

Đây là chỗ Grid (cả hàng) và Flex (trong từng ô) **phối hợp**.

| Nội dung | HTML |
|---|---|
| Tiêu đề khu | `h2` |
| Cặp nhãn–số liệu | `dl > div > dt + dd` |

```html
<section class="stats">
  <h2>Những Con Số Ấn Tượng</h2>
  <dl class="stats-list">
    <div class="stat-item"><dt>Ngành học</dt><dd>…</dd></div>
  </dl>
</section>
```

Nếu số hiện trên nhãn thì dùng `flex-direction: column-reverse` để vẫn giữ `dt` trước `dd` trong mã nguồn.

---

### 2.7. Feature grid (Tuyển sinh, Đào tạo, Sinh viên, Cựu sinh viên)

**Layout [Quan sát]:** 1 ảnh lãnh đạo (alt "Trang Chủ", cần kiểm tra xem là lời chào hay trích dẫn) + 4 khối giới thiệu. Khối **Tuyển sinh** đặc biệt: ảnh, tiêu đề, mô tả, danh sách 3 bậc học (Đại học/Thạc sĩ/Tiến sĩ) và nút "Đăng ký ngay". Ba khối còn lại là một bài nổi bật: ảnh + tiêu đề + trích đoạn.

**CSS [Suy luận: dạng "bento" lệch kích thước]**

```css
.feature-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 24px; }
.teaser.admission { grid-row: span 2; }        /* khối Tuyển sinh cao gấp đôi */
.teaser           { display: flex; flex-direction: column; gap: 12px; }
```

Khi ô có kích thước khác nhau (span 2 hàng) thì **Grid là lựa chọn đúng**, Flex rất khó làm kiểu này.

| Nội dung | HTML |
|---|---|
| Tên khối ("Tuyển Sinh", "Đào Tạo"…) | `h2 > a` |
| Ảnh | `img` |
| Tiêu đề bài | `h3 > a` |
| Trích đoạn | `p` |
| Danh sách bậc học | `ul > li` (là `a` nếu bấm được, cần kiểm tra) |
| "Đăng ký ngay" | `a` (chuyển sang `ts.tlu.edu.vn`, nên là link dạng nút) |

```html
<section class="feature-grid">
  <article class="teaser admission">
    <h2><a href="https://ts.tlu.edu.vn/">Tuyển Sinh</a></h2>
    <img src="..." alt="">
    <p>Cung cấp thông tin tuyển sinh…</p>
    <ul><li>Đại học</li><li>Thạc sĩ</li><li>Tiến sĩ</li></ul>
    <a class="btn" href="https://ts.tlu.edu.vn/">Đăng ký ngay</a>
  </article>
</section>
```

---

### 2.8. Các đơn vị đào tạo, KHCN (master–detail)

**Layout [Quan sát + Suy luận]:** `h2` + 11 liên kết đơn vị (đều trỏ sang subdomain riêng) + một khối mô tả cho **một** đơn vị (Khoa Kinh tế và Quản lý) kèm link "Khám phá…". Có khả năng khi rê chuột hoặc chọn đơn vị thì khối mô tả đổi theo, cần kiểm tra.

```text
div.units
├── div.unit-list → div.unit-item ×11
└── div.unit-detail → (mô tả + link "Khám phá")
```

```css
.units       { display: flex; gap: 32px; }
.unit-list   { flex: 1; }     /* tỷ lệ 1 : 2 */
.unit-detail { flex: 2; }
```

Tỷ lệ **1 : 2** nghĩa là chia chiều ngang thành 3 phần, danh sách chiếm 1, chi tiết chiếm 2.

| Nội dung | HTML |
|---|---|
| Tiêu đề | `h2` |
| Danh sách đơn vị | `ul > li > a` |
| Mô tả | `p` |
| "Khám phá…" | `a` |

---

### 2.9. Topic section (Hợp tác quốc tế, Nghiên cứu, Vì cộng đồng)

**Layout [Quan sát]:** tiêu đề khu (là link) + **3 bài có ảnh** (chuyên mục, tiêu đề, ngày) + **3 bài chỉ có chữ**.

```text
div.topic-section
├── div.section-title
└── div.topic-body
    ├── div.topic-featured → div.news-card ×3
    └── div.topic-list     → div.topic-item ×3
```

```css
.topic-body     { display: flex; gap: 24px; }          /* [Suy luận] hai nhóm cạnh nhau */
.topic-featured { flex: 2; display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 16px; }
.topic-list     { flex: 1; }                            /* chữ xếp dọc: Normal Flow */
```

Bố cục thật (1 bài lớn + 2 bài nhỏ, hay 3 bài ngang) cần xác nhận trên giao diện. Quan trọng là tách đúng: `news-card` dùng lại component ở mục 2.3, còn `topic-item` là phiên bản rút gọn (không có ảnh).

```html
<section class="topic-section">
  <h2><a href="/doi-ngoai">Hợp Tác Quốc Tế</a></h2>
  <div class="topic-body">
    <div class="topic-featured"><!-- 3 × article.news-card --></div>
    <ul class="topic-list">
      <li><span class="category">Hoạt động chung</span>
          <a href="...">Trường Đại học Thủy lợi dự Lễ kỷ niệm…</a>
          <time datetime="2026-11-06">06/11/2026</time></li>
    </ul>
  </div>
</section>
```

---

### 2.10. Mạng lưới đối tác

**[Quan sát]:** `h2` + một đoạn giới thiệu. **Logo đối tác không xuất hiện** trong nội dung lấy được, có thể được nạp bằng JS hoặc là băng chuyền chạy ngang, cần kiểm tra.

```css
.partner-logos { display: flex; flex-wrap: wrap; justify-content: center; align-items: center; gap: 24px; }
```

Tiêu đề và đoạn văn nằm trong Normal Flow. Chỉ riêng hàng logo mới cần Flex.

```html
<section class="partners">
  <h2>Mạng Lưới Đối Tác</h2>
  <p>Trường Đại học Thủy lợi tự hào là đối tác của…</p>
  <ul class="partner-logos"><li><img src="..." alt="Tên đối tác"></li></ul>
</section>
```

---

### 2.11. Khoảnh khắc TLU (Gallery)

**[Quan sát]:** `h2` + 14 ảnh, không có link riêng trong nội dung, chỉ có `alt`.

```css
.gallery { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; }
```

Dạng ảnh kích thước lệch nhau thì dùng `grid` hoặc `column-count` (masonry), còn Flex không hợp vì cần cả hàng lẫn cột.

```html
<section class="gallery">
  <h2>Khoảnh Khắc TLU</h2>
  <ul class="gallery-list"><li><img src="..." alt="Lễ kết nạp Đảng viên…"></li></ul>
</section>
```

Nếu chú thích hiện ra thật trên giao diện thì dùng `figure > img + figcaption`. Nếu chỉ là `alt` thì không cần `figcaption`.

---

### 2.12. Footer

**Layout [Quan sát]:**

```text
div.footer
├── div.footer-main
│   ├── div.footer-brand   → logo, địa chỉ, email, điện thoại, 4 icon mạng xã hội
│   ├── div.footer-column  → "Giới thiệu"  (4 link)
│   ├── div.footer-column  → "Truy cập nhanh" (5 link)
│   ├── div.footer-column  → "Theo dõi TLU" (4 link)
│   └── div.footer-map     → "Trường Đại học Thủy lợi" + "Tải bản đồ tương tác"
└── div.footer-bottom      → bản quyền + đơn vị phát triển
```

```css
.footer-main   { display: flex; flex-wrap: wrap; gap: 32px; }
.footer-brand  { flex: 2; }               /* cột thương hiệu rộng gấp đôi */
.footer-column { flex: 1; }
.footer-bottom { display: flex; justify-content: space-between; flex-wrap: wrap; }
```

Có thể dùng `grid-template-columns: 2fr 1fr 1fr 1fr` thay cho Flex ratio, vì đều là cột cố định. Với footer thì hai cách như nhau.

| Nội dung | HTML |
|---|---|
| Icon mạng xã hội (chỉ có biểu tượng) | `a` + `aria-label` (nội dung hiện chỉ lộ URL, không có chữ) |
| Địa chỉ/Email/Điện thoại | `address` chứa `dl`; `a href="tel:…"`, `a href="mailto:…"` |
| Tiêu đề cột | `h3` |
| Danh sách link | `ul > li > a` |
| "Tải bản đồ tương tác" | `button` (nhiều khả năng bấm để nạp bản đồ chứ không chuyển trang) |
| Bản quyền | `p` |

```html
<footer class="footer">
  <div class="footer-main">
    <div class="footer-brand">
      <img src="logo.webp" alt="Trường Đại học Thủy lợi">
      <address>
        <dl>
          <dt>Địa chỉ:</dt><dd>175 Tây Sơn - P. Kim Liên - Hà Nội</dd>
          <dt>Điện thoại:</dt><dd><a href="tel:02438522201">(024) 38522201</a></dd>
        </dl>
      </address>
    </div>
    <nav class="footer-column" aria-label="Giới thiệu">
      <h3>Giới thiệu</h3>
      <ul><li><a href="/gioi-thieu">Tổng quan</a></li></ul>
    </nav>
  </div>
  <p class="footer-bottom">Bản quyền © 2026 Trường Đại học Thủy lợi…</p>
</footer>
```

---

## 3. Bản đồ Flexbox toàn trang

```text
PAGE
├── HEADER
│   ├── header-container     [FLEX row, space-between]
│   ├── nav-container        [FLEX row]
│   │   └── submenu          [POSITION absolute, bên trong: block]
│   └── header-tools         [FLEX row] → lang-switch [FLEX]
├── HERO
│   └── hero-slides          [FLEX nowrap, slide flex:0 0 100%]  (hoặc position)
├── MAIN
│   ├── news-carousel        [FLEX nowrap] → news-card [FLEX column]
│   ├── notice-item          [FLEX row, title flex:1]
│   ├── event-carousel       [FLEX nowrap] → event-date [FLEX column]
│   ├── stat-item            [FLEX column]   (cả hàng: GRID)
│   ├── teaser               [FLEX column]   (cả khối: GRID)
│   ├── units                [FLEX 1:2]
│   ├── topic-body           [FLEX 2:1]      → topic-featured [GRID]
│   └── partner-logos        [FLEX + WRAP]
└── FOOTER
    ├── footer-main          [FLEX + WRAP, 2:1:1:1]
    └── footer-bottom        [FLEX space-between]
```

**Nơi không cần Flex:** `submenu` (link xếp dọc), `topic-list` (dòng chữ xếp dọc), từng `news-card-body` (nhãn/tiêu đề/mô tả/ngày tự xếp dọc), và toàn bộ `main` (các section xếp dọc theo Normal Flow).

---

## 4. Bản đồ Grid

| Vùng | Flex được không? | Grid phù hợp hơn? | Lý do |
|---|---|---|---|
| `stats-list` (5 cột đều) | Được | **Có** | Cột đều, Flex wrap sẽ lệch kiểu 3+2 |
| `feature-grid` (ô to nhỏ khác nhau) | Khó | **Có** | Cần `grid-row: span 2` |
| `gallery` (14 ảnh) | Không tốt | **Có** | Cấu trúc hàng × cột |
| `topic-featured` (3 card) | Được | Có | `auto-fit` tự chia cột theo bề rộng |
| `footer-main` | Được | Ngang nhau | Cột cố định, hai cách như nhau |
| `nav-container`, `carousel-track`, `notice-item` | **Có** | Không | Một chiều, cần co giãn/trượt |

Quy tắc ngắn: **một chiều (hàng hoặc cột) → Flex; hai chiều (hàng và cột) → Grid**.

---

## 5. Responsive (dự kiến)

| Vùng | Desktop | Tablet | Mobile |
|---|---|---|---|
| Header | logo \| menu 8 mục \| tools | giống desktop hoặc menu gọn | logo + nút ☰, **menu drawer** (bản menu thứ hai) |
| News carousel | 3 card/hàng | 2 | 1 |
| Stats | 5 cột | 3 + 2 | 2 hoặc 1 cột |
| Feature grid | 3 cột (Tuyển sinh span 2 hàng) | 2 cột | 1 cột |
| Units | list \| detail (1:2) | giữ | list xếp trên detail |
| Footer | 4 cột | 2 + 2 | 1 cột |

```css
.carousel-track .news-card { flex-basis: 100%; }
@media (min-width: 768px)  { .carousel-track .news-card { flex-basis: calc(50% - 10px); } }
@media (min-width: 1024px) { .carousel-track .news-card { flex-basis: calc(33.333% - 14px); } }

.units { flex-direction: column; }
@media (min-width: 900px) { .units { flex-direction: row; } }
```

Với Grid, `repeat(auto-fit, minmax(160px, 1fr))` ở mục 2.6 tự co theo bề rộng mà không cần media query.

---

## 6. DOM semantic hoàn chỉnh

```text
body
├── header
│   ├── a > img                (logo)
│   ├── nav > ul > li ×8       (menu chính)
│   └── div: button + button   (tìm kiếm, ngôn ngữ)
├── main
│   ├── section.hero           (h1 ẩn + slides)
│   ├── section.news           (h2 + article ×N)
│   ├── section.notices        (h2 + ul > li > a + time)
│   ├── section.events         (h2 + article ×N)
│   ├── section.stats          (h2 + dl)
│   ├── section.feature-grid   (article ×4)
│   ├── section.units          (h2 + ul > li > a)
│   ├── section.topic ×3       (h2 + article + ul)
│   ├── section.partners       (h2 + p + ul > li > img)
│   └── section.gallery        (h2 + ul > li > img)
└── footer
    ├── address
    ├── nav ×3                 (h3 + ul)
    └── p                      (bản quyền)
```

Mẹo cho "heading outline": tiêu đề các khối "Tin Tức", "Thông Báo", "Sự Kiện" hiện là link, nên cần đặt trong `h2` (`<h2><a>…</a></h2>`) để người dùng công cụ đọc màn hình duyệt được theo tiêu đề.

---

## 7. Bảng tổng hợp

| Component | Layout Box | Kỹ thuật | Flex Container? | Flex Items | Nội dung | HTML semantic |
|---|---|---|---|---|---|---|
| Header | header-container | Flex | Có | brand, nav, header-tools | logo, menu, tìm kiếm | `header`, `a`, `img`, `button` |
| Navigation | nav-container | Flex + Position (submenu) | Có | nav-item | menu 2 cấp | `nav`, `ul`, `li`, `a` |
| Hero | hero-slides | Flex nowrap / Position | Có (cách A) | hero-slide | ảnh, video | `section`, `img`, `video` |
| News card | news-card | Flex column | Có | image, body | bài viết | `article`, `a`, `img`, `h3`, `p`, `time` |
| News carousel | carousel-track | Flex nowrap | Có | news-card | danh sách card | `section`, `button` (điều khiển) |
| Notices | notice-item | Flex row | Có | title, date | thông báo | `ul`, `li`, `a`, `time` |
| Events | event-card | Position + Flex | Có | day, month | sự kiện | `article`, `time`, `h3` |
| Stats | stats-list | Grid | Không | – | số liệu | `section`, `dl`, `dt`, `dd` |
| Feature grid | feature-grid | Grid | Không | – | 4 khối giới thiệu | `article`, `h2`, `a`, `ul` |
| Units | units | Flex 1:2 | Có | unit-list, unit-detail | danh sách khoa | `ul`, `li`, `a`, `p` |
| Topic section ×3 | topic-body | Flex 2:1 + Grid | Có | featured, list | bài + danh sách | `section`, `article`, `ul` |
| Partners | partner-logos | Flex + wrap | Có | logo | logo đối tác | `ul`, `li`, `img` |
| Gallery | gallery | Grid | Không | – | ảnh | `ul`, `li`, `img` (hoặc `figure`) |
| Footer | footer-main | Flex + wrap (2:1:1:1) | Có | footer-brand, footer-column | liên hệ, link | `footer`, `address`, `nav`, `h3`, `ul` |

---

## 8. Checklist tư duy cho sinh viên

```text
Nhìn thấy một vùng giao diện
        ↓
Khoanh hộp → Đặt tên theo vai trò (news-card, không phải box1)
        ↓
Xác định cha – con
        ↓
Các con có đứng cạnh nhau trên một hàng/cột?
   ├── Có, một chiều                → Flexbox
   │     └── Cần trượt/co giãn? → nowrap; cần xuống dòng? → wrap
   └── Không
        ↓
   Có cấu trúc hàng × cột, ô to nhỏ khác nhau?
   ├── Có → Grid
   └── Không
        ↓
   Có nổi lên trên phần tử khác (submenu, badge, nút ‹ ›)?
   ├── Có → Position (cha relative, con absolute)
   └── Không → Normal Flow (không cần làm gì)
        ↓
Nội dung là gì? tiêu đề / đoạn văn / link / nút / ảnh / danh sách / ngày
        ↓
Chọn HTML semantic
```

**Phân biệt nhanh link và button:**

- Bấm để **chuyển trang** → `a` (kể cả khi trông như nút, ví dụ "Đăng ký ngay").
- Bấm để **thực hiện hành động trên trang** (mở tìm kiếm, mở menu, chạy slide, tải bản đồ) → `button`.

---

## 9. Các lỗi tư duy thường gặp

1. **Đặt `display: flex` lên phần tử cần xếp dọc.** Block đã tự xếp dọc, thêm Flex là thừa (ví dụ `submenu`, `topic-list`).
2. **Quên `position: relative` ở cha** khi dùng `absolute` ở con, nên nhãn "Hot" hay ô ngày bay lên góc trang.
3. **Dùng Flex wrap cho lưới số lượng lẻ** như 5 chỉ số, dẫn tới hàng cuối lệch. Dùng Grid với `auto-fit`.
4. **Dùng `div` hoặc `span` cho mọi thứ**: tiêu đề phải là `h2/h3`, danh sách phải là `ul/li`, ngày phải là `time`.
5. **Nhầm trục khi dùng `justify-content`/`align-items`.** Với `flex-direction: column` thì hai thuộc tính này đổi vai trò (trục chính là dọc).
6. **Carousel dùng `flex-wrap: wrap`**: card sẽ xuống dòng thay vì trượt ngang. Carousel phải là `nowrap` kèm `overflow: hidden` ở cha.
7. **Đặt tên theo hình dạng** (`left-box`, `blue-box`) thay vì vai trò (`unit-list`, `news-card`).
8. **Tách rời ba lớp**: nhìn thấy "ảnh + chữ" rồi chỉ nghĩ tới `img`, quên hỏi hộp bao ngoài là gì và xếp bằng kỹ thuật nào.

---

## Việc cần kiểm tra bằng DevTools

- Menu thứ hai trong mã nguồn có đúng là drawer mobile không (thu nhỏ cửa sổ).
- Hero là slider hay chồng lớp; có `h1` thật không.
- Danh sách tin/sự kiện bị lặp có phải bản sao của carousel loop không.
- Số liệu thống kê có phải bộ đếm JS không.
- Khối "Các đơn vị" có đổi nội dung khi chọn/rê chuột không.
- `topic-body` có thật sự chia hai nhóm cạnh nhau, và logo đối tác nạp ra sao.
