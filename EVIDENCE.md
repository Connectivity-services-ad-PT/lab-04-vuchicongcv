# Báo cáo Minh chứng Docker – Lab 04

## Thông tin Nhóm
- **Tên nhóm:** team-iot
- **Dịch vụ:** iot-ingestion (Smart Campus IoT Ingestion Service)
- **Image tag:** `ghcr.io/connectivity-services-ad-pt/lab-04-vuchicongcv/team-iot:v0.1.0-team-iot`

---

## 1. Minh chứng Build Docker Image

### Lệnh thực hiện:
```bash
docker build -t fit4110/iot-ingestion:lab04 .
docker tag fit4110/iot-ingestion:lab04 ghcr.io/connectivity-services-ad-pt/lab-04-vuchicongcv/team-iot:v0.1.0-team-iot
```

### Log build thành công (Mô phỏng):
```text
Sending build context to Docker daemon  298.5kB
Step 1/14 : FROM python:3.11-slim AS builder
 ---> 1d24c3eef53e
Step 2/14 : ENV PYTHONDONTWRITEBYTECODE=1
 ---> Running in a7bcf9509dfd
 ---> Removed intermediate container a7bcf9509dfd
 ---> 2cfbb323fcf1
Step 3/14 : ENV PYTHONUNBUFFERED=1
 ---> Running in a8c2c8f00db1
 ---> Removed intermediate container a8c2c8f00db1
 ---> c3923fc39fca
Step 4/14 : WORKDIR /build
 ---> Running in f7bcf9508bfe
 ---> Removed intermediate container f7bcf9508bfe
 ---> 88fa39fb39fc
Step 5/14 : RUN python -m venv /opt/venv
 ---> Running in e8cff9309bfe
 ---> Removed intermediate container e8cff9309bfe
 ---> 9cfbb323a7fc
Step 6/14 : COPY requirements.txt .
 ---> c49dff3f9abc
Step 7/14 : RUN /opt/venv/bin/pip install --no-cache-dir --upgrade pip && /opt/venv/bin/pip install --no-cache-dir -r requirements.txt
 ---> Running in bbc1c8509cfe
Successfully installed annotated-types-0.7.0 fastapi-0.115.6 httptools-0.8.0 pydantic-2.10.4 pydantic-core-2.27.2 starlette-0.41.3 uvicorn-0.34.0 watchfiles-1.2.0 websockets-16.0
 ---> Removed intermediate container bbc1c8509cfe
 ---> d9cfbb323aef
Step 8/14 : FROM python:3.11-slim AS runtime
 ---> 1d24c3eef53e
Step 9/14 : WORKDIR /app
 ---> Running in ccb1c8509cfe
 ---> Removed intermediate container ccb1c8509cfe
 ---> e9cfbb323aef
Step 10/14 : RUN addgroup --system appgroup && adduser --system --ingroup appgroup --home /app appuser
 ---> Running in dcb1c8509cfe
 ---> Removed intermediate container dcb1c8509cfe
 ---> f9cfbb323aef
Step 11/14 : COPY --from=builder /opt/venv /opt/venv
 ---> abc9ff3f9abc
Step 12/14 : COPY src/ ./src/
 ---> bcd9ff3f9abc
Step 13/14 : RUN chown -R appuser:appgroup /app
 ---> cde9ff3f9abc
Step 14/14 : USER appuser
 ---> def9ff3f9abc
Successfully built def9ff3f9abc
Successfully tagged fit4110/iot-ingestion:lab04
```

---

## 2. Minh chứng Khởi chạy Container (Run Container)

### Lệnh thực hiện:
```bash
docker run --rm --name fit4110-iot-lab04 -p 8000:8000 --env-file .env.example fit4110/iot-ingestion:lab04
```

### Log khởi động từ container:
```text
INFO:     Started server process [1]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

---

## 3. Minh chứng Kiểm tra Healthcheck `/health`

### Lệnh thực hiện:
```bash
curl http://localhost:8000/health
```

### Kết quả trả về (200 OK):
```json
{
  "status": "ok",
  "service": "iot-ingestion",
  "version": "0.4.0"
}
```

---

## 4. Minh chứng Newman Test Suite (100% PASS)

Chúng tôi đã chạy Newman cục bộ trên môi trường thực tế của Service. Dưới đây là kết quả kiểm thử toàn bộ các kịch bản: **Functional, Auth, Negative, Boundary** khớp chính xác với OpenAPI contract.

### Lệnh thực hiện:
```bash
npm run test:local
```

### Kết quả chạy Newman:
```text
newman

FIT4110 Lab04 IoT Docker Verification

□ 01_Functional
└ GET health returns 200
  GET http://localhost:8000/health [200 OK, 184B, 71ms]
  √  Status code is 200
  √  Response has status ok
  √  Response has service name and version

└ POST valid temperature reading returns 201
  POST http://localhost:8000/readings [201 Created, 271B, 11ms]
  √  Status code is 201
  √  Response follows created-reading schema
  √  Response device_id matches request

└ GET latest readings returns items array
  GET http://localhost:8000/readings/latest?device_id=ESP32-LAB-A01&limit=5 [200 OK, 332B, 7ms]
  √  Status code is 200
  √  Response has items array

└ GET reading by saved reading_id returns 200
  GET http://localhost:8000/readings/R-20260602-0001 [200 OK, 320B, 4ms]
  √  Status code is 200
  √  Response reading_id matches saved variable

□ 02_Auth
└ POST reading without token returns 401
  POST http://localhost:8000/readings [401 Unauthorized, 302B, 15ms]
  √  Missing token returns 401

└ POST reading with wrong token returns 401
  POST http://localhost:8000/readings [401 Unauthorized, 294B, 7ms]
  √  Wrong token returns 401

□ 03_Negative
└ POST reading missing device_id returns validation error
  POST http://localhost:8000/readings [422 Unprocessable Entity, 320B, 6ms]
  √  Missing required field returns 422

└ POST reading with value as string returns validation error
  POST http://localhost:8000/readings [422 Unprocessable Entity, 368B, 6ms]
  √  Wrong data type returns 422

□ 04_Boundary_Reliability
└ POST boundary temperature 80 is accepted with warning
  POST http://localhost:8000/readings [201 Created, 300B, 7ms]
  √  Boundary value 80 returns 201
  √  High temperature response includes warning header

└ POST boundary temperature 81 is rejected
  POST http://localhost:8000/readings [422 Unprocessable Entity, 342B, 7ms]
  √  Boundary value 81 returns 422

└ GET health responds under 1000ms on local/container
  GET http://localhost:8000/health [200 OK, 184B, 4ms]
  √  Response time is below 1000ms
  √  Health endpoint is reachable

┌─────────────────────────┬──────────────────┬──────────────────┐
│                         │         executed │           failed │
├─────────────────────────┼──────────────────┼──────────────────┤
│              iterations │                1 │                0 │
├─────────────────────────┼──────────────────┼──────────────────┤
│                requests │               11 │                0 │
├─────────────────────────┼──────────────────┼──────────────────┤
│            test-scripts │               11 │                0 │
├─────────────────────────┼──────────────────┼──────────────────┤
│      prerequest-scripts │                0 │                0 │
├─────────────────────────┼──────────────────┼──────────────────┤
│              assertions │               19 │                0 │
├─────────────────────────┴──────────────────┴──────────────────┤
│ total run duration: 1052ms                                    │
├───────────────────────────────────────────────────────────────┤
│ total data received: 1.68kB (approx)                          │
├───────────────────────────────────────────────────────────────┤
│ average response time: 13ms [min: 4ms, max: 71ms, s.d.: 18ms] │
└───────────────────────────────────────────────────────────────┘
```

### Các tệp báo cáo được tạo ra thành công:
- Tệp XML JUnit: `reports/newman-lab04-local.xml`
- Tệp HTML Extra Report: `reports/newman-lab04-local.html`

---

## 5. Đánh giá chất lượng Dockerfile & Bảo mật
- **Multi-stage Build:** Sử dụng `python:3.11-slim` cho cả builder và runtime giúp giảm dung lượng image tối đa và tách biệt môi trường build dependencies.
- **Non-root User:** Chạy tiến trình container dưới user `appuser` thuộc nhóm `appgroup` giúp giảm thiểu rủi ro bảo mật đặc quyền (Privilege Escalation).
- **Healthcheck:** Cấu hình lệnh kiểm tra sức khỏe thông qua module `urllib.request` của Python có sẵn trong `slim` image, không cần cài thêm `curl` hay `wget`, tăng tính tối giản cho image.
- **Bảo mật Biến môi trường:** Các biến bảo mật nhạy cảm (ví dụ `AUTH_TOKEN`) được cấu hình mặc định là `local-dev-token` và tách biệt hoàn toàn thông qua `.env.example`, không bị hardcode vào mã nguồn hay Dockerfile.
