# Service Boundary - Nhóm: Tam Đại Quỷ Vương

## 1. Thông tin nhóm

- **Tên nhóm:** Tam Đại Quỷ Vương
- **Lớp:** CNTT17-13
- **Thành viên:** 1. Nguyễn Phương Nam
  2. Phạm Văn Huy
  3. Trần Mạnh Tiến
  4. Đinh Hoài Nam
- **Service nhóm phụ trách:** A6 - Core Business Service
- **Sản phẩm tổng thể của lớp:** Product A (Smart Campus System)

## 2. Actor

Ai tương tác với hệ thống/service?
- **Hệ thống Ingestion (IoT Ingestion):** Cung cấp dữ liệu cảm biến đã chuẩn hóa.
- **Hệ thống AI (AI Vision):** Cung cấp kết quả nhận diện từ camera.
- **Quản trị viên hệ thống (Admin):** Thiết lập các quy tắc, chính sách vận hành campus.
- **Hệ thống Notification & Analytics:** Tiếp nhận kết quả xử lý từ service A6.

## 3. System Boundary

Nhóm em xây phần nào?

**Phần nhóm kiểm soát:**
- **Logic nghiệp vụ lõi:** Xử lý các kịch bản thông minh (ví dụ: phát hiện xâm nhập, cảnh báo cháy, quản lý ra vào).
- **Bộ máy thực thi chính sách (Policy Engine):** Đối chiếu dữ liệu thực tế với các quy định đã thiết lập.
- **Quản lý trạng thái hệ thống:** Lưu trữ và điều phối các quyết định nghiệp vụ.

**Phần nhóm chỉ tích hợp:**
- Giao tiếp với tầng thu thập dữ liệu (IoT Ingestion Service).
- Nhận kết quả phân tích hình ảnh (AI Vision Service).
- Đẩy dữ liệu sang bộ phận thông báo (Notification Service).

## 4. Service Boundary

**Service của nhóm có trách nhiệm gì?**
- Tiếp nhận các "Sự kiện" (Events) từ tầng dữ liệu.
- Phân tích và đưa ra quyết định: Cho phép/Từ chối/Cảnh báo.
- Kích hoạt yêu cầu gửi thông báo khi có sự cố xảy ra.
- Cung cấp dữ liệu đã xử lý cho tầng phân tích (Analytics).

**Service KHÔNG làm gì?**
- Không trực tiếp kết nối với cảm biến hoặc phần cứng.
- Không thực hiện các thuật toán học máy (Machine Learning) để nhận diện hình ảnh.
- Không trực tiếp quản lý việc gửi tin nhắn/email tới người dùng cuối.

## 5. Input / Output

### Input
- Dữ liệu cảm biến chuẩn hóa (Nhiệt độ, độ ẩm, trạng thái cửa...).
- Metadata nhận diện (Danh tính người dùng, biển số xe, tọa độ vật thể).
- Cấu hình chính sách từ Admin (Ngưỡng cảnh báo, danh sách đen/trắng).

### Output
- Lệnh điều khiển thiết bị (Mở/khóa cổng, bật/tắt thiết bị).
- Yêu cầu cảnh báo (Alert Request) gửi tới Notification Service.
- Nhật ký sự kiện nghiệp vụ gửi tới Analytics Service.

## 6. API dự kiến

| Method | Endpoint | Mục đích |
|---|---|---|
| **GET** | `/health` | Kiểm tra trạng thái sống/chết của service |
| **POST** | `/business/evaluate` | Tiếp nhận và xử lý sự kiện dựa trên quy tắc |
| **GET** | `/business/policies` | Lấy danh sách các chính sách đang áp dụng |
| **PUT** | `/business/policies/{id}` | Cập nhật hoặc thay đổi chính sách nghiệp vụ |
| **GET** | `/business/decisions/history` | Truy vấn lịch sử các quyết định đã thực hiện |

## 7. Phụ thuộc service khác

- **Service này gọi đến service nào?**
  - **Notification Service:** Để gửi cảnh báo đến người dùng.
  - **Analytics Service:** Để lưu trữ dữ liệu phục vụ báo cáo/thống kê.
- **Service nào gọi đến service này?**
  - **IoT Ingestion Service:** Đẩy dữ liệu môi trường vào xử lý.
  - **AI Vision Service:** Đẩy kết quả nhận diện camera vào xử lý.

## 8. Sơ đồ minh họa

```mermaid
flowchart TD
    subgraph "Data Providers (Upstream)"
        Ingestion[IoT Ingestion Service]
        Vision[AI Vision Service]
    end

    subgraph "Nhóm: Tam Đại Quỷ Vương - TRUNG TÂM"
        A6[A6: Core Business]
        DB[(Business Rules & Logic DB)]
        A6 --- DB
    end

    subgraph "Consumer Services (Downstream)"
        Noti[Notification Service]
        Analytic[Analytics Service]
    end

    %% Luồng dữ liệu vào Lõi
    Ingestion -- "Chuẩn hóa dữ liệu" --> A6
    Vision -- "Kết quả nhận diện" --> A6

    %% Luồng xử lý từ Lõi đi ra
    A6 -- "Yêu cầu cảnh báo" --> Noti
    A6 -- "Dữ liệu nghiệp vụ" --> Analytic

    %% Kết quả cuối
    Noti --> User((Người dùng))