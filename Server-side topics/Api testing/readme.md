# 📌 **Hướng Dẫn Kiểm Thử API (API Testing)**

## **1️⃣ Giới thiệu về API Testing**
### 🔹 **API là gì?**
API (**Application Programming Interface**) là một tập hợp các quy tắc cho phép các ứng dụng giao tiếp với nhau. Nó đóng vai trò trung gian giữa client (ứng dụng hoặc trình duyệt) và server (máy chủ chứa dữ liệu, logic).

Có nhiều loại API khác nhau, trong đó phổ biến nhất là:
- **RESTful API** (dùng HTTP requests như GET, POST, PUT, DELETE)
- **GraphQL API** (truy vấn dữ liệu linh hoạt hơn so với REST)
- **SOAP API** (dựa trên XML và thường dùng trong doanh nghiệp lớn)

### 🔹 **Tại sao cần kiểm thử API?**
Kiểm thử API đảm bảo API hoạt động đúng, an toàn và tối ưu, bao gồm:
- **Xác minh tính đúng đắn của dữ liệu**
- **Kiểm tra bảo mật** (tấn công Injection, lỗ hổng xác thực)
- **Xác định hiệu suất và giới hạn tốc độ (Rate limiting)**
- **Tìm kiếm các điểm yếu có thể khai thác**

---
## **2️⃣ Xác định điểm cuối API (API Endpoints)**

### 🔹 **1. Cách tìm endpoint API**
API endpoints là các URL nơi API nhận yêu cầu từ client. Ví dụ:
```http
GET /api/books HTTP/1.1
Host: example.com
```
👉 Điểm cuối API là `/api/books`, dùng để lấy danh sách sách.

📌 **Các cách tìm endpoint API:**
✅ **Quan sát URL có dấu hiệu API:** `/api/`, `/api/v1/`, `/api/users/`
✅ **Kiểm tra file JavaScript:** Các script frontend có thể tiết lộ các endpoint API.
✅ **Sử dụng Burp Suite Scanner:** Công cụ giúp phát hiện API endpoints.
✅ **Dùng wordlist brute-force:** Dò tìm endpoint API bằng danh sách từ phổ biến.

### 🔹 **2. Xác định các phương thức API hỗ trợ**
| **Phương thức** | **Chức năng** |
|------------|--------------|
| **GET** | Lấy dữ liệu |
| **POST** | Gửi dữ liệu mới |
| **PUT** | Cập nhật toàn bộ dữ liệu |
| **PATCH** | Cập nhật một phần dữ liệu |
| **DELETE** | Xóa dữ liệu |
| **OPTIONS** | Xem API hỗ trợ phương thức nào |

📌 **Dùng Burp Intruder để thử tất cả các phương thức tự động.**

---
## **3️⃣ Tài liệu API (API Documentation)**
### 🔹 **1. Các loại tài liệu API**
📌 **Tài liệu dành cho con người** (*Human-readable documentation*):
- Dễ hiểu, có ví dụ minh họa.
- Ví dụ: **Swagger UI, Postman Collection**.

📌 **Tài liệu dành cho máy** (*Machine-readable documentation*):
- Định dạng JSON, YAML giúp kiểm thử tự động.
- Ví dụ: **OpenAPI (Swagger), RAML, API Blueprint**.

### 🔹 **2. Cách tìm tài liệu API**
✅ **Tìm trong ứng dụng:** Kiểm tra Swagger, OpenAPI, Postman.
✅ **Dùng Burp Scanner:** Quét các URL phổ biến như `/api`, `/swagger/index.html`.
✅ **Sử dụng Intruder để brute-force các đường dẫn API documentation.**

---
## **4️⃣ Kiểm thử bảo mật API (API Security Testing)**
### 🔹 **1. Tìm kiếm tham số ẩn (Hidden Parameters)**
Một số tham số API có thể **không được ghi nhận trong tài liệu** nhưng vẫn tồn tại.

📌 **Cách tìm tham số ẩn:**
✅ **Dùng Burp Intruder để brute-force tham số** (danh sách từ phổ biến).
✅ **Dùng Param Miner (Burp Suite BApp)** để phát hiện tham số ẩn.
✅ **Kiểm tra phản hồi API để tìm tham số chưa được công bố.**

📌 **Ví dụ khai thác lỗ hổng Mass Assignment:**
```json
PATCH /api/users/123
{
    "username": "wiener",
    "isAdmin": true
}
```
💥 Nếu API chấp nhận, tài khoản người dùng có thể trở thành admin!

### 🔹 **2. Kiểm tra cơ chế xác thực API**
✅ **Dùng token cũ để thử Bypass Authentication.**
✅ **Kiểm tra thiếu giới hạn tốc độ (Rate Limiting).**
✅ **Kiểm tra lỗi xử lý JWT hoặc OAuth Token.**

### 🔹 **3. Xác định lỗi bảo mật phổ biến trong API**
| **Lỗ hổng** | **Mô tả** |
|------------|--------------|
| **BOLA (Broken Object Level Authorization)** | Truy cập tài nguyên trái phép |
| **Mass Assignment** | Gán tham số ẩn vào API |
| **Rate Limit Bypass** | Bỏ qua giới hạn tốc độ |
| **Excessive Data Exposure** | API tiết lộ dữ liệu nhạy cảm |

📌 **Sử dụng Burp Suite Scanner để tự động kiểm tra lỗ hổng API.**

---
## **5️⃣ Công cụ kiểm thử API phổ biến**
🔹 **Burp Suite** - Kiểm thử bảo mật API.
🔹 **Postman** - Tạo và gửi yêu cầu API.
🔹 **SoapUI** - Kiểm thử SOAP API và REST API.
🔹 **Fuzzing Tools (ffuf, wfuzz)** - Dò tìm endpoint API ẩn.
🔹 **JWT.io** - Kiểm tra JWT Token.

📌 **Kết hợp các công cụ trên để kiểm thử API hiệu quả!** 🚀

---
## **📌 Kết luận**
✅ **Tìm endpoint API bằng Burp Scanner và JavaScript Analysis.**
✅ **Kiểm tra API bằng cách thử nhiều phương thức HTTP.**
✅ **Dùng Intruder để tìm tham số ẩn và khai thác lỗ hổng.**
✅ **Sử dụng Postman, Burp Suite để kiểm thử bảo mật API.**

👉 **API Testing là một phần quan trọng trong bảo mật ứng dụng. Áp dụng kỹ thuật kiểm thử này để phát hiện và khắc phục lỗ hổng API!** 🔥


---

## 📌 Danh sách công cụ sử dụng
| **Tên công cụ** | **Vai trò** |
|----------------|------------|
| **Burp Scanner** | Quét API tự động để tìm lỗ hổng |
| **Burp Intruder** | Kiểm tra nhiều phương thức HTTP và brute-force tham số |
| **Burp OpenAPI Parser** | Trích xuất endpoints từ tài liệu OpenAPI JSON/YAML |
| **Param Miner** | Phát hiện tham số ẩn trong API |
| **Postman** | Gửi request API thủ công và kiểm tra phản hồi |
| **SoapUI** | Kiểm thử API nâng cao, đặc biệt cho SOAP API |
| **Swagger UI** | Trình bày tài liệu API thân thiện với người dùng |

---
