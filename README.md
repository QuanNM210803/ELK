# ELK Stack Setup - Hướng dẫn sử dụng

Logs đang được ghi từ Filebeat → Logstash → Elasticsearch.

---

## 📊 Truy cập các dịch vụ

- **Elasticsearch**: http://localhost:9200
- **Kibana**: http://localhost:5601
- **Logstash**: Port 5044 (internal)

**Thông tin đăng nhập:**
- Username: `elastic`
- Password: `123456`

---

## ✅ Kiểm tra logs trong Kibana

1. Mở trình duyệt và truy cập: http://localhost:5601
2. Đăng nhập với username: `elastic`, password: `123456`
3. Vào **Menu ≡** → **Discover**
4. Tạo Data View:
   - Click "Create data view"
   - Index pattern: `cvconnect-logs-*`
   - Timestamp field: `@timestamp`
   - Click "Save data view to Kibana"
5. Bạn sẽ thấy tất cả logs từ các containers!

---

## 🔍 Lọc logs theo container cụ thể

### Cách 1: Lọc trong Kibana (Khuyến nghị)
Sau khi vào Discover, thêm filter:
- Field: `container.name`
- Operator: `is`
- Value: `user-service-cvconnect`

### Cách 2: Lọc trong Filebeat
Nếu bạn chỉ muốn gửi logs từ container cụ thể:

1. Sửa file `filebeat/filebeat.yml`:
```yaml
processors:
  - add_docker_metadata: {}
  - drop_event:  # Bỏ comment dòng này
      when:
        not:
          or:
            - equals:
                container.name: "user-service-cvconnect"
```

2. Restart Filebeat:
```powershell
docker-compose restart filebeat
```

---

## 🔧 Các lệnh hữu ích

### Kiểm tra trạng thái containers
```powershell
docker-compose ps
```

### Xem logs của các service
```powershell
# Filebeat logs
docker logs filebeat --tail 50

# Logstash logs
docker logs logstash --tail 50

# Elasticsearch logs
docker logs elasticsearch --tail 50
```

### Restart toàn bộ stack
```powershell
docker-compose restart
```

### Stop và xóa tất cả (cẩn thận!)
```powershell
docker-compose down -v
```

### Khởi động lại
```powershell
docker-compose up -d
```

---

## 📝 Cấu trúc Index trong Elasticsearch

Logs sẽ được lưu với index pattern:
```
cvconnect-logs-YYYY.MM
```

Ví dụ: `cvconnect-logs-2026.01`

---

## ⚙️ Cấu hình hiện tại

### Filebeat (filebeat/filebeat.yml)
- Thu thập logs từ TẤT CẢ Docker containers
- Thêm Docker metadata (container name, image, etc.)
- Gửi đến Logstash port 5044

### Logstash (logstash/logstash.conf)
- Nhận logs từ Filebeat (port 5044)
- Output console (rubydebug) để debug
- Ghi vào Elasticsearch index `cvconnect-logs-YYYY.MM`

### Elasticsearch
- Single node
- Security enabled với password
- Port 9200

### Kibana
- Kết nối với Elasticsearch
- Port 5601

---

## 🚨 Troubleshooting

### Không thấy logs trong Kibana?

1. Kiểm tra Filebeat có kết nối được Logstash không:
```powershell
docker logs filebeat --tail 20
```
Tìm dòng: "Connection to backoff(async(tcp://logstash:5044)) established"

2. Kiểm tra Logstash có nhận logs không:
```powershell
docker logs logstash --tail 50
```
Tìm dòng: "Sending a new message for the listener"

3. Kiểm tra indices trong Elasticsearch:
```powershell
docker exec elasticsearch curl -u elastic:123456 "http://localhost:9200/_cat/indices?v"
```

### Container không start?
```powershell
# Xem lỗi cụ thể
docker logs <container_name>

# Restart container
docker-compose restart <service_name>
```

### Muốn reset toàn bộ?
```powershell
# Stop và xóa tất cả (bao gồm data)
docker-compose down -v

# Start lại
docker-compose up -d

# Đợi 30 giây cho Elasticsearch khởi động
Start-Sleep -Seconds 30
```

---

## 📌 Lưu ý quan trọng

1. **Filter trong filebeat.yml**: Field phải là `container.name` KHÔNG phải `docker.container.name`
2. **Logstash startup**: Mất khoảng 30-40 giây để Logstash khởi động hoàn toàn
3. **Elasticsearch password**: Đã được set sẵn là `123456` trong docker-compose.yml
4. **Logs retention**: Cấu hình hiện tại lưu logs vô thời hạn, bạn nên cấu hình Index Lifecycle Management (ILM) sau

---

## 📞 Liên hệ

Nếu có vấn đề, kiểm tra:
1. Tất cả containers đang chạy: `docker-compose ps`
2. Logs của từng service
3. Network connectivity giữa các containers

---

Chúc bạn monitoring thành công! 🎉
