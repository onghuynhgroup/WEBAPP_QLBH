# NGHIÊN CỨU CHUYÊN SÂU: NỀN TẢNG QUẢN LÝ PHÂN PHỐI SỈ (WHOLESALE DISTRIBUTION PLATFORM)

> **Ngày nghiên cứu:** 13/02/2026
> **Mục tiêu:** Xây dựng hệ thống Web/App quản lý bán hàng buôn sỉ, phân phối đại lý đa cấp, đa ngành hàng
> **Đối tượng khách hàng:** Đại lý cấp 1, cấp 2, cấp 3, doanh nghiệp mua sỉ

---

## MỤC LỤC

1. [Tổng quan yêu cầu hệ thống](#1-tổng-quan-yêu-cầu-hệ-thống)
2. [Danh sách tính năng chi tiết](#2-danh-sách-tính-năng-chi-tiết)
3. [Thiết kế cơ sở dữ liệu](#3-thiết-kế-cơ-sở-dữ-liệu)
4. [Phương án 1: Laravel + Filament + Vue.js](#4-phương-án-1-laravel--filament--vuejs--postgresql)
5. [Phương án 2: Next.js Full-Stack + Medusa.js](#5-phương-án-2-nextjs-full-stack--medusajs--postgresql)
6. [Phương án 3: Django + Django Oscar + React](#6-phương-án-3-django--django-oscar--react--postgresql)
7. [So sánh 3 phương án](#7-so-sánh-3-phương-án)
8. [Kiến trúc hệ thống chung](#8-kiến-trúc-hệ-thống-chung)
9. [Lộ trình triển khai](#9-lộ-trình-triển-khai)
10. [Kết luận & Đề xuất](#10-kết-luận--đề-xuất)

---

## 1. TỔNG QUAN YÊU CẦU HỆ THỐNG

### 1.1 Mô tả doanh nghiệp
- Doanh nghiệp chuyên **phân phối sỉ đa ngành hàng**
- Khách hàng là **đại lý, doanh nghiệp** (B2B - Business to Business)
- Hệ thống phân cấp đại lý: **Cấp 1 → Cấp 2 → Cấp 3** (mỗi cấp có chính sách giá riêng)
- Quản lý **công nợ** cho từng đại lý
- Nhiều **chương trình khuyến mãi** theo cấp đại lý, theo ngành hàng

### 1.2 Các vai trò người dùng (User Roles)

| Vai trò | Mô tả | Quyền hạn chính |
|---------|--------|-----------------|
| **Super Admin** | Chủ doanh nghiệp / Ban giám đốc | Toàn quyền hệ thống, xem báo cáo tổng hợp |
| **Admin** | Quản lý vận hành | Quản lý sản phẩm, đơn hàng, đại lý, khuyến mãi |
| **Kế toán** | Nhân viên kế toán | Quản lý công nợ, thanh toán, báo cáo tài chính |
| **Nhân viên kinh doanh** | Sales / CSKH | Tạo đơn hàng, chăm sóc đại lý, xem báo cáo bán hàng |
| **Nhân viên kho** | Quản lý kho hàng | Nhập/xuất kho, kiểm kê, quản lý tồn kho |
| **Đại lý Cấp 1** | Nhà phân phối lớn | Đặt hàng giá cấp 1, xem công nợ, quản lý đơn hàng |
| **Đại lý Cấp 2** | Đại lý trung gian | Đặt hàng giá cấp 2, xem công nợ |
| **Đại lý Cấp 3** | Đại lý nhỏ / cửa hàng | Đặt hàng giá cấp 3, xem công nợ |

---

## 2. DANH SÁCH TÍNH NĂNG CHI TIẾT

### 2.1 Module Xác thực & Phân quyền (Authentication & Authorization)
- Đăng nhập/Đăng ký cho từng đại lý (email/SĐT + mật khẩu)
- Đăng nhập riêng cho nhân viên công ty (Admin portal)
- Xác thực 2 lớp (2FA) cho tài khoản quản trị
- Phân quyền theo vai trò (RBAC - Role-Based Access Control)
- Quản lý phiên đăng nhập (session management)
- Đặt lại mật khẩu qua email/SMS
- Đại lý tự đăng ký → Admin duyệt → Gán cấp đại lý

### 2.2 Module Quản lý Đại lý (Agency Management)
- **Hồ sơ đại lý:** Tên, địa chỉ, MST, người đại diện, SĐT, email, giấy phép KD
- **Phân cấp đại lý:** Cấp 1, Cấp 2, Cấp 3 (mỗi cấp chính sách giá khác nhau)
- **Cây phân cấp đại lý:** Đại lý cấp 1 quản lý các đại lý cấp 2 bên dưới
- **Hạn mức tín dụng:** Mỗi đại lý có hạn mức công nợ tối đa
- **Trạng thái:** Hoạt động / Tạm ngưng / Khóa (khi nợ quá hạn)
- **Lịch sử hoạt động:** Log mọi giao dịch của đại lý
- **Đo lường đại lý:**
  - Doanh số theo tháng/quý/năm
  - Tần suất đặt hàng
  - Tỷ lệ thanh toán đúng hạn
  - Xếp hạng đại lý (A/B/C/D)
  - Đề xuất nâng/hạ cấp đại lý tự động

### 2.3 Module Quản lý Sản phẩm (Product Management)
- **Thông tin sản phẩm:**
  - Tên, mã SKU, mã vạch (barcode)
  - Mô tả ngắn, mô tả chi tiết (rich text editor)
  - Đơn vị tính (cái, hộp, thùng, kg, lốc,...)
  - Quy cách đóng gói (1 thùng = 24 hộp, 1 lốc = 6 chai,...)
  - Trọng lượng, kích thước
  - Xuất xứ, thương hiệu, nhà sản xuất
- **Danh mục đa cấp:**
  - Ngành hàng → Nhóm hàng → Loại hàng (không giới hạn cấp)
  - Một sản phẩm có thể thuộc **nhiều danh mục** (multi-category)
- **Thư viện hình ảnh:**
  - Upload nhiều hình ảnh cho mỗi sản phẩm
  - Hình ảnh chính / hình ảnh phụ
  - Tự động tạo thumbnail các kích cỡ
  - Quản lý thư viện hình ảnh tập trung (Media Library)
  - Hỗ trợ kéo thả (drag & drop), sắp xếp thứ tự hình
  - Tối ưu dung lượng ảnh tự động (compression)
- **Biến thể sản phẩm:** Màu sắc, kích thước, hương vị,...
- **Trạng thái:** Đang bán / Ngừng bán / Hết hàng / Sắp ra mắt
- **Nhập/Xuất hàng loạt:** Import/Export sản phẩm qua Excel/CSV

### 2.4 Module Quản lý Giá theo Cấp Đại lý (Multi-Tier Pricing)
- **Giá gốc (Base Price):** Giá niêm yết / giá bán lẻ đề xuất
- **Giá theo cấp đại lý:**

  | Cấp đại lý | Chiết khấu mặc định | Ví dụ (Giá gốc 100K) |
  |-------------|---------------------|-----------------------|
  | Cấp 1 | 30-40% | 60,000 - 70,000 VNĐ |
  | Cấp 2 | 20-30% | 70,000 - 80,000 VNĐ |
  | Cấp 3 | 10-20% | 80,000 - 90,000 VNĐ |

- **Giá theo số lượng (Volume Pricing):**
  - Mua 1-99: Giá A
  - Mua 100-499: Giá B (thấp hơn)
  - Mua 500+: Giá C (thấp nhất)
- **Giá theo hợp đồng:** Giá riêng cho từng đại lý cụ thể
- **Giá có hiệu lực theo thời gian:** Lên lịch thay đổi giá (từ ngày → đến ngày)
- **Lịch sử giá:** Lưu toàn bộ lịch sử thay đổi giá
- **Xuất bảng giá:** Xuất bảng giá theo cấp đại lý ra PDF/Excel

### 2.5 Module Quản lý Đơn hàng (Order Management)
- **Quy trình đơn hàng:**
  ```
  Đặt hàng → Xác nhận → Đang xử lý → Đang giao → Đã giao → Hoàn thành
                ↓                                        ↓
            Từ chối                                  Trả hàng
  ```
- **Tạo đơn hàng:**
  - Đại lý tự đặt hàng qua portal
  - Nhân viên kinh doanh tạo hộ đại lý
  - Đặt hàng nhanh bằng mã SKU
  - Import đơn hàng từ Excel
  - Đặt lại đơn hàng cũ (re-order)
- **Xử lý đơn hàng:**
  - Auto tính giá theo cấp đại lý
  - Auto kiểm tra hạn mức công nợ trước khi duyệt
  - Auto kiểm tra tồn kho
  - Ghi chú nội bộ / ghi chú cho đại lý
  - In phiếu xuất kho, hóa đơn, phiếu giao hàng
- **Quản lý trả hàng:**
  - Tạo phiếu trả hàng
  - Ghi nhận lý do trả hàng
  - Hoàn tiền hoặc giảm công nợ

### 2.6 Module Quản lý Công nợ (Debt/Credit Management)
- **Sổ công nợ (Debt Ledger):**
  - Ghi nhận theo kiểu sổ kép (double-entry)
  - Mỗi giao dịch: loại (phát sinh nợ / thanh toán / điều chỉnh / hoàn trả)
  - Số dư luỹ kế (running balance) theo thời gian thực
- **Hạn mức tín dụng:**
  - Thiết lập hạn mức cho từng đại lý
  - Tự động chặn đơn hàng khi vượt hạn mức
  - Cảnh báo khi đạt 80%, 90%, 100% hạn mức
- **Kỳ hạn thanh toán:**
  - Cấp 1: 30 ngày
  - Cấp 2: 15 ngày
  - Cấp 3: 7 ngày (hoặc thanh toán trước)
- **Theo dõi nợ quá hạn:**
  - Phân loại: Trong hạn / Quá hạn 1-30 ngày / Quá hạn 31-60 / Quá hạn >60
  - Tự động gửi nhắc nhở thanh toán
  - Tự động tạm ngưng tài khoản khi nợ quá hạn >60 ngày
- **Ghi nhận thanh toán:**
  - Thanh toán tiền mặt, chuyển khoản, séc
  - Đối soát ngân hàng
  - Upload chứng từ thanh toán
- **Báo cáo công nợ:**
  - Bảng tổng hợp công nợ theo đại lý
  - Tuổi nợ (Aging Report)
  - Dự báo dòng tiền thu về

### 2.7 Module Chương trình Khuyến mãi (Promotion Management)
- **Loại khuyến mãi:**
  - Giảm giá theo % hoặc số tiền cố định
  - Mua X tặng Y (Buy X Get Y Free)
  - Combo / Bundle (mua kèm giá ưu đãi)
  - Tặng quà khi đạt giá trị đơn hàng
  - Chiết khấu thêm theo số lượng
  - Tích điểm thưởng cho đại lý
- **Phạm vi áp dụng:**
  - Theo cấp đại lý (chỉ cấp 1, hoặc tất cả)
  - Theo danh mục sản phẩm
  - Theo sản phẩm cụ thể
  - Theo vùng miền / khu vực
  - Theo giá trị đơn hàng tối thiểu
- **Thời gian:** Ngày bắt đầu → Ngày kết thúc, giới hạn số lần sử dụng
- **Kết hợp khuyến mãi:** Cho phép hoặc không cho phép áp dụng đồng thời nhiều KM

### 2.8 Module Quản lý Kho (Inventory Management)
- **Quản lý tồn kho:**
  - Tồn kho theo sản phẩm, biến thể, kho
  - Cảnh báo tồn kho thấp (safety stock)
  - Hỗ trợ đa kho (multi-warehouse)
- **Nhập kho:** Phiếu nhập kho, nhập từ nhà cung cấp
- **Xuất kho:** Tự động tạo phiếu xuất khi đơn hàng được duyệt
- **Kiểm kê:** Kiểm kê định kỳ, điều chỉnh tồn kho
- **Lịch sử xuất nhập:** Log đầy đủ mọi biến động kho

### 2.9 Module Dashboard & Báo cáo (Reporting & Analytics)
- **Dashboard tổng quan (cho Admin):**
  - Doanh thu hôm nay / tuần / tháng / năm
  - Số đơn hàng mới, đang xử lý, hoàn thành
  - Top 10 sản phẩm bán chạy
  - Top 10 đại lý doanh số cao nhất
  - Tổng công nợ / Nợ quá hạn
  - Biểu đồ doanh thu theo thời gian (line chart)
  - Biểu đồ phân bổ doanh thu theo ngành hàng (pie chart)
  - Biểu đồ so sánh doanh số đại lý (bar chart)
- **Dashboard cho Đại lý:**
  - Đơn hàng gần đây, trạng thái đơn hàng
  - Công nợ hiện tại, kỳ hạn thanh toán
  - Sản phẩm mới, khuyến mãi đang áp dụng
  - Lịch sử mua hàng
- **Báo cáo chi tiết:**
  - Doanh thu theo đại lý / cấp đại lý / khu vực
  - Doanh thu theo sản phẩm / danh mục / ngành hàng
  - Doanh thu theo nhân viên kinh doanh
  - Báo cáo lợi nhuận gộp
  - Báo cáo tồn kho / hàng chậm luân chuyển
  - Báo cáo công nợ / tuổi nợ
  - Xuất báo cáo ra Excel / PDF

### 2.10 Module Thông báo (Notification)
- Thông báo đơn hàng mới (cho Admin)
- Thông báo trạng thái đơn hàng (cho Đại lý)
- Nhắc nhở thanh toán công nợ
- Thông báo khuyến mãi mới
- Cảnh báo tồn kho thấp
- Thông báo qua: Email, SMS, Push notification (nếu có app), In-app notification

---

## 3. THIẾT KẾ CƠ SỞ DỮ LIỆU

### 3.1 Sơ đồ quan hệ các bảng chính (ERD)

```
┌──────────────┐     ┌──────────────────┐     ┌───────────────┐
│    users      │     │    agencies       │     │ pricing_tiers │
├──────────────┤     ├──────────────────┤     ├───────────────┤
│ id (PK)      │     │ id (PK)          │     │ id (PK)       │
│ name         │◄────│ user_id (FK)     │     │ tier_name     │
│ email        │     │ tier_id (FK)     │────►│ tier_level    │
│ password     │     │ parent_agency_id │     │ discount_%    │
│ role         │     │ company_name     │     │ min_order     │
│ phone        │     │ tax_code         │     │ credit_days   │
│ avatar       │     │ credit_limit     │     └───────────────┘
│ status       │     │ current_debt     │
│ created_at   │     │ rating           │
└──────────────┘     │ status           │
                     └──────────────────┘
                              │
           ┌──────────────────┼──────────────────┐
           ▼                  ▼                  ▼
┌──────────────────┐ ┌──────────────┐  ┌─────────────────┐
│     orders        │ │ debt_ledger  │  │agency_addresses │
├──────────────────┤ ├──────────────┤  ├─────────────────┤
│ id (PK)          │ │ id (PK)      │  │ id (PK)         │
│ agency_id (FK)   │ │ agency_id(FK)│  │ agency_id (FK)  │
│ order_number     │ │ type         │  │ address_type    │
│ status           │ │ reference_id │  │ province        │
│ total_amount     │ │ amount       │  │ district        │
│ discount_amount  │ │ balance      │  │ ward            │
│ final_amount     │ │ notes        │  │ address_detail  │
│ payment_status   │ │ created_by   │  │ phone           │
│ payment_due_date │ │ created_at   │  └─────────────────┘
│ applied_tier_id  │ └──────────────┘
│ salesperson_id   │
│ notes            │
│ created_at       │
└──────────────────┘
         │
         ▼
┌──────────────────┐    ┌───────────────┐    ┌──────────────────┐
│   order_items     │    │   products    │    │product_tier_prices│
├──────────────────┤    ├───────────────┤    ├──────────────────┤
│ id (PK)          │    │ id (PK)       │    │ id (PK)          │
│ order_id (FK)    │    │ name          │    │ product_id (FK)  │
│ product_id (FK)  │───►│ sku           │◄───│ tier_id (FK)     │
│ variant_id (FK)  │    │ barcode       │    │ unit_price       │
│ quantity         │    │ description   │    │ min_quantity     │
│ unit_price       │    │ base_price    │    │ max_quantity     │
│ discount_%       │    │ cost_price    │    │ effective_from   │
│ line_total       │    │ unit          │    │ effective_to     │
│ price_snapshot   │    │ brand_id      │    └──────────────────┘
└──────────────────┘    │ status        │
                        │ created_at    │
                        └───────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
   ┌─────────────────┐ ┌────────────┐ ┌──────────────────┐
   │product_categories│ │product_imgs│ │product_variants  │
   ├─────────────────┤ ├────────────┤ ├──────────────────┤
   │ product_id (FK) │ │ id (PK)    │ │ id (PK)          │
   │ category_id(FK) │ │ product_id │ │ product_id (FK)  │
   └─────────────────┘ │ image_url  │ │ variant_name     │
                        │ sort_order │ │ sku              │
   ┌───────────────┐   │ is_primary │ │ price_adjustment │
   │  categories   │   └────────────┘ │ stock_quantity   │
   ├───────────────┤                   └──────────────────┘
   │ id (PK)       │
   │ name          │   ┌──────────────────┐
   │ parent_id(FK) │   │   promotions      │
   │ slug          │   ├──────────────────┤
   │ icon          │   │ id (PK)          │
   │ sort_order    │   │ name             │
   └───────────────┘   │ type             │
                        │ discount_value   │
   ┌───────────────┐   │ applicable_tiers │
   │   warehouses  │   │ applicable_cats  │
   ├───────────────┤   │ min_order_value  │
   │ id (PK)       │   │ start_date       │
   │ name          │   │ end_date         │
   │ address       │   │ is_active        │
   │ manager_id    │   │ usage_limit      │
   └───────────────┘   └──────────────────┘
         │
         ▼
   ┌──────────────────┐
   │inventory_movements│
   ├──────────────────┤
   │ id (PK)          │
   │ warehouse_id(FK) │
   │ product_id (FK)  │
   │ type (in/out/adj)│
   │ quantity         │
   │ reference_type   │
   │ reference_id     │
   │ notes            │
   │ created_by       │
   │ created_at       │
   └──────────────────┘
```

### 3.2 Chi tiết các bảng quan trọng (SQL Schema)

```sql
-- ===================================================
-- BẢNG NGƯỜI DÙNG
-- ===================================================
CREATE TABLE users (
    id              BIGSERIAL PRIMARY KEY,
    name            VARCHAR(255) NOT NULL,
    email           VARCHAR(255) UNIQUE NOT NULL,
    phone           VARCHAR(20),
    password_hash   VARCHAR(255) NOT NULL,
    role            VARCHAR(50) NOT NULL DEFAULT 'agency',
    -- Roles: super_admin, admin, accountant, salesperson, warehouse_staff, agency
    avatar_url      VARCHAR(500),
    is_active       BOOLEAN DEFAULT TRUE,
    two_factor_enabled BOOLEAN DEFAULT FALSE,
    email_verified_at TIMESTAMP,
    last_login_at   TIMESTAMP,
    created_at      TIMESTAMP DEFAULT NOW(),
    updated_at      TIMESTAMP DEFAULT NOW()
);

-- ===================================================
-- BẢNG CẤP ĐẠI LÝ
-- ===================================================
CREATE TABLE pricing_tiers (
    id                      SERIAL PRIMARY KEY,
    tier_name               VARCHAR(100) NOT NULL,  -- 'Đại lý Cấp 1', 'Đại lý Cấp 2',...
    tier_level              INT NOT NULL UNIQUE,    -- 1, 2, 3
    default_discount_percent DECIMAL(5,2) DEFAULT 0,
    min_order_value         DECIMAL(15,2) DEFAULT 0,
    credit_term_days        INT DEFAULT 0,          -- Số ngày cho nợ
    description             TEXT,
    created_at              TIMESTAMP DEFAULT NOW()
);

-- ===================================================
-- BẢNG ĐẠI LÝ
-- ===================================================
CREATE TABLE agencies (
    id                  BIGSERIAL PRIMARY KEY,
    user_id             BIGINT REFERENCES users(id),
    tier_id             INT REFERENCES pricing_tiers(id),
    parent_agency_id    BIGINT REFERENCES agencies(id),  -- Cây phân cấp
    company_name        VARCHAR(255) NOT NULL,
    tax_code            VARCHAR(50),
    representative_name VARCHAR(255),
    phone               VARCHAR(20),
    email               VARCHAR(255),
    business_license_url VARCHAR(500),
    credit_limit        DECIMAL(15,2) DEFAULT 0,
    current_debt        DECIMAL(15,2) DEFAULT 0,
    rating              VARCHAR(5) DEFAULT 'C',  -- A, B, C, D
    status              VARCHAR(20) DEFAULT 'pending',
    -- Status: pending, active, suspended, blocked
    province            VARCHAR(100),
    district            VARCHAR(100),
    ward                VARCHAR(100),
    address_detail      TEXT,
    notes               TEXT,
    approved_at         TIMESTAMP,
    approved_by         BIGINT REFERENCES users(id),
    created_at          TIMESTAMP DEFAULT NOW(),
    updated_at          TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_agencies_tier ON agencies(tier_id);
CREATE INDEX idx_agencies_parent ON agencies(parent_agency_id);
CREATE INDEX idx_agencies_status ON agencies(status);

-- ===================================================
-- BẢNG DANH MỤC SẢN PHẨM (đa cấp, không giới hạn)
-- ===================================================
CREATE TABLE categories (
    id          SERIAL PRIMARY KEY,
    name        VARCHAR(255) NOT NULL,
    slug        VARCHAR(255) UNIQUE NOT NULL,
    parent_id   INT REFERENCES categories(id),
    icon        VARCHAR(255),
    description TEXT,
    sort_order  INT DEFAULT 0,
    is_active   BOOLEAN DEFAULT TRUE,
    created_at  TIMESTAMP DEFAULT NOW()
);

-- ===================================================
-- BẢNG SẢN PHẨM
-- ===================================================
CREATE TABLE products (
    id              BIGSERIAL PRIMARY KEY,
    name            VARCHAR(255) NOT NULL,
    slug            VARCHAR(255) UNIQUE NOT NULL,
    sku             VARCHAR(100) UNIQUE NOT NULL,
    barcode         VARCHAR(100),
    short_description TEXT,
    description     TEXT,               -- Rich text (HTML)
    base_price      DECIMAL(15,2) NOT NULL,  -- Giá niêm yết
    cost_price      DECIMAL(15,2),           -- Giá vốn
    unit            VARCHAR(50) NOT NULL,     -- Cái, hộp, thùng, kg,...
    packing_spec    VARCHAR(255),            -- '1 thùng = 24 hộp'
    weight          DECIMAL(10,3),
    dimensions      VARCHAR(100),            -- 'DxRxC cm'
    brand_id        INT REFERENCES brands(id),
    origin          VARCHAR(100),            -- Xuất xứ
    status          VARCHAR(20) DEFAULT 'active',
    -- Status: active, inactive, out_of_stock, coming_soon
    is_featured     BOOLEAN DEFAULT FALSE,
    min_order_qty   INT DEFAULT 1,
    stock_quantity  INT DEFAULT 0,
    low_stock_threshold INT DEFAULT 10,
    meta_data       JSONB,                   -- Thuộc tính mở rộng
    created_at      TIMESTAMP DEFAULT NOW(),
    updated_at      TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_products_sku ON products(sku);
CREATE INDEX idx_products_status ON products(status);
CREATE INDEX idx_products_brand ON products(brand_id);

-- Bảng liên kết sản phẩm - danh mục (many-to-many)
CREATE TABLE product_categories (
    product_id  BIGINT REFERENCES products(id) ON DELETE CASCADE,
    category_id INT REFERENCES categories(id) ON DELETE CASCADE,
    PRIMARY KEY (product_id, category_id)
);

-- ===================================================
-- BẢNG HÌNH ẢNH SẢN PHẨM (Thư viện hình ảnh)
-- ===================================================
CREATE TABLE product_images (
    id          BIGSERIAL PRIMARY KEY,
    product_id  BIGINT REFERENCES products(id) ON DELETE CASCADE,
    image_url   VARCHAR(500) NOT NULL,
    thumbnail_url VARCHAR(500),
    alt_text    VARCHAR(255),
    sort_order  INT DEFAULT 0,
    is_primary  BOOLEAN DEFAULT FALSE,
    file_size   INT,                -- bytes
    created_at  TIMESTAMP DEFAULT NOW()
);

-- Thư viện hình ảnh tập trung (Media Library)
CREATE TABLE media_library (
    id          BIGSERIAL PRIMARY KEY,
    file_name   VARCHAR(255) NOT NULL,
    file_path   VARCHAR(500) NOT NULL,
    file_type   VARCHAR(50),         -- image/jpeg, image/png,...
    file_size   INT,
    thumbnail_path VARCHAR(500),
    alt_text    VARCHAR(255),
    folder      VARCHAR(255) DEFAULT 'general',
    uploaded_by BIGINT REFERENCES users(id),
    created_at  TIMESTAMP DEFAULT NOW()
);

-- ===================================================
-- BẢNG GIÁ THEO CẤP ĐẠI LÝ
-- ===================================================
CREATE TABLE product_tier_prices (
    id              BIGSERIAL PRIMARY KEY,
    product_id      BIGINT REFERENCES products(id) ON DELETE CASCADE,
    tier_id         INT REFERENCES pricing_tiers(id),
    unit_price      DECIMAL(15,2) NOT NULL,
    min_quantity    INT DEFAULT 1,
    max_quantity    INT,
    effective_from  TIMESTAMP NOT NULL DEFAULT NOW(),
    effective_to    TIMESTAMP,
    created_by      BIGINT REFERENCES users(id),
    created_at      TIMESTAMP DEFAULT NOW()
);

CREATE UNIQUE INDEX idx_tier_price_unique
    ON product_tier_prices(product_id, tier_id, min_quantity, effective_from);

-- Giá riêng cho đại lý cụ thể (hợp đồng)
CREATE TABLE product_agency_prices (
    id          BIGSERIAL PRIMARY KEY,
    product_id  BIGINT REFERENCES products(id) ON DELETE CASCADE,
    agency_id   BIGINT REFERENCES agencies(id) ON DELETE CASCADE,
    unit_price  DECIMAL(15,2) NOT NULL,
    min_quantity INT DEFAULT 1,
    effective_from TIMESTAMP NOT NULL DEFAULT NOW(),
    effective_to   TIMESTAMP,
    notes       TEXT,
    created_at  TIMESTAMP DEFAULT NOW()
);

-- ===================================================
-- BẢNG ĐƠN HÀNG
-- ===================================================
CREATE TABLE orders (
    id                  BIGSERIAL PRIMARY KEY,
    order_number        VARCHAR(50) UNIQUE NOT NULL,  -- VD: ORD-20260213-0001
    agency_id           BIGINT REFERENCES agencies(id),
    salesperson_id      BIGINT REFERENCES users(id),
    applied_tier_id     INT REFERENCES pricing_tiers(id),
    status              VARCHAR(30) DEFAULT 'pending',
    -- Status: pending, confirmed, processing, shipping, delivered, completed, cancelled, returned
    subtotal_amount     DECIMAL(15,2) NOT NULL DEFAULT 0,
    discount_amount     DECIMAL(15,2) DEFAULT 0,
    promotion_id        BIGINT REFERENCES promotions(id),
    shipping_fee        DECIMAL(15,2) DEFAULT 0,
    final_amount        DECIMAL(15,2) NOT NULL DEFAULT 0,
    payment_status      VARCHAR(20) DEFAULT 'unpaid',
    -- Payment: unpaid, partial, paid
    payment_method      VARCHAR(50),
    payment_due_date    DATE,
    shipping_address    TEXT,
    shipping_province   VARCHAR(100),
    internal_notes      TEXT,
    agency_notes        TEXT,
    confirmed_at        TIMESTAMP,
    confirmed_by        BIGINT REFERENCES users(id),
    shipped_at          TIMESTAMP,
    delivered_at        TIMESTAMP,
    completed_at        TIMESTAMP,
    cancelled_at        TIMESTAMP,
    cancel_reason       TEXT,
    created_at          TIMESTAMP DEFAULT NOW(),
    updated_at          TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_orders_agency ON orders(agency_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_payment ON orders(payment_status);
CREATE INDEX idx_orders_date ON orders(created_at);

-- ===================================================
-- BẢNG CHI TIẾT ĐƠN HÀNG
-- ===================================================
CREATE TABLE order_items (
    id              BIGSERIAL PRIMARY KEY,
    order_id        BIGINT REFERENCES orders(id) ON DELETE CASCADE,
    product_id      BIGINT REFERENCES products(id),
    variant_id      BIGINT REFERENCES product_variants(id),
    product_name    VARCHAR(255) NOT NULL,  -- Snapshot tên SP
    product_sku     VARCHAR(100) NOT NULL,  -- Snapshot SKU
    quantity        INT NOT NULL,
    unit_price      DECIMAL(15,2) NOT NULL, -- Giá tại thời điểm đặt
    discount_percent DECIMAL(5,2) DEFAULT 0,
    discount_amount DECIMAL(15,2) DEFAULT 0,
    line_total      DECIMAL(15,2) NOT NULL,
    tier_price_snapshot JSONB,  -- Lưu chi tiết rule giá đã áp dụng
    notes           TEXT,
    created_at      TIMESTAMP DEFAULT NOW()
);

-- ===================================================
-- SỔ CÔNG NỢ (Double-Entry Style)
-- ===================================================
CREATE TABLE debt_ledger (
    id                  BIGSERIAL PRIMARY KEY,
    agency_id           BIGINT REFERENCES agencies(id),
    transaction_type    VARCHAR(30) NOT NULL,
    -- Types: order_charge, payment, credit_adjustment, refund, write_off
    reference_type      VARCHAR(30),  -- 'order', 'payment', 'adjustment'
    reference_id        BIGINT,       -- ID của đơn hàng hoặc phiếu thanh toán
    amount              DECIMAL(15,2) NOT NULL,
    -- Dương = phát sinh nợ, Âm = thanh toán/giảm nợ
    running_balance     DECIMAL(15,2) NOT NULL, -- Số dư sau giao dịch
    payment_method      VARCHAR(50),
    bank_reference      VARCHAR(100),  -- Số tham chiếu ngân hàng
    attachment_url      VARCHAR(500),  -- Chứng từ thanh toán
    notes               TEXT,
    created_by          BIGINT REFERENCES users(id),
    created_at          TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_debt_agency ON debt_ledger(agency_id);
CREATE INDEX idx_debt_date ON debt_ledger(created_at);
CREATE INDEX idx_debt_type ON debt_ledger(transaction_type);

-- ===================================================
-- BẢNG KHUYẾN MÃI
-- ===================================================
CREATE TABLE promotions (
    id                  BIGSERIAL PRIMARY KEY,
    name                VARCHAR(255) NOT NULL,
    code                VARCHAR(50) UNIQUE,  -- Mã khuyến mãi (nếu có)
    description         TEXT,
    type                VARCHAR(30) NOT NULL,
    -- Types: percentage, fixed_amount, buy_x_get_y, bundle, gift
    discount_value      DECIMAL(15,2),
    buy_quantity        INT,          -- Mua X
    get_quantity        INT,          -- Tặng Y
    applicable_tiers    JSONB,        -- [1, 2, 3] hoặc null = tất cả
    applicable_categories JSONB,      -- [cat_id_1, cat_id_2]
    applicable_products JSONB,        -- [product_id_1, product_id_2]
    min_order_value     DECIMAL(15,2) DEFAULT 0,
    max_discount_amount DECIMAL(15,2),  -- Giới hạn số tiền giảm tối đa
    start_date          TIMESTAMP NOT NULL,
    end_date            TIMESTAMP NOT NULL,
    usage_limit         INT,          -- Giới hạn tổng số lần dùng
    usage_per_agency    INT DEFAULT 1, -- Giới hạn mỗi đại lý
    usage_count         INT DEFAULT 0,
    is_active           BOOLEAN DEFAULT TRUE,
    is_combinable       BOOLEAN DEFAULT FALSE,  -- Có kết hợp KM khác không
    created_by          BIGINT REFERENCES users(id),
    created_at          TIMESTAMP DEFAULT NOW(),
    updated_at          TIMESTAMP DEFAULT NOW()
);

-- ===================================================
-- BẢNG KHO HÀNG & TỒN KHO
-- ===================================================
CREATE TABLE warehouses (
    id          SERIAL PRIMARY KEY,
    name        VARCHAR(255) NOT NULL,
    address     TEXT,
    phone       VARCHAR(20),
    manager_id  BIGINT REFERENCES users(id),
    is_active   BOOLEAN DEFAULT TRUE,
    created_at  TIMESTAMP DEFAULT NOW()
);

CREATE TABLE warehouse_stock (
    id              BIGSERIAL PRIMARY KEY,
    warehouse_id    INT REFERENCES warehouses(id),
    product_id      BIGINT REFERENCES products(id),
    variant_id      BIGINT REFERENCES product_variants(id),
    quantity        INT NOT NULL DEFAULT 0,
    reserved_qty    INT DEFAULT 0,  -- Đã đặt, chưa xuất
    available_qty   INT GENERATED ALWAYS AS (quantity - reserved_qty) STORED,
    updated_at      TIMESTAMP DEFAULT NOW(),
    UNIQUE(warehouse_id, product_id, variant_id)
);

CREATE TABLE inventory_movements (
    id              BIGSERIAL PRIMARY KEY,
    warehouse_id    INT REFERENCES warehouses(id),
    product_id      BIGINT REFERENCES products(id),
    variant_id      BIGINT REFERENCES product_variants(id),
    movement_type   VARCHAR(20) NOT NULL,
    -- Types: stock_in, stock_out, adjustment, transfer, return
    quantity        INT NOT NULL,  -- Dương = nhập, Âm = xuất
    reference_type  VARCHAR(30),   -- 'order', 'purchase', 'adjustment'
    reference_id    BIGINT,
    unit_cost       DECIMAL(15,2),
    notes           TEXT,
    created_by      BIGINT REFERENCES users(id),
    created_at      TIMESTAMP DEFAULT NOW()
);

-- ===================================================
-- BẢNG THƯƠNG HIỆU
-- ===================================================
CREATE TABLE brands (
    id          SERIAL PRIMARY KEY,
    name        VARCHAR(255) NOT NULL,
    slug        VARCHAR(255) UNIQUE,
    logo_url    VARCHAR(500),
    description TEXT,
    is_active   BOOLEAN DEFAULT TRUE,
    created_at  TIMESTAMP DEFAULT NOW()
);

-- ===================================================
-- BẢNG BIẾN THỂ SẢN PHẨM
-- ===================================================
CREATE TABLE product_variants (
    id              BIGSERIAL PRIMARY KEY,
    product_id      BIGINT REFERENCES products(id) ON DELETE CASCADE,
    variant_name    VARCHAR(255) NOT NULL,  -- 'Đỏ - Size L'
    sku             VARCHAR(100) UNIQUE NOT NULL,
    barcode         VARCHAR(100),
    price_adjustment DECIMAL(15,2) DEFAULT 0, -- Cộng/trừ so với giá gốc
    weight          DECIMAL(10,3),
    stock_quantity  INT DEFAULT 0,
    image_url       VARCHAR(500),
    attributes      JSONB,  -- {"color": "Đỏ", "size": "L"}
    is_active       BOOLEAN DEFAULT TRUE,
    created_at      TIMESTAMP DEFAULT NOW()
);

-- ===================================================
-- BẢNG THÔNG BÁO
-- ===================================================
CREATE TABLE notifications (
    id          BIGSERIAL PRIMARY KEY,
    user_id     BIGINT REFERENCES users(id),
    title       VARCHAR(255) NOT NULL,
    message     TEXT NOT NULL,
    type        VARCHAR(50),  -- order, payment, promotion, system, stock
    link        VARCHAR(500),
    is_read     BOOLEAN DEFAULT FALSE,
    read_at     TIMESTAMP,
    created_at  TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_notifications_user ON notifications(user_id, is_read);

-- ===================================================
-- BẢNG LOG HOẠT ĐỘNG
-- ===================================================
CREATE TABLE activity_logs (
    id          BIGSERIAL PRIMARY KEY,
    user_id     BIGINT REFERENCES users(id),
    action      VARCHAR(100) NOT NULL,
    entity_type VARCHAR(50),   -- 'order', 'product', 'agency',...
    entity_id   BIGINT,
    old_values  JSONB,
    new_values  JSONB,
    ip_address  VARCHAR(45),
    user_agent  TEXT,
    created_at  TIMESTAMP DEFAULT NOW()
);
```

---

## 4. PHƯƠNG ÁN 1: Laravel + Filament + Vue.js + PostgreSQL

### 4.1 Tổng quan

| Tiêu chí | Chi tiết |
|-----------|----------|
| **Backend** | Laravel 11+ (PHP 8.3+) |
| **Admin Panel** | Filament PHP v4 |
| **Frontend Đại lý** | Vue.js 3 + Inertia.js |
| **Database** | PostgreSQL 16+ |
| **Cache / Queue** | Redis |
| **File Storage** | MinIO (S3-compatible) hoặc AWS S3 |
| **Search Engine** | Meilisearch (tìm kiếm sản phẩm nhanh) |
| **Realtime** | Laravel Reverb / Soketi (WebSocket) |

### 4.2 Tại sao chọn phương án này?

**Ưu điểm:**
1. **Laravel** là framework PHP phổ biến nhất (~35.87% thị phần), cộng đồng Việt Nam rất lớn, dễ tuyển dụng
2. **Filament v4** tiết kiệm 2-3 tháng phát triển admin panel — có sẵn CRUD, dashboard widgets, biểu đồ, phân quyền
3. **Eloquent ORM** xử lý quan hệ phức tạp (đại lý đa cấp, giá đa tầng) rất tốt
4. **Hệ sinh thái Laravel cực kỳ phong phú:**
   - `spatie/laravel-permission` — phân quyền RBAC
   - `maatwebsite/laravel-excel` — xuất báo cáo Excel
   - `barryvdh/laravel-dompdf` — xuất PDF (hóa đơn, bảng giá)
   - `intervention/image` — xử lý ảnh, tạo thumbnail
   - `laravel/scout` + Meilisearch — tìm kiếm sản phẩm
   - `laravel/sanctum` — API authentication
5. **Chi phí thấp:** Hosting PHP rẻ, chạy được trên VPS 2-4GB RAM
6. **Queue system** mạnh mẽ (Laravel Horizon) cho xử lý nền: tạo báo cáo, gửi email, tính toán công nợ

**Nhược điểm:**
1. PHP không mạnh cho xử lý AI/ML (nếu cần sau này)
2. Khó scale theo chiều ngang khi đạt hàng triệu request/ngày
3. Filament tùy biến giao diện phức tạp hơn so với tự code

### 4.3 Kiến trúc chi tiết

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENTS                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ Admin Portal │  │ Agency Portal│  │  Mobile App  │  │
│  │  (Filament)  │  │(Vue+Inertia) │  │  (Flutter)   │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
└─────────┼─────────────────┼─────────────────┼───────────┘
          │                 │                 │
          ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────┐
│                    NGINX + SSL                           │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│              LARAVEL APPLICATION                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Modules (Modular Monolith)                     │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────────┐    │    │
│  │  │ Auth &   │ │ Product  │ │   Pricing    │    │    │
│  │  │ RBAC     │ │ Catalog  │ │   Engine     │    │    │
│  │  └──────────┘ └──────────┘ └──────────────┘    │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────────┐    │    │
│  │  │ Agency   │ │  Order   │ │    Debt      │    │    │
│  │  │ Mgmt     │ │  Mgmt    │ │   Ledger     │    │    │
│  │  └──────────┘ └──────────┘ └──────────────┘    │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────────┐    │    │
│  │  │Promotion │ │Inventory │ │  Reporting   │    │    │
│  │  │ Engine   │ │  & Stock │ │  & Dashboard │    │    │
│  │  └──────────┘ └──────────┘ └──────────────┘    │    │
│  └─────────────────────────────────────────────────┘    │
└───────────┬──────────────┬──────────────┬───────────────┘
            │              │              │
     ┌──────▼──────┐ ┌────▼────┐  ┌──────▼──────┐
     │ PostgreSQL  │ │  Redis  │  │MinIO/S3     │
     │ (Database)  │ │ (Cache  │  │(Hình ảnh,   │
     │             │ │  Queue) │  │ Files)      │
     └─────────────┘ └─────────┘  └─────────────┘
```

### 4.4 Cấu trúc thư mục dự án

```
wholesale-platform/
├── app/
│   ├── Filament/                    # Admin Panel (Filament)
│   │   ├── Resources/
│   │   │   ├── ProductResource.php
│   │   │   ├── AgencyResource.php
│   │   │   ├── OrderResource.php
│   │   │   ├── PromotionResource.php
│   │   │   └── DebtLedgerResource.php
│   │   ├── Widgets/
│   │   │   ├── RevenueChart.php
│   │   │   ├── TopAgenciesWidget.php
│   │   │   ├── OrderStatsWidget.php
│   │   │   └── DebtOverviewWidget.php
│   │   └── Pages/
│   │       ├── Dashboard.php
│   │       └── Reports.php
│   ├── Models/
│   │   ├── User.php
│   │   ├── Agency.php
│   │   ├── PricingTier.php
│   │   ├── Product.php
│   │   ├── Category.php
│   │   ├── Order.php
│   │   ├── OrderItem.php
│   │   ├── DebtLedger.php
│   │   ├── Promotion.php
│   │   └── Warehouse.php
│   ├── Services/                    # Business Logic
│   │   ├── PricingService.php       # Tính giá theo cấp
│   │   ├── OrderService.php         # Xử lý đơn hàng
│   │   ├── DebtService.php          # Quản lý công nợ
│   │   ├── PromotionService.php     # Engine khuyến mãi
│   │   └── ReportService.php        # Tạo báo cáo
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Agency/              # Controllers cho portal đại lý
│   │   │   │   ├── DashboardController.php
│   │   │   │   ├── OrderController.php
│   │   │   │   ├── ProductController.php
│   │   │   │   └── DebtController.php
│   │   │   └── Api/                 # API cho mobile app
│   │   └── Middleware/
│   │       ├── CheckAgencyStatus.php
│   │       └── CheckCreditLimit.php
│   ├── Jobs/                        # Background Jobs
│   │   ├── GenerateMonthlyReport.php
│   │   ├── SendPaymentReminder.php
│   │   ├── CalculateAgencyRating.php
│   │   └── ProcessBulkPriceUpdate.php
│   └── Notifications/
│       ├── NewOrderNotification.php
│       ├── PaymentDueNotification.php
│       └── PromotionNotification.php
├── resources/
│   ├── js/                          # Vue.js (Inertia)
│   │   ├── Pages/
│   │   │   ├── Agency/
│   │   │   │   ├── Dashboard.vue
│   │   │   │   ├── Orders/
│   │   │   │   ├── Products/
│   │   │   │   └── Debt/
│   │   │   └── Auth/
│   │   ├── Components/
│   │   └── Layouts/
│   └── views/
│       └── pdf/                     # PDF templates
│           ├── invoice.blade.php
│           ├── price-list.blade.php
│           └── debt-statement.blade.php
├── database/
│   ├── migrations/
│   └── seeders/
├── config/
├── routes/
│   ├── web.php
│   ├── api.php
│   └── filament.php
└── tests/
```

### 4.5 Ví dụ code minh họa — Pricing Engine

```php
// app/Services/PricingService.php

class PricingService
{
    /**
     * Tính giá sản phẩm cho đại lý theo cấp + số lượng + khuyến mãi
     */
    public function calculatePrice(
        Product $product,
        Agency $agency,
        int $quantity
    ): PriceResult {
        // 1. Kiểm tra giá riêng cho đại lý (hợp đồng)
        $agencyPrice = ProductAgencyPrice::where('product_id', $product->id)
            ->where('agency_id', $agency->id)
            ->where('effective_from', '<=', now())
            ->where(fn($q) => $q->whereNull('effective_to')
                ->orWhere('effective_to', '>=', now()))
            ->where('min_quantity', '<=', $quantity)
            ->orderByDesc('min_quantity')
            ->first();

        if ($agencyPrice) {
            $unitPrice = $agencyPrice->unit_price;
        } else {
            // 2. Lấy giá theo cấp đại lý + số lượng
            $tierPrice = ProductTierPrice::where('product_id', $product->id)
                ->where('tier_id', $agency->tier_id)
                ->where('effective_from', '<=', now())
                ->where(fn($q) => $q->whereNull('effective_to')
                    ->orWhere('effective_to', '>=', now()))
                ->where('min_quantity', '<=', $quantity)
                ->orderByDesc('min_quantity')
                ->first();

            $unitPrice = $tierPrice?->unit_price ?? $product->base_price;
        }

        // 3. Áp dụng khuyến mãi (nếu có)
        $discount = $this->applyPromotions($product, $agency, $quantity, $unitPrice);

        return new PriceResult(
            unitPrice: $unitPrice,
            quantity: $quantity,
            subtotal: $unitPrice * $quantity,
            discountAmount: $discount,
            finalTotal: ($unitPrice * $quantity) - $discount,
        );
    }
}
```

### 4.6 Chi phí ước tính

| Hạng mục | Chi phí/tháng |
|----------|---------------|
| VPS (4 CPU, 8GB RAM, 200GB SSD) | $20-40/tháng |
| Domain + SSL | $15/năm |
| Email service (Mailgun/SES) | $10-30/tháng |
| MinIO (self-hosted) hoặc S3 | $5-20/tháng |
| SMS OTP (eSMS/Twilio) | $10-50/tháng |
| **Tổng hosting** | **~$50-150/tháng** |
| **Chi phí phát triển** (1 team) | **150-300 triệu VNĐ** |

---

## 5. PHƯƠNG ÁN 2: Next.js Full-Stack + Medusa.js + PostgreSQL

### 5.1 Tổng quan

| Tiêu chí | Chi tiết |
|-----------|----------|
| **Commerce Engine** | Medusa.js 2.0 (headless commerce, TypeScript) |
| **Frontend** | Next.js 15+ (React, Server Components) |
| **Admin Panel** | Medusa Admin (có sẵn) + custom pages |
| **Database** | PostgreSQL 16+ |
| **Cache** | Redis |
| **Language** | TypeScript toàn bộ (frontend + backend) |
| **File Storage** | AWS S3 / Cloudflare R2 |

### 5.2 Tại sao chọn phương án này?

**Ưu điểm:**
1. **Medusa.js** là nền tảng headless commerce mã nguồn mở phát triển nhanh nhất (30,970+ GitHub stars)
2. **TypeScript toàn bộ** — 1 ngôn ngữ cho cả frontend + backend, dễ maintain
3. **API-first architecture** — dễ dàng phát triển mobile app song song
4. **Admin panel có sẵn** — Medusa cung cấp dashboard admin đẹp, hiện đại
5. **Hỗ trợ B2B natively:** tiered pricing, bulk ordering, customer groups, invoice payments
6. **Event-driven architecture** — dễ mở rộng, tích hợp với hệ thống bên ngoài
7. **Hiệu năng cao** với Next.js Server Components + React Server Actions
8. **SEO tốt** nhờ Server-Side Rendering

**Nhược điểm:**
1. Medusa.js còn mới, cộng đồng nhỏ hơn Laravel
2. Cần developers có kinh nghiệm TypeScript/Node.js
3. Tuyển dụng dev Node.js tại Việt Nam khó hơn PHP
4. Tốn tài nguyên server hơn PHP (Node.js process)
5. Cần customize nhiều cho logic B2B đặc thù Việt Nam (công nợ, hóa đơn đỏ)

### 5.3 Kiến trúc chi tiết

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENTS                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ Medusa Admin │  │ Agency Store │  │  Mobile App  │  │
│  │  (built-in)  │  │  (Next.js)   │  │(React Native)│  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
└─────────┼─────────────────┼─────────────────┼───────────┘
          │                 │                 │
          ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────┐
│              MEDUSA.JS BACKEND (Node.js)                 │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Core Modules          Custom Modules           │    │
│  │  ┌──────────┐         ┌──────────────┐          │    │
│  │  │ Product  │         │ Agency Tier  │          │    │
│  │  │ Module   │         │ Pricing Mod  │          │    │
│  │  ├──────────┤         ├──────────────┤          │    │
│  │  │ Customer │         │ Debt Ledger  │          │    │
│  │  │ Module   │         │ Module       │          │    │
│  │  ├──────────┤         ├──────────────┤          │    │
│  │  │ Order    │         │ Promotion    │          │    │
│  │  │ Module   │         │ Extension    │          │    │
│  │  ├──────────┤         ├──────────────┤          │    │
│  │  │ Payment  │         │ Reporting    │          │    │
│  │  │ Module   │         │ Module       │          │    │
│  │  └──────────┘         └──────────────┘          │    │
│  └─────────────────────────────────────────────────┘    │
└───────────┬──────────────┬──────────────┬───────────────┘
            │              │              │
     ┌──────▼──────┐ ┌────▼────┐  ┌──────▼──────┐
     │ PostgreSQL  │ │  Redis  │  │   AWS S3    │
     └─────────────┘ └─────────┘  └─────────────┘
```

### 5.4 Ví dụ code — Custom Medusa Module cho Agency Pricing

```typescript
// src/modules/agency-pricing/service.ts

import { MedusaService } from "@medusajs/framework/utils"

class AgencyPricingService extends MedusaService({
  AgencyTier,
  ProductTierPrice,
}) {
  async calculatePriceForAgency(
    productId: string,
    agencyId: string,
    quantity: number
  ): Promise<PriceResult> {
    const agency = await this.agencyRepository.findOne({
      where: { id: agencyId },
      relations: ["tier"],
    })

    // Tìm giá theo cấp đại lý + số lượng
    const tierPrice = await this.productTierPriceRepository.findOne({
      where: {
        product_id: productId,
        tier_id: agency.tier_id,
        min_quantity: LessThanOrEqual(quantity),
        effective_from: LessThanOrEqual(new Date()),
      },
      order: { min_quantity: "DESC" },
    })

    const unitPrice = tierPrice?.unit_price ?? product.base_price
    const subtotal = unitPrice * quantity

    // Áp dụng khuyến mãi
    const discount = await this.promotionService.calculateDiscount(
      productId, agency, quantity, subtotal
    )

    return {
      unitPrice,
      quantity,
      subtotal,
      discountAmount: discount,
      finalTotal: subtotal - discount,
    }
  }
}
```

### 5.5 Chi phí ước tính

| Hạng mục | Chi phí/tháng |
|----------|---------------|
| VPS (4 CPU, 8GB RAM) — Node.js cần nhiều RAM hơn | $30-60/tháng |
| Vercel (Next.js frontend, Pro plan) | $20/tháng |
| AWS S3 + CloudFront CDN | $10-30/tháng |
| Domain + SSL | $15/năm |
| Email + SMS | $20-50/tháng |
| **Tổng hosting** | **~$80-170/tháng** |
| **Chi phí phát triển** | **200-400 triệu VNĐ** |

---

## 6. PHƯƠNG ÁN 3: Django + Django Oscar + React + PostgreSQL

### 6.1 Tổng quan

| Tiêu chí | Chi tiết |
|-----------|----------|
| **Backend** | Django 5+ (Python 3.12+) |
| **E-commerce Framework** | Django Oscar (modular e-commerce) |
| **Admin Panel** | Django Admin + Django Unfold (UI đẹp) |
| **Frontend Đại lý** | React 19 + Vite + Tailwind CSS |
| **API** | Django REST Framework (DRF) |
| **Database** | PostgreSQL 16+ |
| **Cache / Queue** | Redis + Celery |
| **Search** | Elasticsearch hoặc Meilisearch |
| **File Storage** | MinIO / AWS S3 |

### 6.2 Tại sao chọn phương án này?

**Ưu điểm:**
1. **Python** là ngôn ngữ phổ biến nhất thế giới — dễ tuyển dụng, dễ đào tạo
2. **Django Oscar** — framework e-commerce modular, thiết kế domain-driven, mọi model đều có thể override
3. **Tích hợp AI/ML tuyệt vời** — Python có TensorFlow, PyTorch, Pandas, scikit-learn
   - Dự báo nhu cầu (demand forecasting)
   - Đề xuất sản phẩm thông minh
   - Phân tích hành vi đại lý
   - Tối ưu giá tự động
4. **Django Admin** rất mạnh cho back-office, kết hợp Django Unfold có giao diện hiện đại
5. **Django REST Framework** — API mạnh mẽ, documentation tự động (Swagger)
6. **Bảo mật cao** — Django có các tính năng bảo mật built-in (CSRF, XSS, SQL injection protection)
7. **Xử lý dữ liệu lớn tốt** — Python + Pandas cho báo cáo phức tạp

**Nhược điểm:**
1. Django có learning curve cao hơn Laravel
2. Tốc độ xử lý request đơn lẻ chậm hơn PHP/Node.js
3. Cộng đồng Django tại Việt Nam nhỏ hơn Laravel
4. Django Oscar documentation không phong phú bằng
5. Hosting Python tốn tài nguyên hơn PHP

### 6.3 Kiến trúc chi tiết

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENTS                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ Admin Panel  │  │ Agency SPA   │  │  Mobile App  │  │
│  │(Django Admin │  │  (React +    │  │  (Flutter)   │  │
│  │ + Unfold)    │  │   Vite)      │  │              │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
└─────────┼─────────────────┼─────────────────┼───────────┘
          │                 │                 │
          ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────┐
│              DJANGO APPLICATION                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Django Oscar Core     Custom Apps              │    │
│  │  ┌──────────┐         ┌──────────────┐          │    │
│  │  │Catalogue │         │ agencies     │          │    │
│  │  │(Products)│         │ (Đại lý)     │          │    │
│  │  ├──────────┤         ├──────────────┤          │    │
│  │  │ Partner  │         │ tier_pricing │          │    │
│  │  │(Supplier)│         │ (Giá cấp)    │          │    │
│  │  ├──────────┤         ├──────────────┤          │    │
│  │  │ Order    │         │ debt_mgmt    │          │    │
│  │  │          │         │ (Công nợ)    │          │    │
│  │  ├──────────┤         ├──────────────┤          │    │
│  │  │ Offer    │         │ analytics    │          │    │
│  │  │(Promotn) │         │ (AI Reports) │          │    │
│  │  └──────────┘         └──────────────┘          │    │
│  │                                                  │    │
│  │  ┌──────────────────────────────────┐           │    │
│  │  │  Django REST Framework (API)     │           │    │
│  │  └──────────────────────────────────┘           │    │
│  │  ┌──────────────────────────────────┐           │    │
│  │  │  Celery (Background Tasks)       │           │    │
│  │  │  - Report generation             │           │    │
│  │  │  - AI/ML predictions             │           │    │
│  │  │  - Email/SMS notifications       │           │    │
│  │  └──────────────────────────────────┘           │    │
│  └─────────────────────────────────────────────────┘    │
└───────────┬──────────┬──────────┬───────────┬───────────┘
            │          │          │           │
     ┌──────▼───┐ ┌───▼────┐ ┌──▼───┐ ┌─────▼──────┐
     │PostgreSQL│ │ Redis  │ │MinIO │ │Elasticsearch│
     └──────────┘ └────────┘ └──────┘ └────────────┘
```

### 6.4 Ví dụ code — Custom Oscar Strategy cho Agency Pricing

```python
# apps/tier_pricing/strategy.py

from oscar.apps.partner.strategy import Default as DefaultStrategy
from apps.agencies.models import Agency, ProductTierPrice

class AgencyPricingStrategy(DefaultStrategy):
    """
    Custom pricing strategy cho đại lý đa cấp
    """

    def pricing_policy_for_product(self, product, agency: Agency = None):
        if not agency:
            return super().pricing_policy_for_product(product)

        # 1. Tìm giá theo cấp đại lý
        tier_price = ProductTierPrice.objects.filter(
            product=product,
            tier=agency.tier,
            effective_from__lte=timezone.now(),
        ).filter(
            Q(effective_to__isnull=True) | Q(effective_to__gte=timezone.now())
        ).order_by('-min_quantity').first()

        if tier_price:
            return FixedPrice(
                currency='VND',
                excl_tax=tier_price.unit_price,
                tax=Decimal('0'),
            )

        # Fallback: giá gốc
        return super().pricing_policy_for_product(product)


# apps/analytics/services.py — AI-Powered Analytics

import pandas as pd
from sklearn.ensemble import RandomForestRegressor

class DemandForecastingService:
    """
    Dự báo nhu cầu sản phẩm bằng Machine Learning
    """
    def predict_demand(self, product_id: int, days_ahead: int = 30):
        # Lấy dữ liệu lịch sử đơn hàng
        orders = OrderItem.objects.filter(
            product_id=product_id,
            order__status='completed',
            order__created_at__gte=timezone.now() - timedelta(days=365)
        ).values('order__created_at__date').annotate(
            daily_qty=Sum('quantity')
        ).order_by('order__created_at__date')

        df = pd.DataFrame(orders)
        # ... feature engineering & model training ...
        model = RandomForestRegressor()
        model.fit(X_train, y_train)
        prediction = model.predict(X_future)

        return {
            'product_id': product_id,
            'predicted_demand': prediction.tolist(),
            'confidence': model.score(X_test, y_test),
        }
```

### 6.5 Chi phí ước tính

| Hạng mục | Chi phí/tháng |
|----------|---------------|
| VPS (4 CPU, 16GB RAM) — Python + Celery + Elasticsearch | $40-80/tháng |
| Domain + SSL | $15/năm |
| Email + SMS | $20-50/tháng |
| Elasticsearch (nếu managed) | $30-50/tháng |
| **Tổng hosting** | **~$100-200/tháng** |
| **Chi phí phát triển** | **200-400 triệu VNĐ** |

---

## 7. SO SÁNH 3 PHƯƠNG ÁN

### 7.1 Bảng so sánh tổng hợp

| Tiêu chí | PA1: Laravel+Filament | PA2: Next.js+Medusa | PA3: Django+Oscar |
|----------|----------------------|--------------------|--------------------|
| **Độ khó phát triển** | Trung bình | Cao | Cao |
| **Thời gian MVP** | 4-5 tháng | 5-6 tháng | 5-7 tháng |
| **Chi phí phát triển** | 150-300 tr VNĐ | 200-400 tr VNĐ | 200-400 tr VNĐ |
| **Chi phí hosting/tháng** | $50-150 | $80-170 | $100-200 |
| **Tuyển dụng VN** | Dễ (PHP phổ biến) | Trung bình | Khó hơn |
| **Hiệu năng** | Tốt | Rất tốt | Tốt |
| **Khả năng mở rộng** | Tốt | Rất tốt | Tốt |
| **Mobile App** | API riêng | API sẵn | API (DRF) |
| **Admin Panel** | Filament (tuyệt vời) | Medusa Admin (tốt) | Django Admin (tốt) |
| **AI/ML tích hợp** | Yếu | Trung bình | Xuất sắc |
| **Cộng đồng VN** | Rất lớn | Nhỏ | Trung bình |
| **Bảo mật** | Tốt | Tốt | Rất tốt |
| **SEO** | Trung bình | Xuất sắc (SSR) | Tốt |
| **Realtime** | Laravel Reverb | Socket.io | Django Channels |
| **Báo cáo/Analytics** | Tốt | Tốt | Xuất sắc (Pandas) |

### 7.2 Đánh giá theo tính năng

| Tính năng | PA1 | PA2 | PA3 |
|-----------|-----|-----|-----|
| Đăng nhập/Phân quyền | 9/10 | 8/10 | 9/10 |
| Quản lý đại lý đa cấp | 9/10 | 7/10 | 8/10 |
| Quản lý sản phẩm | 9/10 | 9/10 | 9/10 |
| Giá theo cấp đại lý | 9/10 | 8/10 | 9/10 |
| Quản lý đơn hàng | 9/10 | 9/10 | 9/10 |
| Quản lý công nợ | 9/10 | 7/10 | 8/10 |
| Dashboard | 9/10 | 8/10 | 8/10 |
| Khuyến mãi | 8/10 | 8/10 | 9/10 |
| Thư viện hình ảnh | 9/10 | 8/10 | 7/10 |
| Báo cáo doanh thu | 8/10 | 7/10 | 9/10 |
| Đo lường đại lý | 8/10 | 7/10 | 9/10 |
| **Tổng điểm** | **96/110** | **86/110** | **94/110** |

### 7.3 Ma trận quyết định

```
                    Dễ triển khai
                         ▲
                         │
          PA1 ■          │
      (Laravel+Filament) │
                         │
    ─────────────────────┼─────────────────────► Nhiều tính năng
                         │                        nâng cao
                         │
              PA3 ■      │          PA2 ■
          (Django+Oscar) │      (Next.js+Medusa)
                         │
                         │
                    Phức tạp hơn
```

---

## 8. KIẾN TRÚC HỆ THỐNG CHUNG

### 8.1 Kiến trúc triển khai (Deployment Architecture)

```
                        Internet
                           │
                    ┌──────▼──────┐
                    │ Cloudflare  │  CDN + DDoS Protection
                    │   / DNS     │  + SSL Termination
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │   Nginx     │  Reverse Proxy
                    │  (Gateway)  │  + Load Balancing
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
       ┌──────▼──────┐    │     ┌──────▼──────┐
       │  Admin Web  │    │     │ Agency Web  │
       │  (Server)   │    │     │  (Server)   │
       └──────┬──────┘    │     └──────┬──────┘
              │            │            │
              └────────────┼────────────┘
                           │
                    ┌──────▼──────┐
                    │ Application │
                    │   Server    │
                    │ (2+ nodes)  │
                    └──────┬──────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
  ┌──────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐
  │ PostgreSQL  │  │    Redis    │  │   MinIO     │
  │  Primary    │  │  (Cluster)  │  │ (Storage)   │
  │  + Replica  │  │             │  │             │
  └─────────────┘  └─────────────┘  └─────────────┘
```

### 8.2 Giao diện người dùng (UI/UX Wireframe)

#### Admin Dashboard
```
┌─────────────────────────────────────────────────────────────┐
│  ☰  WHOLESALE ADMIN          🔔 3   👤 Admin ▼             │
├────────┬────────────────────────────────────────────────────┤
│        │                                                    │
│ 📊 Dashboard │  TỔNG QUAN HÔM NAY                          │
│ 🏢 Đại lý   │  ┌──────────┐ ┌──────────┐ ┌──────────┐    │
│ 📦 Sản phẩm │  │ Doanh thu │ │ Đơn hàng │ │ Công nợ  │    │
│ 🏷️ Bảng giá  │  │ 125.5 tr  │ │  47 đơn  │ │ 890 tr   │    │
│ 📋 Đơn hàng │  │  ↑ 12%    │ │  ↑ 8%    │ │  ↓ 5%    │    │
│ 💰 Công nợ  │  └──────────┘ └──────────┘ └──────────┘    │
│ 🎁 Khuyến mãi│                                             │
│ 📊 Báo cáo  │  ┌────────────────────────────────────────┐  │
│ 🖼️ Thư viện  │  │     BIỂU ĐỒ DOANH THU 12 THÁNG       │  │
│ ⚙️ Cài đặt   │  │     📈 ████████████████████████        │  │
│              │  │        ██████████████████████           │  │
│              │  └────────────────────────────────────────┘  │
│              │                                              │
│              │  ┌──────────────────┐ ┌──────────────────┐  │
│              │  │ TOP ĐẠI LÝ      │ │ TOP SẢN PHẨM    │  │
│              │  │ 1. Đại lý A 45tr│ │ 1. SP-001  1200  │  │
│              │  │ 2. Đại lý B 38tr│ │ 2. SP-042   980  │  │
│              │  │ 3. Đại lý C 32tr│ │ 3. SP-015   875  │  │
│              │  └──────────────────┘ └──────────────────┘  │
│              │                                              │
│              │  ĐƠN HÀNG GẦN ĐÂY                          │
│              │  ┌─────────┬────────┬─────────┬──────────┐  │
│              │  │ Mã ĐH   │Đại lý │Giá trị  │Trạng thái│  │
│              │  ├─────────┼────────┼─────────┼──────────┤  │
│              │  │ORD-0047 │DL A   │12.5 tr  │⏳ Chờ    │  │
│              │  │ORD-0046 │DL C   │ 8.2 tr  │✅ Xong   │  │
│              │  │ORD-0045 │DL B   │15.8 tr  │🚚 Giao   │  │
│              │  └─────────┴────────┴─────────┴──────────┘  │
└────────┴────────────────────────────────────────────────────┘
```

#### Agency Portal (Portal Đại lý)
```
┌─────────────────────────────────────────────────────────────┐
│  🏪 WHOLESALE PORTAL     Xin chào, Đại lý ABC  🔔 👤 ▼   │
├─────────────────────────────────────────────────────────────┤
│  Dashboard │ Sản phẩm │ Đặt hàng │ Đơn hàng │ Công nợ    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐  │
│  │  Công nợ   │ │Đơn đang   │ │ Hạn mức   │ │ Cấp đại lý│  │
│  │ 45.2 tr   │ │xử lý: 3   │ │còn: 54.8tr│ │  Cấp 1    │  │
│  └───────────┘ └───────────┘ └───────────┘ └───────────┘  │
│                                                             │
│  🔥 KHUYẾN MÃI ĐANG ÁP DỤNG                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 🎁 Mua 10 thùng SP-001 tặng 1 thùng | Đến 28/02   │   │
│  │ 💰 Giảm thêm 5% cho đơn hàng > 50 triệu           │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  SẢN PHẨM MỚI / NỔI BẬT                                   │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐              │
│  │ 🖼️     │ │ 🖼️     │ │ 🖼️     │ │ 🖼️     │              │
│  │ SP-001 │ │ SP-042 │ │ SP-015 │ │ SP-088 │              │
│  │125,000đ│ │ 89,000đ│ │210,000đ│ │ 45,000đ│              │
│  │[Đặt]   │ │[Đặt]   │ │[Đặt]   │ │[Đặt]   │              │
│  └────────┘ └────────┘ └────────┘ └────────┘              │
│                                                             │
│  ĐƠN HÀNG GẦN ĐÂY                                        │
│  ┌─────────┬───────────┬──────────┬──────────┬──────────┐  │
│  │ Mã ĐH   │ Ngày đặt  │ Giá trị  │Trạng thái│ Thao tác │  │
│  ├─────────┼───────────┼──────────┼──────────┼──────────┤  │
│  │ORD-0045 │13/02/2026 │ 15.8 tr  │🚚 Giao  │ Xem      │  │
│  │ORD-0039 │10/02/2026 │  8.5 tr  │✅ Xong   │ Đặt lại  │  │
│  └─────────┴───────────┴──────────┴──────────┴──────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 9. LỘ TRÌNH TRIỂN KHAI

### Phase 1: Nền tảng (Tuần 1-6)
- [x] Thiết kế CSDL & kiến trúc hệ thống
- [ ] Setup dự án, CI/CD pipeline
- [ ] Module xác thực & phân quyền (đăng nhập, RBAC)
- [ ] Module quản lý đại lý (CRUD, phân cấp, duyệt đại lý)
- [ ] Module quản lý sản phẩm (CRUD, danh mục đa cấp, thư viện ảnh)
- [ ] Module thương hiệu, đơn vị tính

### Phase 2: Nghiệp vụ cốt lõi (Tuần 7-12)
- [ ] Module giá theo cấp đại lý (pricing engine)
- [ ] Module đơn hàng (tạo, duyệt, trạng thái, in phiếu)
- [ ] Module công nợ (sổ nợ, ghi nhận thanh toán, hạn mức)
- [ ] Portal đặt hàng cho đại lý (giao diện đại lý)
- [ ] Thông báo đơn hàng (email, in-app)

### Phase 3: Quản lý tài chính (Tuần 13-16)
- [ ] Dashboard quản lý (biểu đồ doanh thu, top sản phẩm, top đại lý)
- [ ] Báo cáo doanh thu (theo đại lý, sản phẩm, thời gian)
- [ ] Báo cáo công nợ & tuổi nợ (aging report)
- [ ] Xuất báo cáo Excel/PDF
- [ ] Auto nhắc nhở thanh toán

### Phase 4: Nâng cao (Tuần 17-20)
- [ ] Module khuyến mãi (đa loại, theo cấp, theo thời gian)
- [ ] Quản lý kho (nhập/xuất/kiểm kê, đa kho)
- [ ] Đo lường & xếp hạng đại lý (A/B/C/D)
- [ ] Tìm kiếm nâng cao sản phẩm (full-text search)
- [ ] Tối ưu hiệu năng & bảo mật

### Phase 5: Mở rộng (Tuần 21+)
- [ ] Mobile app (Flutter / React Native)
- [ ] Tích hợp thanh toán trực tuyến (VNPay, MoMo)
- [ ] Tích hợp vận chuyển (GHN, GHTK, Viettel Post)
- [ ] AI/ML: Dự báo nhu cầu, đề xuất sản phẩm
- [ ] Multi-language, multi-currency (nếu mở rộng thị trường)

---

## 10. KẾT LUẬN & ĐỀ XUẤT

### Đề xuất chính: **PHƯƠNG ÁN 1 — Laravel + Filament + Vue.js + PostgreSQL**

**Lý do:**

1. **Phù hợp nhất với thị trường Việt Nam:**
   - Cộng đồng PHP/Laravel tại VN rất lớn → dễ tuyển dụng, dễ tìm hỗ trợ
   - Chi phí hosting thấp nhất trong 3 phương án
   - Nhiều developer VN có kinh nghiệm với Laravel

2. **Thời gian ra mắt nhanh nhất:**
   - Filament v4 tiết kiệm 2-3 tháng xây dựng admin panel
   - Laravel ecosystem phong phú, nhiều package sẵn có
   - MVP có thể hoàn thành trong 4-5 tháng

3. **Chi phí tổng thể thấp nhất:**
   - Phát triển: 150-300 triệu VNĐ
   - Hosting: chỉ $50-150/tháng
   - Bảo trì: dễ dàng, ít phức tạp

4. **Đáp ứng đầy đủ yêu cầu nghiệp vụ:**
   - Điểm tổng: 96/110 (cao nhất)
   - Đặc biệt mạnh ở: quản lý đại lý, công nợ, dashboard, thư viện hình ảnh

5. **Khả năng mở rộng:**
   - Bắt đầu modular monolith → tách microservices khi cần
   - API (Laravel Sanctum) sẵn sàng cho mobile app
   - Redis + Queue cho xử lý nền quy mô lớn

### Khi nào chọn phương án khác?

- **Chọn PA2 (Next.js + Medusa)** nếu: team chủ yếu là JavaScript developers, muốn hiệu năng frontend tối ưu, hoặc cần headless commerce cho nhiều kênh bán
- **Chọn PA3 (Django + Oscar)** nếu: có kế hoạch tích hợp AI/ML sâu (dự báo nhu cầu, tối ưu giá tự động), hoặc cần xử lý dữ liệu phân tích phức tạp

---

> **Ghi chú:** Tài liệu này là kết quả nghiên cứu tổng hợp. Bước tiếp theo là chọn phương án và bắt đầu triển khai Phase 1. Nếu cần, có thể tư vấn thêm về chi tiết kỹ thuật cho từng module cụ thể.
