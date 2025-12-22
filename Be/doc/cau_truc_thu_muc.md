# 📁 CẤU TRÚC THƯ MỤC BACKEND FASTAPI – BOOMKIDS

**Mô hình:** Controller – Service – Repository – Model

**Áp dụng cho:** Website bán quần áo trẻ em + Chat + Recommendation + AI

---

## 🏗️ CẤU TRÚC TỔNG QUÁT

```
backend/
│
├── alembic/
│   └── versions/
│
├── configs/
│   ├── database.py
│   ├── settings.py
│   └── jwt.py
│
├── controllers/
│   ├── auth_controller.py
│   ├── user_controller.py
│   ├── category_controller.py
│   ├── product_controller.py
│   ├── cart_controller.py
│   ├── order_controller.py
│   ├── review_controller.py
│   ├── voucher_controller.py
│   ├── chat_controller.py
│   └── recommend_controller.py
│
├── services/
│   ├── auth_service.py
│   ├── user_service.py
│   ├── product_service.py
│   ├── cart_service.py
│   ├── order_service.py
│   ├── chat_service.py
│   ├── recommend_service.py
│   └── ai_service.py
│
├── repositories/
│   ├── user_repo.py
│   ├── category_repo.py
│   ├── product_repo.py
│   ├── product_variant_repo.py
│   ├── cart_repo.py
│   ├── order_repo.py
│   ├── review_repo.py
│   ├── voucher_repo.py
│   ├── chat_repo.py
│   └── behavior_repo.py
│
├── models/
│   ├── base.py
│   ├── user.py
│   ├── category.py
│   ├── product.py
│   ├── product_variant.py
│   ├── product_image.py
│   ├── cart.py
│   ├── order.py
│   ├── review.py
│   ├── voucher.py
│   ├── conversation.py
│   ├── message.py
│   └── user_behavior.py
│
├── schemas/
│   ├── auth_schema.py
│   ├── user_schema.py
│   ├── category_schema.py
│   ├── product_schema.py
│   ├── cart_schema.py
│   ├── order_schema.py
│   ├── chat_schema.py
│   └── recommend_schema.py
│
├── middlewares/
│   ├── auth_middleware.py
│   └── logging_middleware.py
│
├── exceptions/
│   ├── base_exception.py
│   ├── auth_exception.py
│   └── business_exception.py
│
├── resources/
│   ├── prompts/
│   └── seed/
│
├── swagger/
│   └── openapi.py
│
├── tests/
│
├── utils/
│   ├── enums.py
│   ├── pagination.py
│   └── response.py
│
├── main.py
├── requirements.txt
└── README.md
```

---

## 🧠 GIẢI THÍCH NGẮN GỌN TỪNG THƯ MỤC (ĐỂ HỌC THUỘC)

### 🔹 `alembic/`

- Migration database
- Version hóa CSDL theo ERD

### 🔹 `configs/`

- Cấu hình hệ thống
- Database, JWT, biến môi trường

### 🔹 `controllers/`

- Định nghĩa API endpoint
- KHÔNG chứa logic nghiệp vụ

### 🔹 `services/` ⭐ (QUAN TRỌNG NHẤT)

- Xử lý nghiệp vụ
- Chat flow
- Recommendation logic
- Gọi AI

### 🔹 `repositories/`

- Giao tiếp database
- Query SQLAlchemy
- Không xử lý nghiệp vụ

### 🔹 `models/`

- SQLAlchemy models
- Ánh xạ 1–1 với ERD

### 🔹 `schemas/`

- Pydantic schemas
- Validate request / response

### 🔹 `middlewares/`

- JWT authentication
- Logging, request tracking

### 🔹 `exceptions/`

- Custom exception
- Thống nhất lỗi trả về

### 🔹 `resources/`

- Prompt AI
- File seed dữ liệu
- Static resource

### 🔹 `swagger/`

- Custom OpenAPI
- Nhóm & mô tả API

### 🔹 `utils/`

- Enum dùng chung
- Helper
- Response chuẩn

### 🔹 `tests/`

- Unit test
- Integration test

---

## 🎯 NGUYÊN TẮC VÀNG KHI CODE

✅ **Controller gọn** - chỉ nhận request và trả response

✅ **Logic nằm ở Service** - xử lý toàn bộ nghiệp vụ

✅ **DB nằm ở Repository** - query và thao tác database

✅ **Model bám ERD** - ánh xạ chính xác với thiết kế CSDL

✅ **Không viết logic trong `main.py`** - chỉ dùng để khởi tạo app

---

## 📊 LUỒNG XỬ LÝ REQUEST

```
Request → Controller → Service → Repository → Database
                          ↓
                      AI Service (nếu cần)
                          ↓
Response ← Controller ← Service ← Repository ← Database
```

---

## 💡 VÍ DỤ MINH HỌA

### Controller (chat_controller.py)

```python
@router.post("/chat/send")
async def send_message(request: ChatRequest):
    return await chat_service.send_message(request)
```

### Service (chat_service.py)

```python
async def send_message(request):
    # Logic nghiệp vụ
    conversation = await chat_repo.get_or_create_conversation(user_id)

    if conversation.type == "USER_BOT":
        response = await ai_service.generate_response(request.content)
    else:
        response = await notify_admin(request.content)

    await chat_repo.save_message(conversation.id, response)
    return response
```

### Repository (chat_repo.py)

```python
async def get_or_create_conversation(user_id):
    conversation = db.query(Conversation).filter_by(user_id=user_id).first()
    if not conversation:
        conversation = Conversation(user_id=user_id, type="USER_BOT")
        db.add(conversation)
        db.commit()
    return conversation
```

---

## 🚀 HƯỚNG DẪN SỬ DỤNG

1. **Clone và cài đặt**

```bash
pip install -r requirements.txt
```

2. **Chạy migration**

```bash
alembic upgrade head
```

3. **Khởi động server**

```bash
uvicorn main:app --reload
```

4. **Truy cập API docs**

```
http://localhost:8000/docs
```

---

## 📝 GHI CHÚ

- Tuân thủ nghiêm ngặt mô hình **Controller → Service → Repository**
- Mỗi module chỉ làm một nhiệm vụ rõ ràng
- Code dễ đọc, dễ bảo trì, dễ mở rộng
- Phù hợp cho đồ án tốt nghiệp và dự án thực tế
