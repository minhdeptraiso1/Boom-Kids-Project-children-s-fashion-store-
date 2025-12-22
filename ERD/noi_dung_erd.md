# 📘 GIẢI THÍCH CHI TIẾT ERD

**Hệ thống Website Quần áo Trẻ em Boomkids**

---

## I. TỔNG QUAN THIẾT KẾ ERD

Hệ thống được thiết kế theo mô hình thương mại điện tử hoàn chỉnh, bao gồm:

- Quản lý người dùng
- Quản lý sản phẩm & tồn kho
- Giỏ hàng & đơn hàng
- Chat – đánh giá – khuyến mãi
- Recommendation System (mức 2 – dựa trên hành vi)

👉 **CSDL được chuẩn hóa 1NF – 3NF**, vừa đúng lý thuyết, vừa áp dụng thực tế.

---

## II. PHÂN HỆ NGƯỜI DÙNG (User Management)

### 🔹 Bảng `users`

**Chức năng:**  
Lưu thông tin tài khoản của người dùng trong hệ thống.

**Thông tin lưu trữ:**

- `id`: định danh duy nhất của người dùng
- `name`: họ tên
- `email`, `password`: thông tin đăng nhập
- `phone`: số điện thoại liên hệ
- `role`: phân quyền (ADMIN, CUSTOMER)
- `created_at`, `updated_at`: theo dõi thời gian

**Vai trò trong hệ thống:**

- **CUSTOMER**: mua hàng, chat, đánh giá
- **ADMIN**: quản lý sản phẩm, đơn hàng, trả lời chat

> 📌 **Ghi nhớ:**  
> Mọi nghiệp vụ đều liên kết gián tiếp hoặc trực tiếp tới bảng `users`.

---

## III. PHÂN HỆ SẢN PHẨM (Product System)

### 🔹 Bảng `categories`

**Chức năng:**  
Quản lý danh mục sản phẩm theo cấu trúc cây (cha – con).

**Thông tin:**

- `parent_id`: cho phép tạo danh mục con (VD: Bé gái → Váy)

👉 Giúp lọc và hiển thị sản phẩm dễ dàng.

---

### 🔹 Bảng `products`

**Chức năng:**  
Lưu thông tin chung của sản phẩm, không phụ thuộc size/màu.

**Thông tin quan trọng:**

- `original_price`: giá gốc
- `discount_percent`: % giảm giá
- `current_price`: giá bán hiện tại
- `gender`: BOY / GIRL / UNISEX
- `status`: đang bán hay ngưng bán

> 📌 **Thiết kế này giúp:**
> - Hiển thị khuyến mãi
> - Tối ưu truy vấn
> - Phù hợp nghiệp vụ thực tế

---

### 🔹 Bảng `product_images`

**Chức năng:**

- Lưu nhiều ảnh cho 1 sản phẩm
- `is_main` xác định ảnh đại diện

---

### 🔹 Bảng `sizes` & `colors`

**Chức năng:**

- Chuẩn hóa dữ liệu size và màu
- Tránh nhập trùng lặp
- Dễ mở rộng

---

### 🔹 Bảng `product_variants` (CỰC KỲ QUAN TRỌNG)

**Chức năng:**

- Quản lý biến thể sản phẩm
- Lưu tồn kho thực tế

**Thông tin:**

- `size_id`, `color_id`: xác định biến thể
- `quantity`: số lượng còn
- `sku`: mã kho duy nhất

> 📌 **Ghi nhớ kỹ:**  
> Tồn kho KHÔNG nằm trong `products` mà nằm trong `product_variants`

---

## IV. GIỎ HÀNG & BÁN HÀNG (Shopping & Sales)

### 🔹 Bảng `carts`

- Mỗi user chỉ có 1 giỏ hàng
- Lưu trạng thái giỏ trong quá trình mua

---

### 🔹 Bảng `cart_items`

- Mỗi dòng = 1 biến thể sản phẩm
- Liên kết tới `product_variants`
- Lưu số lượng người dùng chọn

---

### 🔹 Bảng `orders`

**Chức năng:**  
Lưu đơn hàng chính

**Thông tin:**

- `status`: trạng thái xử lý đơn
- `payment_status`: trạng thái thanh toán
- `payment_method`: COD, Banking, MoMo
- `shipping_address`, `customer_note`

---

### 🔹 Bảng `order_items`

- Chi tiết từng sản phẩm trong đơn
- `price_at_purchase`: giá tại thời điểm mua  
  → Không bị ảnh hưởng nếu giá sau này thay đổi

---

## V. CHAT SYSTEM (User – Admin – Chatbot)

### 🔹 Bảng `conversations`

- Quản lý phiên chat
- Phân biệt:
  - User ↔ Admin
  - User ↔ Chatbot

---

### 🔹 Bảng `messages`

- Lưu nội dung từng tin nhắn
- `sender_type`: USER / ADMIN / BOT
- `sender_id`:
  - Có giá trị nếu là USER/ADMIN
  - NULL nếu là BOT

> 📌 **Ưu điểm thiết kế:**
> - Dùng chung cho chat admin & chatbot
> - Dễ mở rộng WebSocket / AI

---

## VI. ĐÁNH GIÁ – KHUYẾN MÃI – RECOMMEND

### 🔹 Bảng `reviews`

**Chức năng:**  
Đánh giá sản phẩm sau khi mua

**Điểm mạnh:**

- Có `order_item_id` → chỉ đánh giá khi đã mua

---

### 🔹 Bảng `vouchers` & `order_vouchers`

- Voucher có thể áp dụng cho nhiều đơn
- Một đơn có thể dùng nhiều voucher

👉 Quan hệ N–N, thiết kế đúng chuẩn.

---

### 🔹 Bảng `user_behaviors` (RECOMMEND LEVEL 2)

**Chức năng:**  
Ghi nhận hành vi người dùng:

- VIEW
- ADD_TO_CART
- PURCHASE

**Vai trò:**

- Phân tích sở thích
- Gợi ý sản phẩm phù hợp
- Không cần Machine Learning

> 📌 **Câu ghi nhớ:**  
> Recommendation system dựa trên hành vi giúp cá nhân hóa trải nghiệm người dùng.

---

## VII. TỔNG KẾT ĐỂ EM DỄ NHỚ

| Nhóm | Vai trò |
|------|---------|
| `users` | trung tâm hệ thống |
| `products` | thông tin sản phẩm |
| `product_variants` | tồn kho |
| `orders` | bán hàng |
| `conversations/messages` | hỗ trợ khách |
| `user_behaviors` | gợi ý sản phẩm |

---

**© 2024 Boomkids System Documentation**