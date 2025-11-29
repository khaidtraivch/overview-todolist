# TỔNG QUAN DỰ ÁN TODO LIST / NOTES APPLICATION

## 1. GIỚI THIỆU DỰ ÁN

Dự án là một ứng dụng web quản lý ghi chú (Notes/Todo List) được xây dựng bằng Flask framework. Ứng dụng cho phép người dùng đăng ký, đăng nhập, tạo và quản lý các ghi chú cá nhân của mình.

## 2. CÔNG NGHỆ SỬ DỤNG

### 2.1. Backend Framework
- **Flask 3.1.1**: Framework web nhẹ và linh hoạt của Python
- **Flask-Login 0.6.3**: Quản lý phiên đăng nhập và xác thực người dùng
- **Flask-SQLAlchemy 3.1.1**: ORM (Object-Relational Mapping) để tương tác với database
- **Flask-Migrate 4.1.0**: Quản lý migrations cho database schema

### 2.2. Database
- **SQLite**: Database nhẹ, phù hợp cho ứng dụng nhỏ và vừa
- **SQLAlchemy 2.0.41**: ORM layer để thao tác với database một cách an toàn

### 2.3. Frontend
- **HTML5**: Cấu trúc trang web
- **Bootstrap 4**: Framework CSS để tạo giao diện responsive và đẹp mắt
- **JavaScript (Vanilla)**: Xử lý tương tác phía client (AJAX requests)
- **Jinja2**: Template engine của Flask để render HTML động

### 2.4. Bảo mật
- **Werkzeug**: Hashing mật khẩu bằng thuật toán pbkdf2:sha256
- **Session Management**: Quản lý phiên đăng nhập với thời gian hết hạn
- **Flask-Login**: Bảo vệ các route yêu cầu đăng nhập

### 2.5. Công cụ hỗ trợ
- **python-dotenv**: Quản lý biến môi trường (SECRET_KEY, DB_NAME)
- **Alembic**: Database migration tool

## 3. KIẾN TRÚC HỆ THỐNG

### 3.1. Cấu trúc thư mục
```
do an 2/
├── app.py                 # Entry point của ứng dụng
├── requirements.txt       # Danh sách dependencies
├── data/                  # Thư mục chứa database
│   └── todolist.db
├── todolist/              # Package chính
│   ├── __init__.py        # Khởi tạo Flask app và cấu hình
│   ├── models.py          # Database models (User, Note)
│   ├── user.py            # Blueprint xử lý authentication
│   ├── views.py           # Blueprint xử lý notes
│   ├── static/
│   │   └── index.js       # JavaScript cho xử lý AJAX
│   └── templates/         # HTML templates
│       ├── base.html      # Template cơ sở
│       ├── login.html     # Trang đăng nhập
│       ├── signup.html    # Trang đăng ký
│       ├── index.html     # Trang chủ (hiển thị notes)
│       └── change_password.html  # Trang đổi mật khẩu
```

### 3.2. Mô hình kiến trúc
Dự án sử dụng mô hình **MVC (Model-View-Controller)**:

- **Model** (`models.py`): Định nghĩa cấu trúc dữ liệu
  - `User`: Model người dùng (id, email, password, user_name)
  - `Note`: Model ghi chú (id, data, date, user_id)

- **View** (`templates/`): Giao diện người dùng
  - Templates HTML với Jinja2
  - JavaScript cho tương tác động

- **Controller** (`user.py`, `views.py`): Xử lý logic nghiệp vụ
  - Blueprint pattern để tổ chức routes
  - Xử lý request/response
  - Tương tác với database

### 3.3. Blueprint Pattern
Dự án sử dụng Flask Blueprint để tổ chức code:
- **`user` Blueprint**: Quản lý authentication (login, signup, logout, change password)
- **`views` Blueprint**: Quản lý notes (home, delete note)

## 4. CHỨC NĂNG CHÍNH

### 4.1. Quản lý người dùng (Authentication)

#### 4.1.1. Đăng ký tài khoản (`/signup`)
- Người dùng nhập: email, user_name, password, confirm_password
- Validation:
  - Email phải > 3 ký tự
  - Password phải >= 7 ký tự
  - Password và confirm_password phải khớp
  - Email không được trùng với tài khoản đã tồn tại
- Mật khẩu được hash bằng `generate_password_hash()` trước khi lưu
- Sau khi đăng ký thành công, tự động đăng nhập người dùng

#### 4.1.2. Đăng nhập (`/login`)
- Xác thực email và password
- Sử dụng `check_password_hash()` để so sánh mật khẩu
- Session được set là permanent với thời gian hết hạn
- Sử dụng `remember=True` để lưu trạng thái đăng nhập

#### 4.1.3. Đăng xuất (`/logout`)
- Xóa session đăng nhập
- Chuyển hướng về trang login

#### 4.1.4. Đổi mật khẩu (`/change-password`)
- Yêu cầu đăng nhập (`@login_required`)
- Xác thực mật khẩu hiện tại
- Validation mật khẩu mới (>= 7 ký tự)
- Cập nhật mật khẩu mới vào database

#### 4.1.5. Xem thông tin profile (`/profile`)
- Trả về JSON với thông tin username và email của user hiện tại

### 4.2. Quản lý Notes

#### 4.2.1. Hiển thị danh sách notes (`/home`, `/`)
- Yêu cầu đăng nhập (`@login_required`)
- Hiển thị tất cả notes của user hiện tại
- Mỗi note hiển thị nội dung và nút xóa

#### 4.2.2. Tạo note mới
- Form POST với textarea để nhập nội dung
- Validation: note phải có ít nhất 1 ký tự
- Lưu note vào database với:
  - `data`: Nội dung note
  - `user_id`: ID của user tạo note
  - `date`: Thời gian tạo (tự động)

#### 4.2.3. Xóa note (`/delete-note`)
- Sử dụng AJAX (POST request với JSON)
- Kiểm tra quyền: chỉ cho phép xóa note của chính user
- Xóa note khỏi database
- Trả về JSON response với code 200

## 5. CÁCH TRIỂN KHAI

### 5.1. Khởi tạo ứng dụng

#### Bước 1: Cài đặt môi trường
```bash
# Tạo virtual environment
python -m venv venv

# Kích hoạt virtual environment
# Windows:
venv\Scripts\activate
# Linux/Mac:
source venv/bin/activate

# Cài đặt dependencies
pip install -r requirements.txt
```

#### Bước 2: Cấu hình môi trường
Tạo file `.env` với nội dung:
```
KEY=your-secret-key-here
DB_NAME=todolist.db
```

#### Bước 3: Chạy ứng dụng
```bash
python app.py
```
Ứng dụng sẽ chạy tại `http://0.0.0.0:5000` (hoặc `http://localhost:5000`)

### 5.2. Quy trình khởi tạo database

1. **Tạo database tự động**: 
   - Hàm `create_database()` trong `__init__.py` kiểm tra và tạo database nếu chưa tồn tại
   - Database được lưu tại `data/todolist.db`

2. **Tạo tables**:
   - Sử dụng `db.create_all()` để tạo các bảng User và Note
   - Dựa trên models đã định nghĩa

### 5.3. Luồng xử lý request

#### Luồng đăng nhập:
1. User truy cập `/login`
2. Nhập email và password
3. Server query User từ database theo email
4. So sánh password hash
5. Nếu đúng: tạo session, đăng nhập, redirect về `/home`
6. Nếu sai: hiển thị thông báo lỗi

#### Luồng tạo note:
1. User đã đăng nhập truy cập `/home`
2. Nhập nội dung note vào form
3. Submit form (POST request)
4. Server validate nội dung
5. Tạo Note object với `user_id = current_user.id`
6. Lưu vào database
7. Redirect và hiển thị note mới

#### Luồng xóa note:
1. User click nút xóa trên note
2. JavaScript gọi hàm `deleteNote(noteId)`
3. Gửi AJAX POST request đến `/delete-note` với JSON `{note_id: id}`
4. Server kiểm tra quyền sở hữu
5. Xóa note khỏi database
6. Trả về JSON response
7. JavaScript reload trang để cập nhật danh sách

### 5.4. Bảo mật

1. **Password Hashing**:
   - Sử dụng `generate_password_hash()` với thuật toán pbkdf2:sha256
   - Mật khẩu không bao giờ được lưu dạng plain text

2. **Session Management**:
   - Session được set là permanent
   - Thời gian hết hạn: 1 phút (có thể điều chỉnh)
   - Sử dụng SECRET_KEY để mã hóa session

3. **Route Protection**:
   - Decorator `@login_required` bảo vệ các route yêu cầu đăng nhập
   - Tự động redirect về `/login` nếu chưa đăng nhập

4. **Authorization**:
   - Kiểm tra `user_id` khi xóa note để đảm bảo user chỉ xóa được note của mình

## 6. DATABASE SCHEMA

### 6.1. Bảng User
```sql
CREATE TABLE user (
    id INTEGER PRIMARY KEY,
    email VARCHAR(150) UNIQUE,
    password VARCHAR(150),
    user_name VARCHAR(150)
);
```

**Quan hệ**: Một User có nhiều Notes (one-to-many)

### 6.2. Bảng Note
```sql
CREATE TABLE note (
    id INTEGER PRIMARY KEY,
    data VARCHAR(10000),
    date DATETIME,
    user_id INTEGER,
    FOREIGN KEY (user_id) REFERENCES user(id)
);
```

**Quan hệ**: Một Note thuộc về một User (many-to-one)

## 7. GIAO DIỆN NGƯỜI DÙNG

### 7.1. Navigation Bar
- **Khi chưa đăng nhập**: Hiển thị "Login" và "Sign Up"
- **Khi đã đăng nhập**: Hiển thị "Home", "Change Password", "Logout"

### 7.2. Flash Messages
- Hiển thị thông báo thành công (màu xanh) hoặc lỗi (màu đỏ)
- Tự động ẩn khi click nút đóng

### 7.3. Responsive Design
- Sử dụng Bootstrap 4 để đảm bảo giao diện responsive trên mọi thiết bị

## 8. ĐIỂM MẠNH CỦA DỰ ÁN

1. **Kiến trúc rõ ràng**: Sử dụng Blueprint pattern, tách biệt concerns
2. **Bảo mật tốt**: Hash password, session management, route protection
3. **Code tổ chức tốt**: MVC pattern, separation of concerns
4. **Dễ mở rộng**: Có thể thêm features mới dễ dàng
5. **Database relationships**: Sử dụng foreign key để đảm bảo tính toàn vẹn dữ liệu

## 9. HẠN CHẾ VÀ HƯỚNG PHÁT TRIỂN

### 9.1. Hạn chế hiện tại
- Session timeout quá ngắn (1 phút)
- Chưa có chức năng chỉnh sửa note
- Chưa có phân loại/category cho notes
- Chưa có tìm kiếm notes
- Chưa có pagination cho danh sách notes dài

### 9.2. Hướng phát triển
- Thêm chức năng edit note
- Thêm tags/categories cho notes
- Thêm tìm kiếm và filter notes
- Thêm pagination
- Thêm chức năng share note
- Nâng cấp lên PostgreSQL cho production
- Thêm API RESTful
- Thêm unit tests và integration tests
- Thêm Docker containerization

## 10. KẾT LUẬN

Dự án là một ứng dụng web quản lý ghi chú hoàn chỉnh với đầy đủ các chức năng cơ bản: đăng ký, đăng nhập, tạo và xóa notes. Ứng dụng được xây dựng với Flask framework, sử dụng SQLite database và có giao diện thân thiện với Bootstrap. Code được tổ chức tốt theo mô hình MVC và Blueprint pattern, dễ bảo trì và mở rộng.

