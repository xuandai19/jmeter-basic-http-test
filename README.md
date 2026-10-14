# Báo cáo thực hành Apache JMeter – Kiểm thử một trang web và kiểm thử tải

> **Sinh viên:** [Điền họ tên]  
> **Mã sinh viên / Lớp:** [Điền thông tin]  
> **Môn học:** Kiểm thử phần mềm  
> **Ngày thực hiện:** [Điền ngày]

## 1. Mục tiêu

- Sử dụng Apache JMeter để gửi HTTP request tới **một trang/endpoint cụ thể**.
- Kiểm tra chức năng phản hồi của trang bằng mã trạng thái HTTP và Response Assertion.
- Hiểu sự khác nhau giữa kiểm tra nhẹ (smoke test) và kiểm thử tải (load test).
- Đọc các chỉ số như số mẫu, thời gian phản hồi trung bình, Median, phân vị 90%, Error % và Throughput.
- Tạo báo cáo HTML, chụp ảnh minh họa và lưu sản phẩm lên GitHub.

## 2. Lưu ý quan trọng về đối tượng kiểm thử

Bài thực hành có hai kịch bản riêng:

1. **Kiểm tra nhẹ trang Google:** có thể dùng `https://www.google.com/` với 1 người dùng ảo và 1 vòng lặp để xác nhận JMeter gửi request và nhận phản hồi. Đây không phải bài đo khả năng chịu tải của Google.
2. **Kiểm thử tải với số lượng lớn hơn:** chỉ thực hiện với website do sinh viên sở hữu, môi trường local/staging, hoặc hệ thống mà sinh viên đã được chủ quản cấp phép kiểm thử. Ví dụ cấu hình học tập: 100 người dùng ảo, ramp-up 60 giây, 10 vòng lặp, tương đương khoảng 1.000 lần thực thi sampler HTTP chính.

**Không chạy cấu hình tải lớn lên Google hoặc website bên thứ ba khi chưa có văn bản/ủy quyền rõ ràng.** Không dùng cấu hình tải lớn để gây gián đoạn dịch vụ. Nếu chỉ muốn minh họa một trang như Google, giữ ở mức kiểm tra nhẹ. Nếu cần kiểm thử hiệu năng, hãy dùng máy chủ demo hoặc môi trường được phép.

## 3. Công cụ và môi trường

- **Apache JMeter:** [Tải tại trang chính thức](https://jmeter.apache.org/download_jmeter.cgi).
- **Tài liệu bắt đầu:** [JMeter Getting Started](https://jmeter.apache.org/usermanual/get-started.html).
- **Java:** [Điền kết quả `java -version` trên máy của bạn].
- **JMeter:** [Điền phiên bản trong Help → About Apache JMeter].
- **Hệ điều hành:** [Ví dụ: Windows 11 64-bit].
- **Máy kiểm thử:** [Điền CPU/RAM nếu giảng viên yêu cầu].

Không ghi rằng bài kiểm thử thành công nếu chưa chạy thực tế. Số liệu và ảnh trong báo cáo phải lấy từ lần chạy của chính bạn.

## 4. Kịch bản kiểm thử

Có hai file kịch bản dùng chung các tham số: [`test-plan/jmeter-page-test.jmx`](test-plan/jmeter-page-test.jmx) để mở bằng GUI và chụp kết quả; [`test-plan/jmeter-load-test.jmx`](test-plan/jmeter-load-test.jmx) dành cho chạy tải non-GUI (các listener đồ họa đã tắt để giảm tiêu thụ bộ nhớ).

### 4.1. Các tham số có thể cấu hình

| Tham số JMeter | Giá trị mặc định | Ý nghĩa |
|---|---:|---|
| `target_protocol` | `https` | Giao thức HTTP hoặc HTTPS |
| `target_host` | `www.google.com` | Tên miền cần kiểm thử, không ghi `https://` |
| `target_port` | `443` | Cổng máy chủ; thường là 443 cho HTTPS hoặc 80 cho HTTP |
| `target_path` | `/` | Đường dẫn của trang, ví dụ `/`, `/search` hoặc `/health` |
| `threads` | `1` | Số người dùng ảo |
| `rampup` | `1` | Số giây để khởi động toàn bộ người dùng ảo |
| `loops` | `1` | Số vòng lặp của mỗi người dùng |

Một người dùng thực thi sampler một lần trong mỗi vòng lặp. Vì vậy, với một HTTP Request chính:

**Số lần thực thi dự kiến = `threads × loops`.**

Ví dụ `100 threads × 10 loops = 1.000` lần thực thi HTTP Request chính. Đây là tổng số lần thực thi trong suốt bài kiểm thử, **không** có nghĩa là 1.000 người dùng đồng thời. Với 100 threads, mức đồng thời tối đa mô phỏng là khoảng 100 người dùng ảo; số lượng thực tế và thời điểm request phụ thuộc thời gian phản hồi, redirect và cấu hình kịch bản.

### 4.2. Luồng Test Plan

```text
Test Plan - Single Page / Load Test
└── Thread Group (cấu hình bằng tham số)
    └── HTTP Request - GET trang đích
        └── Response Assertion - kiểm tra HTTP 200
    ├── View Results Tree
    ├── Summary Report
    └── Aggregate Report
```

Kịch bản có assertion mong đợi mã HTTP `200`. Nếu trang đích hợp lệ nhưng chủ đích trả mã khác (ví dụ endpoint redirect hoặc trang xác thực), cần điều chỉnh assertion theo tiêu chí của bài kiểm thử và ghi rõ lý do trong báo cáo.

## 5. Cách thực hiện

### 5.1. Cài đặt và mở JMeter

1. Kiểm tra Java bằng Command Prompt hoặc PowerShell:

   ```powershell
   java -version
   ```

2. Tải gói **Binaries** của JMeter từ [trang tải chính thức](https://jmeter.apache.org/download_jmeter.cgi), giải nén.
3. Trên Windows, chạy `bin\jmeter.bat` trong thư mục JMeter.
4. Trong JMeter, chọn **File → Open** và mở `test-plan/jmeter-page-test.jmx`.

### 5.2. Kịch bản A – kiểm tra nhẹ trang Google

Mục đích là kiểm tra kết nối và cách sử dụng JMeter, không tạo tải lớn lên Google. Có thể chạy bằng giao diện đồ họa hoặc chế độ dòng lệnh.

**PowerShell:** chạy tại thư mục gốc repository. Nếu lệnh `jmeter` chưa có trong PATH, thay bằng đường dẫn đầy đủ đến `jmeter.bat`.

```powershell
New-Item -ItemType Directory -Force results, reports | Out-Null
Remove-Item results/google-smoke.jtl -ErrorAction SilentlyContinue
Remove-Item reports/google-smoke-html -Recurse -Force -ErrorAction SilentlyContinue
jmeter -n -t "test-plan/jmeter-page-test.jmx" `
  -Jtarget_protocol=https -Jtarget_host=www.google.com -Jtarget_port=443 -Jtarget_path=/ `
  -Jthreads=1 -Jrampup=1 -Jloops=1 `
  -l "results/google-smoke.jtl" -e -o "reports/google-smoke-html"
```

Sau khi hoàn tất, mở `reports/google-smoke-html/index.html` để xem báo cáo. Nếu request thất bại do mạng, chính sách truy cập, redirect hoặc phản hồi của website thay đổi, hãy ghi đúng kết quả thực tế; không lặp lại với tải lớn để cố vượt qua lỗi.

### 5.3. Kịch bản B – kiểm thử tải lớn hơn trên hệ thống được phép

Trước khi chạy, thay `staging.example.com` bằng **tên miền môi trường kiểm thử do bạn sở hữu hoặc được cấp phép sử dụng**. Có thể thay `target_path` bằng đường dẫn trang cụ thể của hệ thống đó, chẳng hạn `/products` hoặc `/health`. Không dùng lệnh này với `www.google.com` hay website bên thứ ba nếu không được cho phép rõ ràng.

**Cấu hình ví dụ: 100 người dùng ảo × 10 vòng lặp ≈ 1.000 lần thực thi sampler chính.** Ramp-up 60 giây giúp các thread khởi động dần thay vì bắt đầu đồng thời.

```powershell
New-Item -ItemType Directory -Force results, reports | Out-Null
Remove-Item results/authorized-load-test.jtl -ErrorAction SilentlyContinue
Remove-Item reports/authorized-load-test-html -Recurse -Force -ErrorAction SilentlyContinue
jmeter -n -t "test-plan/jmeter-load-test.jmx" `
  -Jtarget_protocol=https -Jtarget_host=staging.example.com -Jtarget_port=443 -Jtarget_path=/ `
  -Jthreads=100 -Jrampup=60 -Jloops=10 `
  -l "results/authorized-load-test.jtl" -e -o "reports/authorized-load-test-html"
```

**Trình tự tăng tải được khuyến nghị cho môi trường được cấp phép:**

| Lần chạy | Người dùng ảo (`threads`) | Ramp-up | Vòng lặp (`loops`) | Số lần thực thi sampler chính dự kiến |
|---|---:|---:|---:|---:|
| Baseline | 10 | 30 giây | 2 | 20 |
| Mức trung bình | 50 | 60 giây | 5 | 250 |
| Mức cao hơn trong bài thực hành | 100 | 60 giây | 10 | 1.000 |

Chạy từng mức một và quan sát kết quả trước khi tăng. Chỉ tăng tiếp nếu hệ thống thử nghiệm được cho phép và không có dấu hiệu quá tải. Thỏa thuận trước với người quản trị về thời gian chạy, giới hạn tải và điều kiện dừng.

**Lưu ý khi chạy tải:**

- Dùng chế độ non-GUI (`-n`) để chạy tải; không dùng View Results Tree để theo dõi từng request ở bài chạy lớn.
- Trong giao diện JMeter, có thể disable listener nặng như **View Results Tree** trước khi chạy tải. Dùng file `.jtl` và báo cáo HTML để phân tích sau.
- Không chạy nhiều bài tải đồng thời từ cùng máy nếu chưa tính đến tài nguyên máy tạo tải.
- Dừng bài kiểm thử nếu hệ thống có dấu hiệu chậm bất thường, báo lỗi, hoặc người quản trị yêu cầu dừng.

### 5.4. Tạo hoặc xem báo cáo HTML

Lệnh chạy ở mục 5.2 và 5.3 đã dùng `-e -o` để tạo báo cáo HTML. Thư mục đích báo cáo phải chưa tồn tại hoặc đang trống. Mở file `index.html` trong thư mục báo cáo tương ứng bằng trình duyệt.

Nếu chỉ chạy qua GUI, trước tiên có thể chạy và xem listener. Để sinh báo cáo HTML, lưu kết quả ra file `.jtl`, sau đó dùng lệnh JMeter non-GUI với tùy chọn `-g` để tạo dashboard từ file kết quả, ví dụ:

```powershell
jmeter -g "results/google-smoke.jtl" -o "reports/google-dashboard"
```

Không chạy lệnh tạo báo cáo nếu thư mục `reports/google-dashboard` đã có dữ liệu cũ; hãy đổi tên hoặc xóa thư mục cũ trước.

## 6. Ghi nhận và phân tích kết quả

Điền bảng theo đúng kết quả xuất hiện trong Summary/Aggregate Report hoặc HTML Dashboard. Không dùng số liệu giả định thay cho kết quả thật.

| Chỉ số | Kết quả thực tế | Cách diễn giải |
|---|---:|---|
| Số mẫu (`# Samples`) | [Điền] | Số mẫu được báo cáo trong lần chạy; kiểm tra redirect/sub-sample nếu có |
| Số request lỗi | [Điền] | Đối chiếu với Error % và các mẫu lỗi |
| Tỷ lệ lỗi (`Error %`) | [Điền]% | Tỷ lệ mẫu bị đánh dấu lỗi; không đồng nghĩa với thời gian phản hồi chậm |
| Trung bình (`Average`) | [Điền] ms | Thời gian phản hồi trung bình của các mẫu |
| Trung vị (`Median`) | [Điền] ms | Mốc giữa phân bố thời gian phản hồi |
| Phân vị 90% (`90% Line`) | [Điền] ms | Khoảng 90% mẫu có thời gian phản hồi không vượt quá mốc này |
| Nhỏ nhất (`Min`) | [Điền] ms | Mẫu nhanh nhất |
| Lớn nhất (`Max`) | [Điền] ms | Mẫu chậm nhất |
| Thông lượng (`Throughput`) | [Điền] | Ghi đúng đơn vị hiển thị trong báo cáo JMeter |

### Mẫu nhận xét

- **Độ đúng:** [Assertion HTTP 200 đạt hay không? Có lỗi mạng hoặc mã phản hồi khác không?]
- **Thời gian phản hồi:** [Nêu Average, Median, 90% Line, Min và Max từ dữ liệu thật.]
- **Tỷ lệ lỗi và thông lượng:** [Nhận xét từ kết quả thực tế; không kết luận vượt quá phạm vi dữ liệu.]
- **So sánh các mức tải:** [Chỉ điền nếu bạn đã chạy nhiều mức tải trên cùng môi trường, cùng endpoint và cấu hình có thể so sánh.]
- **Giới hạn:** kết quả phụ thuộc vào máy chạy JMeter, kết nối mạng, cấu hình môi trường và tình trạng máy chủ. Một lần chạy ngắn không đủ để khẳng định khả năng chịu tải tối đa của toàn bộ website.

## 7. Ảnh minh họa và hướng dẫn chụp

Tạo thư mục `screenshots/`. Trên Windows có thể nhấn **Win + Shift + S**, chọn vùng cần chụp và lưu dưới dạng PNG. Ảnh phải là ảnh chụp thật từ máy của bạn; không thay bằng ảnh minh họa trên Internet. Không để lộ thông tin đăng nhập hoặc dữ liệu nhạy cảm.

### Ảnh 1 – Phiên bản công cụ

- **Tên:** `screenshots/01-jmeter-version.png`
- **Cách chụp:** mở JMeter → **Help → About Apache JMeter**; chụp hộp thoại có phiên bản. Mở PowerShell chạy `java -version` và chụp thêm nếu cần.
- **Ý nghĩa:** ghi nhận môi trường thực hành.

### Ảnh 2 – Cấu hình endpoint/trang đích

- **Tên:** `screenshots/02-target-page.png`
- **Cách chụp:** mở file `.jmx`, chọn HTTP Request. Chụp các trường giao thức, server, port, method `GET` và path. Với Google, trang đích là `https://www.google.com/`, ở mức smoke test. Với load test, chụp endpoint thuộc môi trường được phép kiểm thử.
- **Ý nghĩa:** xác nhận đang kiểm thử đúng một trang cụ thể.

### Ảnh 3 – Cấu hình số người dùng và vòng lặp

- **Tên:** `screenshots/03-thread-group.png`
- **Cách chụp:** chọn Thread Group. Nếu mở file `.jmx` có sử dụng `${__P(...)}`, chụp cấu hình tham số hoặc chụp cửa sổ terminal thể hiện các giá trị `threads`, `rampup`, `loops` của lần chạy.
- **Ý nghĩa:** minh họa lượng tải mô phỏng và thời gian khởi động.

### Ảnh 4 – Response Assertion

- **Tên:** `screenshots/04-response-assertion.png`
- **Cách chụp:** chọn **Assert response code is 200** dưới HTTP Request; chụp trường Response Code và giá trị `200`.
- **Ý nghĩa:** thể hiện điều kiện kiểm tra phản hồi.

### Ảnh 5 – Kết quả kiểm tra nhẹ Google

- **Tên:** `screenshots/05-google-smoke-result.png`
- **Cách chụp:** chạy kịch bản A với `threads=1`, `loops=1`; mở View Results Tree, chọn mẫu và chụp Sampler Result gồm response code, thời gian và trạng thái assertion nếu có.
- **Ý nghĩa:** chứng minh JMeter đã thử truy cập trang đích ở mức nhẹ. Nếu bị chặn hoặc không nhận được mã 200, chụp đúng kết quả đó và giải thích.

### Ảnh 6 – Kết quả Summary Report

- **Tên:** `screenshots/06-summary-report.png`
- **Cách chụp:** sau khi chạy xong, chọn Summary Report; để các cột `# Samples`, `Average`, `Min`, `Max`, `Error %`, `Throughput` hiển thị rõ rồi chụp.
- **Ý nghĩa:** tổng hợp chỉ số của lần chạy.

### Ảnh 7 – Aggregate Report

- **Tên:** `screenshots/07-aggregate-report.png`
- **Cách chụp:** chọn Aggregate Report và chụp các cột `# Samples`, `Average`, `Median`, `90% Line`, `Min`, `Max`, `Error %`, `Throughput`.
- **Ý nghĩa:** hỗ trợ phân tích phân bố thời gian phản hồi.

### Ảnh 8 – Kết quả tải lớn trên môi trường được cấp phép

- **Tên:** `screenshots/08-authorized-load-result.png`
- **Cách chụp:** chỉ sau khi hoàn thành kịch bản B trên môi trường được cho phép, chụp terminal thể hiện lệnh chạy (ẩn thông tin nhạy cảm nếu có) và/hoặc HTML Dashboard. Đảm bảo ảnh thể hiện target đã dùng, số người dùng, ramp-up và vòng lặp.
- **Ý nghĩa:** chứng minh cấu hình tải lớn được chạy trên hệ thống được phép, không phải Google.

### Ảnh 9 – HTML Dashboard

- **Tên:** `screenshots/09-html-dashboard.png`
- **Cách chụp:** mở file `index.html` được tạo sau khi chạy; chụp phần Statistics/summary. Chụp thêm ảnh khác nếu cần biểu đồ phía dưới.
- **Ý nghĩa:** minh họa kết quả tổng hợp và biểu đồ.

### Ảnh 10 – Repository GitHub

- **Tên:** `screenshots/10-github-repository.png`
- **Cách chụp:** sau khi push lên GitHub, mở repository và chụp phần tên repo, danh sách file cùng phần đầu README.
- **Ý nghĩa:** chứng minh sản phẩm đã được tổ chức và nộp.

Sau khi có ảnh, có thể nhúng ảnh trực tiếp vào README bằng cú pháp Markdown, ví dụ:

```markdown
![Cấu hình trang đích](screenshots/02-target-page.png)
![Summary Report](screenshots/06-summary-report.png)
```

## 8. Cấu trúc repository

```text
jmeter-github-submission/
├── README.md
├── .gitignore
├── test-plan/
│   ├── jmeter-page-test.jmx
│   └── jmeter-load-test.jmx
├── screenshots/
│   └── (các ảnh PNG chụp từ lần chạy thật)
├── results/
│   └── (file .jtl, mặc định không commit)
└── reports/
    └── (HTML Dashboard, có thể không commit nếu dung lượng lớn)
```

Không cần đưa toàn bộ report hoặc file `.jtl` dung lượng lớn lên GitHub nếu giảng viên chỉ yêu cầu ảnh minh họa; README và ảnh chụp quan trọng là đủ. Tuân theo quy định nộp bài của giảng viên nếu có yêu cầu khác.

## 9. Đưa dự án lên GitHub

1. Truy cập [GitHub](https://github.com/new) và tạo repository tên `jmeter-github-submission`.
2. Giải nén project vào một thư mục trên máy.
3. Mở terminal tại thư mục dự án và chạy (thay `<username>` bằng tài khoản GitHub của bạn):

```bash
git init
git add README.md .gitignore test-plan screenshots
git commit -m "Add JMeter page and load test report"
git branch -M main
git remote add origin https://github.com/<username>/jmeter-github-submission.git
git push -u origin main
```

Nếu đã cấu hình remote `origin`, dùng `git remote -v` kiểm tra thay vì thêm lại. Sau khi chụp đủ ảnh, chạy `git add screenshots README.md` rồi commit và push lần nữa để cập nhật báo cáo.

## 10. Kết luận

[Viết 3–5 câu dựa trên kết quả thật: đã thiết lập được kịch bản kiểm thử một trang, assertion có đạt không, những chỉ số chính đo được, sự khác biệt giữa smoke test và load test, và giới hạn của kết quả.]

## Tài liệu tham khảo

1. [Apache JMeter – Download](https://jmeter.apache.org/download_jmeter.cgi)
2. [Apache JMeter – Getting Started](https://jmeter.apache.org/usermanual/get-started.html)
3. [Apache JMeter – Building a Web Test Plan](https://jmeter.apache.org/usermanual/build-web-test-plan)
4. [Apache JMeter – Elements of a Test Plan](https://jmeter.apache.org/usermanual/test_plan.html)
5. [Apache JMeter – Generating Dashboard Report](https://jmeter.apache.org/usermanual/generating-dashboard.html)
