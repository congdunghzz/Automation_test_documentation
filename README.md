# AUTOMATION API VỚI POSTMAN & NEWMAN

## MỤC LỤC

1. [Tổng quan quy trình Automation API](#1-tổng-quan-quy-trình-automation-api-với-postman)
2. [Xây dựng Test Script trong Postman](#2-xây-dựng-test-script-trong-postman)
3. [Kiến thức JavaScript cần thiết](#3-kiến-thức-javascript-cần-thiết)
4. [Thực thi bằng Newman CLI](#4-thực-thi-bằng-newman)
5. [Tích hợp CI/CD](#5-tích-hợp-cicd)

---

## 1. TỔNG QUAN QUY TRÌNH AUTOMATION API VỚI POSTMAN

Quy trình kiểm thử tự động API sử dụng Postman được triển khai theo **6 bước tuần tự**, chuyển đổi từ kiểm thử thủ công sang tự động hóa hoàn toàn:

```mermaid
    A[1. Tạo Collection] --> B[2. Xây dựng API]
    B --> C[3. Viết Test Script]
    C --> D[4. Chạy Collection Runner]
    D --> E[5. Export & Newman CLI]
    E --> F[6. Tích hợp CI/CD]
```

### Chi tiết 6 bước thực hiện

| Bước | Mô tả | Mục đích |
|------|-------|----------|
| **1** | Tạo Collection để quản lý các API theo module | Tổ chức và phân nhóm API |
| **2** | Xây dựng các API tương ứng cần kiểm thử | Định nghĩa endpoints |
| **3** | Viết Test Script bằng JavaScript | Tự động hóa kiểm tra |
| **4** | Thực thi kiểm thử cục bộ bằng Collection Runner | Validate locally |
| **5** | Export Collection & Environment, chạy bằng Newman CLI | Automation outside Postman |
| **6** | Tích hợp vào CI/CD pipeline (GitLab CI, Jenkins...) | Tự động hóa sau deploy |

## 2. XÂY DỰNG TEST SCRIPT TRONG POSTMAN

### 2.1 Các hàm cơ bản thường sử dụng

#### Hàm `pm.test()` - Định nghĩa test case

```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});
```

#### Hàm `pm.expect()` - Thực hiện assertion

```javascript
// So sánh giá trị thực tế và giá trị mong đợi
pm.expect(expected_error).to.eql(true);
```

#### Đối tượng `pm.response` - Thông tin response

```javascript
// Lấy JSON response
let jsonData = pm.response.json();

// Lấy status code
let status = pm.response.code;

// Lấy thời gian phản hồi
let responseTime = pm.response.responseTime;
```

#### Biến môi trường - `pm.environment` và `pm.variables`

```javascript
// Lưu biến vào environment
pm.environment.set("token", "abc123");

// Lấy biến từ environment
let token = pm.environment.get("token");
```

> **Phân biệt:**
> - `pm.environment`: Sử dụng cho nhiều request trong cùng environment
> - `pm.variables`: Phạm vi hẹp hơn, dùng trong một request hoặc collection

---

### 2.2 Các dạng kiểm thử phổ biến

#### Kiểm tra Status Code

```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});
```

#### Kiểm tra nội dung Response

```javascript
pm.test("Response has success = true", function () {
    let jsonData = pm.response.json();
    pm.expect(jsonData.success).to.eql(true);
});
```

#### Kiểm tra thời gian phản hồi

```javascript
pm.test("Response time < 2000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});
```

---

### 2.3 Các hàm xử lý dữ liệu và điều hướng

#### Hàm `pm.iterationData.get()` - Lấy dữ liệu từ iteration data file

```javascript
// Lấy dữ liệu từ CSV/JSON file được cung cấp qua --iteration-data
const username = pm.iterationData.get("username");
const email = pm.iterationData.get("email");

// Sử dụng trong request body hoặc test script
pm.variables.set("dynamicUsername", username);
```

#### Hàm `pm.request` - Truy cập thông tin request

```javascript
// Lấy request body (raw JSON)
const requestBody = JSON.parse(pm.request.body.raw);

// Lấy query parameters
const queryParams = pm.request.url.query.all();

// Lấy headers
const authHeader = pm.request.headers.get("Authorization");

// Lấy URL
const requestUrl = pm.request.url.toString();
```

#### Hàm `postman.setNextRequest()` - Điều hướng request tiếp theo

```javascript
// Chuyển đến request cụ thể trong collection
postman.setNextRequest("Login API");

// Bỏ qua các request còn lại và dừng execution
postman.setNextRequest(null);
```

#### Hàm `pm.collectionVariables` - Quản lý biến cấp Collection

```javascript
// Lưu biến vào collection
pm.collectionVariables.set("userId", 12345);

// Lấy biến từ collection
const userId = pm.collectionVariables.get("userId");

// Xóa biến khỏi collection
pm.collectionVariables.unset("userId");

// Xóa tất cả biến trong collection
pm.collectionVariables.clear();
```

#### Hàm `pm.response.to.have.*` - Các assertion phổ biến

```javascript
// Kiểm tra status code
pm.test("Status is 200", () => pm.response.to.have.status(200));
pm.test("Status is success", () => pm.response.to.be.success);
pm.test("Status is client error", () => pm.response.to.be.clientError);
pm.test("Status is server error", () => pm.response.to.be.serverError);

// Kiểm tra body
pm.test("Body contains 'success'", () => {
    pm.response.to.have.body("success");
});

// Kiểm tra JSON property
pm.test("Has userId property", () => {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property("userId");
});
```

#### Hàm `pm.expect()` - Các assertion nâng cao 

```javascript
const jsonData = pm.response.json();

// So sánh bằng
pm.expect(jsonData.status).to.eql("success");
pm.expect(jsonData.count).to.equal(10);

// So sánh lớn hơn/nhỏ hơn
pm.expect(jsonData.age).to.be.above(18);
pm.expect(jsonData.score).to.be.below(100);
pm.expect(jsonData.limit).to.be.at.most(50);
pm.expect(jsonData.minimum).to.be.at.least(1);

// Kiểm tra null/undefined
pm.expect(jsonData.optionalField).to.be.null;
pm.expect(jsonData.requiredField).to.not.be.undefined;

// Kiểm tra boolean
pm.expect(jsonData.isActive).to.be.true;
pm.expect(jsonData.isDeleted).to.be.false;

// Kiểm tra string
pm.expect(jsonData.name).to.be.a("string");
pm.expect(jsonData.name).to.include("John");
pm.expect(jsonData.email).to.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/);

// Kiểm tra array
pm.expect(jsonData.items).to.be.an("array");
pm.expect(jsonData.items).to.have.lengthOf(5);
pm.expect(jsonData.items).to.include("item1");
pm.expect(jsonData.items).to.be.empty;

// Kiểm tra object
pm.expect(jsonData.data).to.be.an("object");
pm.expect(jsonData.data).to.have.all.keys("id", "name", "email");
pm.expect(jsonData.data).to.have.any.keys("id", "name");
```


#### Hàm `console.log()` - Debug và logging

```javascript
const jsonData = pm.response.json();

console.log("Response Data:", jsonData);
console.log("Status Code:", pm.response.code);
console.log("Response Time:", pm.response.responseTime);
console.log("Environment:", pm.environment.name);
console.log("Collection:", pm.collectionName);

// Log trong Pre-request Script
console.log("Request URL:", pm.request.url.toString());
console.log("Request Method:", pm.request.method);
```

---

## 3. KIẾN THỨC JAVASCRIPT CẦN THIẾT

### 3.1 Khai báo biến

```javascript
var a = 1;      // Function-scoped
let b = 2;      // Block-scoped, có thể gán lại
const c = 3;    // Block-scoped, không thể gán lại
```

### 3.2 Cấu trúc điều kiện

```javascript
if (response.error === true) {
    pm.expect(expected_error).to.eql(true);
    pm.expect(
        (response.toastMessage || "").normalize(),
        "toastMessage phải chứa nội dung chính"
    ).to.include(expected_toastmessage_1);
    
    postman.setNextRequest(null);  // Dừng execution
    return;
}
```

### 3.3 Khai báo hàm

```javascript
function sum(a, b) {
    return a + b;
}
```

### 3.4 Xử lý JSON

```javascript
let jsonData = pm.response.json();
console.log(jsonData.username);
```

### 3.5 Test Script hoàn chỉnh - Ví dụ thực tế

```javascript
// Kiểm tra thời gian phản hồi
pm.test("Response time is acceptable (under 3 seconds)", function () {
    pm.expect(pm.response.responseTime).to.be.below(3000);
});

// Parse request body và lưu vào collection variables
const requestBody = JSON.parse(pm.request.body.raw);

pm.collectionVariables.set("entitytypeid", Number(requestBody.entityTypeId));
pm.collectionVariables.set("entityid", Number(requestBody.entityId));
pm.collectionVariables.set("companyid", Number(requestBody.companyId));
pm.collectionVariables.set("channel", requestBody.channel);

// Xử lý mảng listPurposeOfConsent
const listPurpose = (requestBody.listPurposeOfConsent || [])
    .map(p => Number(p.purposeId));

// Loại bỏ phần tử trùng lặp
const uniqueExpected = [...new Set(listPurpose)];
pm.collectionVariables.set("listpurpose", JSON.stringify(uniqueExpected));

console.log("listpurpose =", pm.collectionVariables.get("listpurpose"));
```

---

## 4. THỰC THI BẰNG NEWMAN

### 4.1 Cài đặt Newman

Newman là CLI tool của Postman, cho phép chạy Collection từ command line:

```bash
# Cài đặt toàn cục
npm install -g newman
```

### 4.2 Cài đặt Reporter

Reporter tạo báo cáo HTML chi tiết sau khi chạy:

```bash
npm install -g newman-reporter-htmlextra
```

### 4.3 Chạy Collection với Newman

```bash
# Chạy collection với data file CSV và xuất HTML report
newman run "./consent.json" \
    --iteration-data "./consent_data.csv" \
    -r htmlextra
```

#### Các tham số quan trọng

| Tham số | Mô tả |
|---------|-------|
| `run` | File Collection JSON cần chạy |
| `--iteration-data` | File dữ liệu (CSV/JSON) cho data-driven testing |
| `-r htmlextra` | Sử dụng htmlextra reporter để tạo HTML report |

---

## 5. TÍCH HỢP CI/CD

### 5.1 Quy trình tích hợp

Sau khi chạy thành công với Newman, Collection có thể được tích hợp vào các hệ thống CI/CD:

```yaml
# Ví dụ GitLab CI
stages:
  - test

api_test:
  stage: test
  script:
    - npm install -g newman
    - npm install -g newman-reporter-htmlextra
    - newman run "./consent.json" --iteration-data "./consent_data.csv" -r htmlextra
  artifacts:
    paths:
      - newman/
    expire_in: 1 week
```

### 5.2 Lợi ích của CI/CD integration

- **Tự động chạy test sau mỗi lần deploy**
- **Báo cáo kết quả ngay lập tức**
- **Phát hiện lỗi sớm trong pipeline**
- **Theo dõi chất lượng API theo thời gian**

### 5.3 Điểm yếu và thách thức của CI/CD integration

| Điểm yếu | Mô tả |
|----------|-------|
| **Độ phức tạp cao** | Yêu cầu kiến thức về cả Postman, Newman và CI/CD pipeline | 
| **Thời gian chạy test lâu** | Collection lớn có thể làm chậm pipeline | 
| **Flaky tests** | Test pass/fail không ổn định do network, timing | 
| **Maintenance overhead** | Test scripts cần update khi API thay đổi | 

> **Lưu ý quan trọng:** Để giảm thiểu các điểm yếu trên, nên bắt đầu với pipeline đơn giản, sau đó mở rộng dần khi team đã quen với quy trình automation.
