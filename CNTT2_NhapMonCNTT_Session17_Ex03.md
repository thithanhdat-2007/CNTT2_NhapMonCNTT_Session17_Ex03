# BÀI TẬP THỰC HÀNH: GỠ LỖI GIAO DIỆN VỚI INSPECT ELEMENT VÀ CSS BOX MODEL

## PHẦN 1: BÁO CÁO PHÂN TÍCH (I/O)

### 1.1. Hiện trạng giao diện và nguyên nhân trực tiếp qua Inspect Element (F12)
Khi kiểm tra trang khóa học trên hệ thống PTIT-LMS bằng công cụ Inspect Element (phím F12), hai khối chính của giao diện đang bị chồng lấn lên nhau:
1. Khung bên trái là "Nội dung bài học" (`div#lesson-content`) tràn sang bên phải và đè trực tiếp lên cạnh trái của khung "Danh sách video" (`div#video-list`).
2. Khi chọn phần tử trong bảng Elements, vùng hiển thị nội dung màu xanh dương và đường viền màu đỏ (`border: 2px solid red`) của hai khung cắt vào nhau, che khuất một phần tiêu đề và danh sách video ở cột bên phải.

### 1.2. Phân tích 4 lớp của CSS Box Model trên tab Computed và bảng Styles
Qua việc soi trực tiếp phần tử đang chọn (`div#lesson-content`) và đối chiếu với phần tử kế bên (`div#video-list`) trên cửa sổ DevTools, thông số của 4 lớp trong Box Model được ghi nhận như sau:

1. **Lớp Content (Vùng nội dung - lớp trong cùng, màu xanh dương):**
   - Trên sơ đồ Box Model (tab Computed): Kích thước vùng chứa nội dung đang hiển thị là `840 x 600` (rộng 840px, cao 600px).
   - Trên bảng Styles: Cả hai khung `div#lesson-content` và `div#video-list` đều được khai báo `width: 65%` (riêng `#lesson-content` có gắn `!important`) và cùng dùng `float: left` để xếp nằm ngang cạnh nhau.
   - Điểm bất thường: Tổng chiều rộng chỉ riêng phần Content của hai khung đã là `65% + 65% = 130%`, vượt quá `100%` chiều rộng của khung chứa cha.

2. **Lớp Padding (Vùng đệm - nằm giữa Content và Border, màu xanh lá nhạt):**
   - Trên sơ đồ Box Model (tab Computed): Thông số hiển thị lần lượt là cạnh trên `12px`, cạnh dưới `10px`, cạnh trái `10px`, cạnh phải `10px`.
   - Trên bảng Styles: Quy tắc CSS đang gán `padding: 15px` cho cả 4 cạnh của `#lesson-content` và `#video-list`.
   - Ý nghĩa: Lớp đệm tạo khoảng trống giữa chữ bên trong và đường viền đỏ, nhưng đồng thời cộng thêm `20px` đến `30px` (trái + phải) vào tổng chiều rộng thực tế của mỗi hộp.

3. **Lớp Border (Đường viền - bao quanh Padding, màu vàng nhạt trên sơ đồ):**
   - Trên sơ đồ Box Model (tab Computed): Độ dày viền hiển thị ở cạnh trên `1px`, cạnh trái `1px`, cạnh phải `1px`, cạnh dưới `15px`.
   - Trên bảng Styles: Đang khai báo `border: 2px solid red;` (đường viền nét liền màu đỏ dày 2px).
   - Ý nghĩa: Đường viền giúp nhìn rõ ranh giới của khung nhưng cũng làm tăng thêm `4px` chiều rộng cho mỗi phần tử.

4. **Lớp Margin (Lề ngoài - lớp ngoài cùng, màu cam nâu):**
   - Trên sơ đồ Box Model (tab Computed): Cạnh trên và cạnh trái xuất hiện giá trị âm `-10px`, cạnh phải `15px`, cạnh dưới `0px`.
   - Trên bảng Styles: Cả `#lesson-content` và `#video-list` đều bị gán `margin-right: -20px;` và `margin-left: 20px;`.
   - Điểm bất thường nghiêm trọng: Giá trị lề âm (`margin-right: -20px` trên CSS và `-10px` trên sơ đồ) không đẩy phần tử ra xa mà kéo phần tử kế tiếp (`#video-list`) dịch ngược vào bên trong khung `#lesson-content`. Kết hợp với thuộc tính `overflow: visible;`, phần nội dung bị tràn không được gói gọn mà đè thẳng lên nhau.

### 1.3. Phác thảo sơ đồ Box Model hiện tại của phần tử bị lỗi (`div#lesson-content`)
Dưới đây là sơ đồ phác thảo mô hình hộp của phần tử đang bị lỗi dựa trên thông số thực tế trong DevTools:

```text
                       Margin (Lề ngoài)
                    Cạnh trên: -10px (CSS: 0px)
          ___________________________________________________
         |              Border (Viền: 2px solid red)         |
         |                  Cạnh trên: 1px                   |
         |      _______________________________________      |
         |     |        Padding (Đệm - CSS: 15px)      |     |
         |     |            Cạnh trên: 12px            |     |
         |     |      ___________________________      |     |
         |     |     |                           |     |     |
  Trái:  |Trái:|Trái:|     CONTENT (Nội dung)    |Phải:|Phải:| Phải:
  -10px  | 1px |10px |         840 x 600         |10px | 1px | 15px
 (CSS:   |     |     |  (width: 65% !important)  |     |     | (CSS:
  20px)  |     |     |___________________________|     |     | -20px)
         |     |                                       |     |
         |     |            Cạnh dưới: 10px            |     |
         |     |_______________________________________|     |
         |                                                   |
         |                  Cạnh dưới: 15px                  |
         |___________________________________________________|

                     Cạnh dưới: 0px / -10px
```

### 1.4. Dự đoán điều gì xảy ra nếu chỉ thay đổi Margin mà không kiểm tra Padding
Mặc định, Rendering Engine của trình duyệt tính kích thước phần tử theo cơ chế `box-sizing: content-box`. Khi đó, tổng chiều rộng thực tế mà một phần tử chiếm chỗ trên trang được tính theo công thức:

`Tổng chiều rộng chiếm chỗ = width (Content) + Padding trái + Padding phải + Border trái + Border phải + Margin trái + Margin phải`

Nếu chỉ sửa `margin` từ số âm sang số dương (ví dụ đổi `margin-right: -20px` thành `margin-right: 20px` để tách hai khung ra) mà không kiểm tra `padding`, `border` và `width`:
1. **Giao diện bị vỡ bố cục 2 cột (rớt dòng):** Ngay cả khi giảm `width` của hai khung để tổng bằng `100%` (ví dụ `65%` và `35%`), phần `padding: 15px` mỗi bên (`30px` mỗi khung) và `border: 2px` mỗi bên (`4px` mỗi khung) cộng với `margin` dương sẽ làm tổng chiều rộng thực tế vượt quá `100%` chiều ngang khung chứa (`100% + 60px padding + 8px border + margin > 100%`).
2. **Hậu quả với thuộc tính `float: left`:** Khi tổng chiều rộng thực tế của hai khối vượt quá `100%` dù chỉ 1px, khung thứ hai (`div#video-list`) không còn đủ chỗ đứng bên cạnh nữa mà bị đẩy tụt hẳn xuống bên dưới khung `#lesson-content`.
3. **Tràn ngang khung chứa:** Nếu cố giữ hai khối trên cùng một hàng mà không gộp `padding` vào `width`, hộp phần tử sẽ bị phình ra ngoài phạm vi trang và tạo thanh cuộn ngang ngoài ý muốn.

## PHẦN 2: THIẾT KẾ GIẢI PHÁP

### 2.1. Nguyên tắc điều chỉnh tại bảng Styles
Để tách biệt hai khung "Nội dung bài học" và "Danh sách video" mà không làm vỡ tổng thể trang web, em thực hiện 4 bước chỉnh sửa tại bảng Styles:
1. Thêm `box-sizing: border-box;` cho cả `#lesson-content` và `#video-list` để trình duyệt tự động gộp `padding` (`15px`) và `border` (`2px`) vào bên trong chiều rộng `width` đã khai báo.
2. Xóa bỏ giá trị lề âm `margin-right: -20px;` và thay bằng khoảng cách lề dương hợp lý để hai khung tách rời nhau.
3. Cân đối lại tỷ lệ `width` và `margin` của hai khung sao cho tổng chiều ngang bằng đúng `100%`:
   - Khung `#lesson-content`: `width: 65%`, `margin-left: 2%`, `margin-right: 2%`.
   - Khung `#video-list`: Giảm `width` từ `65%` xuống `29%`, `margin-left: 0`, `margin-right: 2%`.
   - Kiểm tra tổng kích thước: `2% + 65% + 2% + 29% + 2% = 100%`.
4. Đổi `overflow: visible;` thành `overflow: hidden;` để ngăn các phần tử con hoặc hình ảnh bên trong tràn ra ngoài đè lên khung bên cạnh.

### 2.2. Bảng đối chiếu thông số trước và sau khi sửa

| Phần tử | Thuộc tính CSS | Thông số đang lỗi | Thông số đề xuất sửa | Tác dụng sau khi sửa |
| :--- | :--- | :--- | :--- | :--- |
| `div#lesson-content` | `box-sizing` | Mặc định (`content-box`) | `border-box` | Gộp padding và border vào trong 65% width |
| `div#lesson-content` | `width` | `65% !important` | `65%` | Giữ tỷ lệ cột nội dung bài học chiếm 65% |
| `div#lesson-content` | `margin-left` | `20px` | `2%` | Tạo khoảng cách đều với mép trái trang |
| `div#lesson-content` | `margin-right` | `-20px` | `2%` | Xóa lề âm, tạo khoảng trống tách khỏi cột video |
| `div#lesson-content` | `padding` | `15px` | `15px` | Giữ độ thoáng cho chữ bên trong khung |
| `div#lesson-content` | `border` | `2px solid red` | `2px solid red` | Giữ viền kiểm tra nhưng không làm phình khung |
| `div#lesson-content` | `overflow` | `visible` | `hidden` | Chặn nội dung con tràn sang cột bên phải |
| `div#video-list` | `box-sizing` | Mặc định (`content-box`) | `border-box` | Gộp padding và border vào trong 29% width |
| `div#video-list` | `width` | `65%` | `29%` | Giảm chiều rộng để tổng 2 cột và lề vừa khít 100% |
| `div#video-list` | `margin-left` | `20px` | `0` | Tránh cộng dồn lề thừa ở giữa hai khung |
| `div#video-list` | `margin-right` | `-20px` | `2%` | Xóa lề âm, cách mép phải trang 2% |
| `div#video-list` | `padding` | `15px` | `15px` | Giữ khoảng đệm đồng bộ với khung bài học |
| `div#video-list` | `overflow` | `visible` | `hidden` | Đảm bảo danh sách video gọn trong khung |

### 2.3. Mã nguồn CSS đã sửa (`style.css`)

```css
/* =========================================================
   BÀI TẬP SESSION 16: GỠ LỖI GIAO DIỆN VỚI CSS BOX MODEL
   Sinh viên: Thi Thành Đạt - MSSV: N26DTCN041 - Lớp: CNTT2
   ========================================================= */

/* Khung Nội dung bài học (Cột bên trái) */
div#lesson-content {
    box-sizing: border-box;
    float: left;
    width: 65%;
    margin-left: 2%;
    margin-right: 2%;
    padding: 15px;
    border: 2px solid red;
    overflow: hidden;
}

/* Khung Danh sách video (Cột bên phải) */
div#video-list {
    box-sizing: border-box;
    float: left;
    width: 29%;
    margin-left: 0;
    margin-right: 2%;
    padding: 15px;
    border: 2px solid red;
    overflow: hidden;
}
```

### 2.4. Mã nguồn HTML minh họa kiểm thử (`index.html`)

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>PTIT-LMS - Lập trình Web</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h2>Trang chủ - Bài học 3: Xây dựng giao diện web</h2>

    <div id="lesson-content">
        <h3>NỘI DUNG BÀI HỌC</h3>
        <p><strong>BÀI HỌC 3: XÂY DỰNG GIAO DIỆN WEB</strong></p>
        <p>Vận dụng Inspect Element và mô hình hộp (CSS Box Model) để kiểm tra kích thước Content, Padding, Border và Margin của các phần tử trên trang.</p>
    </div>

    <div id="video-list">
        <h3>DANH SÁCH VIDEO</h3>
        <p>1. Giới thiệu lập trình Web</p>
        <p>2. Cấu trúc HTML và CSS cơ bản</p>
        <p>3. Thực hành gỡ lỗi với DevTools</p>
    </div>
</body>
</html>
```
