# Thông Tin Deploy — Checkpoint 5

> Service đã deploy trên Railway. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Railway

`railway.toml` build bằng Dockerfile, healthcheck `/health`, và chạy Uvicorn
trên `0.0.0.0:$PORT`. Railway cấp `PORT`; không tự tạo hoặc ghi đè biến này.

Có thể deploy bằng CLI:

```bash
railway login
railway init
railway add --database redis
railway up
railway domain
railway logs
```

Trong Railway, mở service agent → **Variables** và cấu hình:

| Biến | Nguồn / giá trị |
|------|------------------|
| `AGENT_API_KEY` | Tạo khóa ngẫu nhiên và lưu như secret trong dashboard; không commit vào repo hoặc ghi vào terminal history. |
| `REDIS_URL` | Tham chiếu biến URL của service Redis, ví dụ `${{Redis.REDIS_URL}}`; thay `Redis` bằng đúng tên service nếu khác. |
| `RATE_LIMIT_PER_MINUTE` | `10` |
| `MONTHLY_BUDGET_USD` | `10.0` |
| `LOG_LEVEL` | `INFO` |

Sau deploy, kiểm tra `/health` trả 200 và `/ready` trả 200. Nếu `/ready` trả 503,
xác nhận `REDIS_URL` trong service agent đang tham chiếu đúng Redis service.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Minh Thái |
| Mã học viên | 2A202602726 |
| Repo | https://github.com/aoi36/K4-L3B-Day12-NguyenMinhThai-2A202602726Cloud-Service-And-Deployment

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-production-68dc.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-09-29 |

## Biến Môi Trường Cloud

Sau khi cấu hình, ghi trạng thái và **nguồn giá trị**, không ghi secret:

| Biến | Trạng thái | Ghi chú |
|------|--------|---------|
| `PORT` | Platform cấp | Không tự ghi đè |
| `AGENT_API_KEY` | Đã cấu hình | Secret trong Railway Variables; không lưu giá trị trong repo |
| `REDIS_URL` | Đã cấu hình | Redis trên Railway; kết nối được xác nhận qua `/ready` |
| `RATE_LIMIT_PER_MINUTE` | Đã cấu hình | `10` |
| `MONTHLY_BUDGET_USD` | Đã cấu hình | `10.0` |
| `LOG_LEVEL` | Đã cấu hình | `INFO` |

## Lệnh Kiểm Tra

Các lệnh đã dùng để kiểm tra service:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://day12-agent-production-68dc.up.railway.app/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://day12-agent-production-68dc.up.railway.app/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://day12-agent-production-68dc.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://day12-agent-production-68dc.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-agent-production-68dc.up.railway.app/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

```text
GET /health: 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET /ready: 200 {"status":"ready","redis":true}
POST /ask without X-API-Key: 401 {"detail":"invalid or missing API key"}
Authenticated /ask and rate-limit burst: not recorded here; these checks require the API secret.
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

## Phương Án Dự Phòng

Không sử dụng; service đã deploy trực tiếp trên Railway.
