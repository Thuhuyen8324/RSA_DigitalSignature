# Digital Signature and Chat Application (Ứng dụng Chữ ký số và Chat)

## Giới thiệu

Đây là một ứng dụng web được phát triển bằng Flask và Flask-SocketIO, tích hợp chức năng ký số file bằng RSA và một phòng chat để chia sẻ file đã ký. Mục tiêu của dự án là minh họa quá trình tạo khóa, ký số, xác minh chữ ký số và chia sẻ an toàn các file thông qua một giao diện web trực quan, đồng thời cung cấp một môi trường chat thời gian thực.

## Ý tưởng

Trong thế giới số hóa ngày nay, việc đảm bảo tính toàn vẹn (integrity) và xác thực (authenticity) của dữ liệu là vô cùng quan trọng. Chữ ký số là một công cụ mật mã mạnh mẽ giúp đạt được hai mục tiêu này.

Ý tưởng chính của ứng dụng bao gồm:

* **Minh họa Chữ ký số RSA:** Người dùng có thể tạo cặp khóa RSA (khóa riêng tư và khóa công khai), sử dụng khóa riêng tư để ký bất kỳ file nào, và sau đó dùng khóa công khai tương ứng để xác minh tính hợp lệ của chữ ký và file gốc.
* **Chia sẻ file an toàn:** Các file đã được ký và xác minh thành công có thể được chia sẻ trong một phòng chat chung, nơi mọi người có thể thấy thông tin về file và tải xuống. Điều này mô phỏng cách dữ liệu có thể được phân phối và xác thực trong một nhóm.
* **Giao tiếp thời gian thực:** Phòng chat được xây dựng bằng Flask-SocketIO, cho phép gửi và nhận tin nhắn tức thì, tạo cảm giác tương tác liền mạch giữa các người dùng.

## Tính năng chính

* **Tạo cặp khóa RSA:** Tự động tạo và lưu trữ khóa riêng tư (`private.pem`) và khóa công khai (`public.pem`).
* **Ký số file:** Tải lên một file và sử dụng khóa riêng tư đã tạo để tạo chữ ký số SHA256/PKCS1_15.
* **Xác minh chữ ký số:**
    * **Xác minh tự động:** Sau khi ký, hệ thống tự động kiểm tra chữ ký bằng khóa công khai.
    * **Xác minh độc lập:** Tải lên một file gốc và một file chữ ký riêng biệt để xác minh bằng khóa công khai hiện có.
* **Tải xuống file đã ký và chữ ký:** Người dùng có thể tải xuống file gốc và file `.sig` tương ứng.
* **Phòng Chat:**
    * Gửi và nhận tin nhắn theo thời gian thực.
    * Hiển thị thông báo khi file đã ký được xác minh và sẵn sàng chia sẻ trong phòng chat.
    * Người dùng có thể tải xuống các file được chia sẻ trực tiếp từ phòng chat.

## Quá trình hoạt động

1.  **Khởi động ứng dụng:**
    * Khi ứng dụng Flask được khởi động, nó sẽ lắng nghe trên cổng `5000` (hoặc cổng được chỉ định khác).
    * Terminal sẽ hiển thị các đường link truy cập vào trang chính và phòng chat cho cả truy cập nội bộ (localhost) và truy cập trong mạng LAN (địa chỉ IP cục bộ của bạn).

2.  **Tạo khóa RSA:**
    * Trên trang chính (`/`), người dùng có thể nhấp vào nút "Generate Keys" để tạo một cặp khóa RSA (khóa riêng tư và khóa công khai). Các khóa này sẽ được lưu trữ trong thư mục `signed_files`.

3.  **Ký số file:**
    * Người dùng chọn một file từ máy tính của mình và nhấp vào "Upload and Sign".
    * Ứng dụng đọc nội dung file, tạo hàm băm SHA256 cho nó.
    * Sử dụng khóa riêng tư đã tạo, ứng dụng ký vào hàm băm này để tạo ra chữ ký số.
    * Chữ ký số (`.sig` file) và file gốc được lưu vào thư mục `signed_files` trên server.
    * Hệ thống tự động xác minh chữ ký vừa tạo bằng khóa công khai và hiển thị kết quả "Chữ ký hợp lệ" hoặc "Chữ ký KHÔNG hợp lệ".

4.  **Xác minh độc lập:**
    * Người dùng có thể tải lên một file gốc bất kỳ và một file chữ ký `.sig` tương ứng.
    * Ứng dụng sẽ sử dụng khóa công khai đã lưu để xác minh xem chữ ký có hợp lệ với file gốc hay không.
    * Kết quả xác minh (hợp lệ/không hợp lệ) sẽ được hiển thị trên giao diện.
    * Nếu xác minh thành công, một thông báo sẽ được gửi đến phòng chat thông báo rằng file và chữ ký đã sẵn sàng để chia sẻ. Trình duyệt cũng tự động chuyển hướng đến phòng chat.

5.  **Phòng Chat:**
    * Khi người dùng truy cập vào `/chat`, họ sẽ tham gia vào phòng chat chung.
    * Các tin nhắn được gửi từ client sẽ được chuyển tiếp đến tất cả các client khác trong phòng chat thông qua SocketIO.
    * Khi một file được xác minh thành công từ trang chính, thông tin về file (tên và URL tải xuống) sẽ được gửi đến phòng chat dưới dạng sự kiện `file_shared`, cho phép người dùng trong phòng chat tải về file đã ký và chữ ký của nó.

## Cài đặt và Chạy ứng dụng

### Các bước cài đặt



4.  **Cài đặt các thư viện cần thiết:**
    ```bash
    pip install -r requirements.txt
    ```
    Nếu bạn chưa có file `requirements.txt`, bạn có thể tạo nó bằng cách chạy các lệnh sau (sau khi đã kích hoạt môi trường ảo):
    ```bash
    pip install Flask Flask-SocketIO pycryptodome python-dotenv Werkzeug
    pip freeze > requirements.txt
    ```
    *Lưu ý: `python-dotenv` và `Werkzeug` đã được bao gồm nếu code của bạn có sử dụng, đảm bảo tất cả các phụ thuộc được ghi lại.*

### Chạy ứng dụng

1.  Chạy file `digital_signature.py`:
    ```bash
    python digital_signature.py
    ```
2.  Khi server khởi động, bạn sẽ thấy các đường link trong terminal tương tự như sau:
    ```
    Ứng dụng đang chạy trên các địa chỉ sau:
    ----------------------------------------------------
    1. Trang chính  [http://127.0.0.1:5000/](http://127.0.0.1:5000/)
    2. Phòng Chat  [http://127.0.0.1:5000/chat](http://127.0.0.1:5000/chat)
   
    ----------------------------------------------------
    Chú ý: Giữ Ctrl và click vào link trong terminal để mở.
    ```
    *(Thay `<YOUR_LOCAL_IP>` bằng địa chỉ IP cục bộ thực tế của máy tính bạn, ví dụ: `192.168.1.20`.)*
3.  **Để truy cập giao diện:**
    * **Trên máy tính của bạn:** Giữ `Ctrl` (hoặc `Cmd` trên macOS) và nhấp vào `http://127.0.0.1:5000/` để mở trang chính, hoặc `http://127.0.0.1:5000/chat` để mở phòng chat.
   <h2 align="center">📸 Kết quả hiển thị</h2>

<table align="center">
  <tr>
    <td align="center">
      <img src="main.jpg" alt="màn hình điền thông tin" width="30%"><br>
      <strong>Màn hình nhập thông tin</strong>
    </td>
    <td align="center">
      <img src="ketqua.jpg" alt="Kết quả tính toán" width="30%"><br>
      <strong>Màn hình kết quả tính toán</strong>
    </td>
  </tr>
</table>

