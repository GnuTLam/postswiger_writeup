# Server-side request forgery

## Lab: Basic SSRF against the local server
**Yêu cầu**
This lab has a stock check feature which fetches data from an internal system.
To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`. 

**Thực hiện**
- Quan sát request và response khi sử dụng chức năng `checkStock`.
![alt text](image.png)

- Tham số `stockApi` được truyền vào là một URL và địa chỉ này được truy cập thông qua mạng local.
- Ta có thể dự đoán tại đây có lỗ hổng SSRF. Và thực hiện kiểm thử thông qua `Burp Collaborator`.
![alt text](image-1.png)

- Xác nhận có lỗ hổng SSRF -> Thực hiện payload tấn công `http://localhost/admin`
![alt text](image-2.png)

- Để xóa `carlos` ta chỉnh payload lại thành `http://localhost/admin/delete?username=carlos`
![alt text](image-3.png)

**Notes**
Vấn đề bảo mật: SSRF (Server-Side Request Forgery) trong tính năng `Check stock`
Khi người dùng bấm `Check stock`, client gửi request chứa tham số `stockApi` (URL API nội bộ) lên server.
Nếu ứng dụng không xác thực hoặc giới hạn giá trị `stockApi`, kẻ tấn công có thể chỉnh sửa nó thành bất kỳ URL nào.
Điều này cho phép server gửi request đến các địa chỉ nội bộ (`localhost`, `127.0.0.1`), gây rò rỉ dữ liệu hoặc tấn công hệ thống nội bộ.

---

## Lab: Basic SSRF against another back-end system

**Yêu cầu**
This lab has a stock check feature which fetches data from an internal system.
To solve the lab, use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port `8080`, then use it to delete the user `carlos`.

**Thực hiện**
- Quan sát request và response khi sử dụng chức năng `checkStock`.
- Chúng ta biết rằng ở đây tồn tại lỗ hổng ssrf
- Theo yêu cầu chúng ta biết dải mạng 192.168.0/24 còn có tồn tại trang admin. Để tìm ra chúng ta có thể brute force.
![alt text](image-4.png)

- Thông qua `404` ta biết đây là một địa chỉ ip có thể kết nối tới được
![alt text](image-5.png)

- Thay đổi payload `stockApi=http%3A%2F%2F192.168.0.165%3A8080/admin` để có thể truy cập trang admin.
![alt text](image-6.png)

- Sử dụng payload `stockApi=http%3A%2F%2F192.168.0.165%3A8080/admin/delete?username=carlos` để xóa `carlos` và hoàn thành bài lab.

**Notes**
Bên cạnh việc tận dụng lỗ hổng SSRF truy cập vào server local, chúng ta cũng có thể truy cập vào các hệ thống back-end khác. 
Các hệ thống này thường không thể truy cập trực tiếp từ public server, để tấn công được các server này chúng ta cần biết chính xác domain hoặc địa chỉ IP của chúng. Có thể sử dụng phương pháp Brute force tìm kiếm trong dải địa chỉ IP phù hợp với mục tiêu tấn công.
SSRF là một lỗ hổng giúp kẻ tấn công có thể tìm kiếm nhiều các attack vector khác nữa.

---

## Lab: SSRF with blacklist-based input filter

**Yêu cầu**
This lab has a stock check feature which fetches data from an internal system.
To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.
The developer has deployed two weak anti-SSRF defenses that you will need to bypass.

**Thực hiện**
- Quan sát gói tin request và response của chức năng checkStock. Thử truy cập tới `localhost` hoặc `127.0.0.1`
![alt text](image-7.png)

- Cả hai trường hợp thì đều bị chặn. Dựa theo đề bài thì có lẽ ta nên tìm cách bypass cơ chế này. Đây là một đoạn mã phỏng đoán theo chức năng của bài lab.
```python
from flask import *
import requests

app = Flask(__name__)

@app.route('/ssrf')
def follow_url():
    url = request.args.get('url', '')
    blacklist = ['127.0.0.1', 'localhost']
    for check in blacklist:
        if check in urk:
            return "Attack SSRF detected!"
    return (requests.get(url).text)

if __name__ == '__main__':
    app.run(host = "0.0.0.0", port = 9999)

```

- Ta có thể thử sử dụng dạng rút gọn của địa chỉ ip. Payload `stockApi=http://127.0.1`
![alt text](image-8.png)

- Truy cập tiếp `/admin` tuy nhiên lại bị chặn ( có lẽ là do 'admin' cũng nằm trong black list của phía server). Thay thế bằng `/AdMiN` -> Payload `stockApi=http://127.0.1/AdMiN`
![alt text](image-9.png)

- Thực hiện nốt việc xóa `carlos`. Payload `stockApi=http://127.0.1/AdMiN/delete?username=carlos`

**Notes**
Ở bài này, có 1 kĩ thuật khác có thể được sử dụng để bypass là `double encoding`. 
- Double Encoding:  
  - Bộ lọc chỉ giải mã 1 lần trước khi kiểm tra.  
  - Kẻ tấn công mã hóa 2 lần để bypass kiểm tra, khi decode 2 lần, chuỗi trở thành giá trị nhạy cảm ban đầu.  
  - Ví dụ: /admin → %2fadmin → %252fadmin (bypass filter kiểm tra %2fadmin).  
Ngoài ra kĩ thuật Alternative IP được đề cập trong bài cũng khá hay.
- Alternative IP:  
  - IP có thể viết theo nhiều cách khác nhau, bypass bộ lọc kiểm tra chính xác.  
  - Ví dụ: 127.0.0.1 → 127.1 (hệ thống tự điền 0 còn thiếu).  
  - Hoặc: 127.0.0.1 → 2130706433 (dạng số nguyên).

---

## Lab: SSRF with whitelist-based input filter

**Yêu cầu**
This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.

The developer has deployed an anti-SSRF defense you will need to bypass.

**Thực hiện**
![alt text](image-29.png)

![alt text](image-30.png)

![alt text](image-31.png)

![alt text](image-32.png)

![alt text](image-33.png)

![alt text](image-34.png)

![alt text](image-35.png)

![alt text](image-36.png)

**Notes**

**Kí tự `@`**
URL chuẩn có format: `[protocol]://[username]:[password]@[hostname]...`
Ví dụ: `https://user:pass@hostname/path`
Nếu whitelist-check chỉ kiểm tra phần đầu của URL, nó có thể thấy:
`https://expected-host:fakepassword@evil-host`
và cho rằng hostname là expected-host (vì nó dừng kiểm tra sớm khi thấy expected-host:). Nhưng thực tế thì trước phần `@` đều là thông tin xác thực, còn hostname ở sau kí tự `@`

**Ký tự `#`**
Trong một URL, dấu # thường đánh dấu fragment (phần neo, anchor) phía client.
Ví dụ: `https://example.com/page#section2`
1 số bộ lọc không kiểm tra cẩn thận, chỉ kiểm tra xem có chuỗi `expected-host` trong url hay không -> tạo ra payload: `https://evil-host#expected-host`
Bộ lọc thấy `expected-host` trong chuỗi, nên cho qua.Nhưng trình duyệt/server có thể coi host chính là evil-host, còn #expected-host là fragment. Kết quả: request thực sự đi đến `evil-host`.

**Lợi dụng cấu trúc DNS**
DNS cho phép sử dụng dạng subdomain. Nếu whitelist đòi hỏi hostname phải chứa expected-host, ta có thể đăng ký (hoặc sử dụng) một domain có dạng: `expected-host.evil-host.com`. Ứng dụng thấy cụm expected-host trong domain, nên nó cho rằng hợp lệ. Nhưng DNS thực tế lại trỏ expected-host.evil-host.com đến server của kẻ tấn công.
Kết quả: request vẫn đến evil-host, chứ không phải expected-host.com

## Lab: Blind SSRF with out-of-band detection

**Yêu cầu**
This site uses analytics software which fetches the URL specified in the Referer header when a product page is loaded.

To solve the lab, use this functionality to cause an HTTP request to the public Burp Collaborator server.

**Thực hiện**
![alt text](image-39.png)

![alt text](image-40.png)

![alt text](image-41.png)

**Notes**
Web được cấu hình để thực hiện HTTP request đến URL ghi nhận trong header Referer để thu thập thông tin thống kê. Khi kẻ tấn công gửi request lên server, có thể tuỳ ý thay đổi giá trị Referer. Do cơ chế, server tự động truy cập URL lấy từ Referer. Hacker lợi dụng điều này để bắt server truy cập vào bất kỳ URL nội bộ nào. Trong trường hợp này, sẽ là blind ssrf vì phản hồi trực tiếp không hiển thị cho hacker, nhưng hacker có thể kiểm chứng qua out-of-band (OOB) – ví dụ như sử dụng Burp Collaborator hoặc Webhook để quan sát tương tác.

## Blind SSRF with Shellshock exploitation

**Notes**
Ở bài này, có 1 thứ cần lưu ý đó chính là `Shellshock`. Shellshock, còn được gọi là Bashdoor, là một lỗ hổng bảo mật trong Bash.

**Cơ chế hoạt động của Shellshock**
Trên các hệ điều hành tựa Unix và một số hệ điều hành khác sử dụng Bash để chạy các chương trình bằng dòng lệnh, mỗi chương trình khi chạy sẽ có một danh sách các cặp name/value được gọi là các biến môi trường. Khi một chương trình khởi chạy một chương trình khác nó sẽ cung cấp một danh sách khởi tạo các biến môi trường cho chương trình mới này. Mặt khác, Bash lưu giữ một danh sách các hàm chứa một tập các lệnh mà sẽ được thực thi cùng với chương trình.
Ta có thể đặt hàm vào biến môi trường. Ví dụ: `export myfunc='() { echo "Hello"; }'`
Khi Bash khởi tạo, nó sẽ đọc biến môi trường myfunc, nhận biết rằng nó chứa một định nghĩa hàm, và “chuyển” hàm đó vào trong shell hiện tại để có thể sử dụng.

**Lỗi ở Shellshock**
Lỗi xảy ra khi Bash không chỉ dừng lại ở phần định nghĩa hàm mà còn tiếp tục thực thi bất kỳ lệnh nào nằm phía sau thân hàm trong cùng biến môi trường đó. Ví dụ:

```bash
env x='() { :;}; echo vulnerable' bash -c "echo this is a test" 
```

Nó tạo ra một biến môi trường bắt đầu với chuỗi `() { :; }`, tạo ra một định nghĩa hàm rỗng. Theo sau là lệnh sẽ được thực thi khi biến môi trường này được export. Lệnh Bash được chạy sau đó sẽ thực hiện việc export biến môi trường này và lệnh echo sẽ được thực thi cùng với nó.
