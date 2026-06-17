# 📡 Báo Cáo Kiểm Thử API — Open-Meteo Weather API

> **Người kiểm thử:** Giang Thành An
> **Ngày kiểm thử:** 17/06/2026
> **Endpoint:** `https://api.open-meteo.com/v1/forecast`

---

## 📋 Mục Lục

- [Mục Tiêu Kiểm Thử](#-mục-tiêu-kiểm-thử)
- [Môi Trường Kiểm Thử](#-môi-trường-kiểm-thử)
- [Kết Quả Tổng Quan](#-kết-quả-tổng-quan)
- [Kịch Bản Kiểm Thử](#-kịch-bản-kiểm-thử)
- [Phát Hiện Lỗi](#-phát-hiện-lỗi)
- [Nhận Xét & Đề Xuất](#-nhận-xét--đề-xuất)
- [Mã Trạng Thái HTTP](#-mã-trạng-thái-http-tham-khảo)
- [Bộ Sưu Tập Postman](#-bộ-sưu-tập-postman)
- [Phụ Lục](#-phụ-lục)

---

## 🎯 Mục Tiêu Kiểm Thử

- Xác thực chức năng và độ tin cậy của Open-Meteo Weather API
- Kiểm thử API với các tham số khác nhau
- Xác minh định dạng dữ liệu và độ chính xác của nội dung
- Đánh giá hiệu suất API và khả năng xử lý lỗi

---

## 🛠 Môi Trường Kiểm Thử

| Thông Tin | Chi Tiết |
|-----------|----------|
| Công cụ | Postman (Phiên bản mới nhất) |
| Base URL | `https://api.open-meteo.com/v1/forecast` |
| Phương thức | GET |
| Loại kiểm thử | Tự động và thủ công |

---

## 📊 Kết Quả Tổng Quan

| Chỉ Số | Giá Trị |
|--------|---------|
| Tổng số kịch bản | 4 |
| ✅ Thành công | 3 |
| ❌ Thất bại | 1 |
| Tỷ lệ thành công | **75%** |

---

## 🧪 Kịch Bản Kiểm Thử

### Kịch Bản 1 — Yêu Cầu Cơ Bản ✅

**Mục đích:** Xác minh API trả về dữ liệu thời tiết với tham số tối thiểu

```
GET https://api.open-meteo.com/v1/forecast
  ?latitude=52.52
  &longitude=13.41
  &hourly=temperature_2m
```

| Thông Tin | Chi Tiết |
|-----------|----------|
| HTTP Status | `200 OK` |
| Thời gian phản hồi | ~55ms |
| Trạng thái | ✅ THÀNH CÔNG |

<details>
<summary>Xem phản hồi mẫu</summary>

```json
{
  "latitude": 52.52,
  "longitude": 13.419998,
  "generationtime_ms": 0.055,
  "utc_offset_seconds": 0,
  "timezone": "GMT",
  "timezone_abbreviation": "GMT",
  "elevation": 38.0,
  "hourly_units": {
    "time": "iso8601",
    "temperature_2m": "°C"
  },
  "hourly": {
    "time": ["2026-06-17T00:00", "2026-06-17T01:00", "..."],
    "temperature_2m": [17.2, 16.8, 16.5, "..."]
  }
}
```

</details>

---

### Kịch Bản 2 — Tham Số Mở Rộng ✅

**Mục đích:** Kiểm thử API với nhiều tham số và múi giờ khác nhau

```
GET https://api.open-meteo.com/v1/forecast
  ?latitude=52.52
  &longitude=13.41
  &hourly=temperature_2m,relativehumidity_2m,windspeed_10m
  &timezone=Asia/Singapore
  &past_days=1
```

| Thông Tin | Chi Tiết |
|-----------|----------|
| HTTP Status | `200 OK` |
| Thời gian phản hồi | ~117ms |
| Trạng thái | ✅ THÀNH CÔNG |

**Dữ liệu đã xác minh:**
- Nhiều biến số thời tiết được trả về đầy đủ
- Múi giờ `Asia/Singapore` (+08) được áp dụng chính xác
- Bao gồm dữ liệu ngày hôm trước

<details>
<summary>Xem phản hồi mẫu</summary>

```json
{
  "latitude": 52.52,
  "longitude": 13.419998,
  "utc_offset_seconds": 28800,
  "timezone": "Asia/Singapore",
  "timezone_abbreviation": "+08",
  "elevation": 38.0,
  "hourly_units": {
    "time": "iso8601",
    "temperature_2m": "°C",
    "relativehumidity_2m": "%",
    "windspeed_10m": "km/h"
  },
  "hourly": {
    "time": ["2026-06-16T00:00", "2026-06-16T01:00", "..."],
    "temperature_2m": [18.2, 17.8, 17.3, "..."],
    "relativehumidity_2m": [65, 68, 72, "..."],
    "windspeed_10m": [12.3, 10.8, 9.5, "..."]
  }
}
```

</details>

---

### Kịch Bản 3 — Tham Số Không Hợp Lệ ✅

**Mục đích:** Xác minh khả năng xử lý lỗi khi truyền vĩ độ không hợp lệ

```
GET https://api.open-meteo.com/v1/forecast
  ?latitude=999
  &longitude=13.41
  &hourly=temperature_2m
```

| Thông Tin | Chi Tiết |
|-----------|----------|
| HTTP Status | `400 Bad Request` |
| Thời gian phản hồi | ~45ms |
| Trạng thái | ✅ THÀNH CÔNG |

```json
{
  "error": true,
  "reason": "Invalid latitude. Must be between -90 and 90.",
  "error_id": 0
}
```

---

### Kịch Bản 4 — Dữ Liệu Lịch Sử ❌

**Mục đích:** Kiểm tra khả năng truy xuất dữ liệu quá khứ

```
GET https://api.open-meteo.com/v1/forecast
  ?latitude=52.52
  &longitude=13.41
  &hourly=temperature_2m
  &start_date=2020-01-01
  &end_date=2020-01-07
```

| Thông Tin | Chi Tiết |
|-----------|----------|
| HTTP Status | `200 OK` |
| Trạng thái | ❌ KHÔNG THÀNH CÔNG |

**Vấn đề phát hiện:** API trả về 168 giá trị thời gian bắt đầu từ tháng 6/2026 và quay ngược về năm 1920, không khớp với khoảng thời gian yêu cầu (2020-01-01 đến 2020-01-07).

> ⚠️ **Ghi chú:** Đây có thể là lỗi khi xử lý tham số `start_date`/`end_date`, hoặc cần sử dụng endpoint riêng cho dữ liệu lịch sử.

---

## 🐛 Phát Hiện Lỗi

### `API-001` — Dữ Liệu Lịch Sử Không Chính Xác

| Thuộc tính | Chi tiết |
|------------|----------|
| **Mức độ** | 🔴 Cao |
| **Tham số liên quan** | `start_date`, `end_date` |

**Mô tả:** Khi truyền `start_date=2020-01-01` và `end_date=2020-01-07`, API trả về dữ liệu từ khoảng 1920–2026 thay vì 7 ngày được yêu cầu.

**Đề xuất:** Kiểm tra lại logic xử lý tham số thời gian. Có thể cần thêm `timezone` hoặc sử dụng endpoint `/v1/archive` dành riêng cho dữ liệu lịch sử.

---

### `API-002` — Định Dạng Thời Gian Không Chuẩn

| Thuộc tính | Chi tiết |
|------------|----------|
| **Mức độ** | 🟡 Trung bình |

**Mô tả:** Một số trường hợp API trả về thời gian không hợp lệ như `2026-06-17T99:00` hoặc `2026-06-18:00`, gây khó khăn khi parse dữ liệu.

**Đề xuất:** Chuẩn hóa toàn bộ timestamp đầu ra theo chuẩn ISO 8601.

---

## 💡 Nhận Xét & Đề Xuất

### ✅ Điểm Mạnh

- **Hiệu suất tốt:** Thời gian phản hồi dưới 120ms
- **Thông báo lỗi rõ ràng:** Cấu trúc lỗi được định dạng tốt, dễ debug
- **Tham số linh hoạt:** Hỗ trợ nhiều tổ hợp biến số thời tiết
- **Hỗ trợ múi giờ:** Chuyển đổi chính xác theo yêu cầu
- **Định dạng JSON sạch:** Cấu trúc phản hồi nhất quán

### 📝 Đề Xuất Cải Thiện

- **Rate Limiting:** Bổ sung response headers giới hạn tốc độ để tránh lạm dụng
- **Tài liệu:** Cập nhật hướng dẫn sử dụng `start_date`/`end_date` rõ ràng hơn
- **Xác thực đầu vào:** Tăng cường validate tham số trước khi xử lý
- **Dữ liệu lịch sử:** Sửa lỗi `API-001` hoặc hướng dẫn người dùng dùng endpoint phù hợp

---

## 📘 Mã Trạng Thái HTTP Tham Khảo

| Mã | Mô Tả | Cách Xử Lý |
|----|--------|------------|
| `200` | Thành công | Parse và sử dụng dữ liệu |
| `400` | Yêu cầu không hợp lệ | Kiểm tra lại giá trị tham số |
| `404` | Không tìm thấy | Xác minh URL endpoint |
| `429` | Quá nhiều yêu cầu | Triển khai cơ chế retry với backoff |
| `500` | Lỗi máy chủ nội bộ | Liên hệ nhà cung cấp API |

---

## 📦 Bộ Sưu Tập Postman

### Cấu Trúc Collection

```
Open-Meteo Weather API Tests
├── Yêu Cầu Dự Báo Cơ Bản
│   └── GET ?latitude=52.52&longitude=13.41&hourly=temperature_2m
├── Yêu Cầu Dự Báo Mở Rộng
│   └── GET — Nhiều biến số + múi giờ
├── Kiểm Thử Tham Số Không Hợp Lệ
│   └── GET ?latitude=999&...
├── Kiểm Thử Dữ Liệu Lịch Sử
│   └── GET ?start_date=...&end_date=...
└── Kiểm Thử Các Thành Phố Khác
    ├── Tokyo:    latitude=35.6762, longitude=139.6503
    ├── New York: latitude=40.7128, longitude=-74.0060
    └── Sydney:   latitude=-33.8688, longitude=151.2093
```

### Biến Môi Trường

```json
{
  "base_url": "https://api.open-meteo.com/v1/forecast",
  "latitude": "52.52",
  "longitude": "13.41",
  "timezone": "Asia/Singapore"
}
```

### Script Kiểm Thử Tự Động

```javascript
// Kiểm tra mã trạng thái
pm.test("Mã trạng thái là 200", function () {
    pm.response.to.have.status(200);
});

// Kiểm tra phản hồi JSON hợp lệ
pm.test("Phản hồi là JSON hợp lệ", function () {
    pm.response.to.be.json;
});

// Kiểm tra các trường bắt buộc
pm.test("Phản hồi chứa các trường bắt buộc", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('latitude');
    pm.expect(jsonData).to.have.property('longitude');
    pm.expect(jsonData).to.have.property('hourly');
    pm.expect(jsonData.hourly).to.have.property('temperature_2m');
});

// Kiểm tra kiểu dữ liệu
pm.test("Dữ liệu nhiệt độ là mảng số", function () {
    const jsonData = pm.response.json();
    const temps = jsonData.hourly.temperature_2m;
    pm.expect(temps).to.be.an('array');
    pm.expect(temps[0]).to.be.a('number');
});
```

---

## 📎 Phụ Lục

### Lệnh cURL Mẫu

```bash
# Yêu cầu dự báo cơ bản
curl "https://api.open-meteo.com/v1/forecast?latitude=52.52&longitude=13.41&hourly=temperature_2m"

# Yêu cầu dự báo với nhiều tham số
curl "https://api.open-meteo.com/v1/forecast?latitude=52.52&longitude=13.41&hourly=temperature_2m,relativehumidity_2m,windspeed_10m&timezone=Asia/Singapore&past_days=1"

# Yêu cầu dữ liệu lịch sử (⚠️ cần kiểm tra lại)
curl "https://api.open-meteo.com/v1/forecast?latitude=52.52&longitude=13.41&hourly=temperature_2m&start_date=2020-01-01&end_date=2020-01-07"
```

---

> 📅 **Ngày tạo báo cáo:** 17/06/2026 | **Người tạo:** Giang Thành An
> 🔄 **Đánh giá tiếp theo:** Sau khi sửa lỗi `API-001` và `API-002`
