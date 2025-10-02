# Process Monitoring Stack

Một stack monitoring hoàn chỉnh sử dụng Process Exporter, Prometheus và Grafana để theo dõi hiệu suất và tài nguyên của các tiến trình trên hệ thống Linux.

## 🚀 Tính năng

- **Process Exporter**: Thu thập metrics chi tiết về các tiến trình từ `/proc`
- **Prometheus**: Lưu trữ và query metrics
- **Grafana**: Dashboard trực quan với các biểu đồ monitoring
- **Node Exporter**: Monitor tài nguyên hệ thống (optional)

## 📊 Metrics được thu thập

- **CPU Usage**: Thời gian CPU sử dụng theo nhóm tiến trình
- **Memory Usage**: RAM sử dụng (resident, virtual, swapped)
- **I/O Metrics**: Bytes đọc/ghi, page faults
- **File Descriptors**: Số lượng file descriptors mở
- **Context Switches**: Chuyển đổi ngữ cảnh
- **Thread Metrics**: Thông tin chi tiết về threads

## 🛠️ Cài đặt và chạy

### Yêu cầu hệ thống

- Docker và Docker Compose
- Hệ điều hành Linux (để truy cập `/proc`)
- Port 3000, 9090, 9256, 9100 (optional) phải available

### Chạy nhanh

```bash
# Clone repository
git clone <your-repo-url>
cd process-monitoring-stack

# Chạy tất cả services
docker compose up -d

# Kiểm tra trạng thái
docker compose ps
```

### Truy cập giao diện

- **Grafana Dashboard**: http://localhost:3000
  - Username: `admin`
  - Password: `admin123`
- **Prometheus**: http://localhost:9090
- **Process Exporter Metrics**: http://localhost:9256/metrics

## 📁 Cấu trúc dự án

```
process-monitoring-stack/
├── docker-compose.yml          # Cấu hình tất cả services
├── prometheus.yml              # Cấu hình Prometheus
├── grafana/
│   ├── provisioning/
│   │   ├── datasources/
│   │   │   └── prometheus.yml  # Cấu hình datasource
│   │   └── dashboards/
│   │       └── dashboard.yml   # Cấu hình dashboard
│   └── dashboards/
│       └── process-monitoring.json  # Dashboard mẫu
├── cmd/process-exporter/       # Source code process-exporter
├── collector/                  # Process collector logic
├── config/                     # Configuration handling
├── proc/                       # /proc filesystem reader
└── README.md
```

## ⚙️ Cấu hình

### Process Exporter

Mặc định sẽ monitor tất cả processes. Để tùy chỉnh, chỉnh sửa file `grafana/provisioning/datasources/prometheus.yml`:

```yaml
# Monitor specific processes
process_names:
  - name: "nginx"
    exe:
    - nginx
  - name: "postgres"
    exe:
    - postgres
  - name: "{{.Comm}}"  # Monitor all processes
    cmdline:
    - '.+'
```

### Prometheus

Cấu hình scrape targets trong `prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'process-exporter'
    static_configs:
      - targets: ['process-exporter:9256']
    scrape_interval: 5s
```

## 📈 Dashboard

Dashboard "Process Monitoring Dashboard" bao gồm:

1. **Process Count**: Số lượng tiến trình theo nhóm
2. **CPU Usage %**: % CPU sử dụng theo nhóm tiến trình
3. **Memory Usage**: RAM sử dụng (resident memory)
4. **I/O Rate**: Tốc độ đọc/ghi dữ liệu

## 🔧 Quản lý Services

```bash
# Xem logs
docker compose logs -f

# Restart service
docker compose restart process-exporter

# Dừng tất cả
docker compose down

# Dừng và xóa dữ liệu
docker compose down -v

# Rebuild và chạy lại
docker compose up -d --build
```

## 🐛 Troubleshooting

### Process Exporter không thu thập được metrics

```bash
# Kiểm tra logs
docker compose logs process-exporter

# Kiểm tra quyền truy cập /proc
docker compose exec process-exporter ls -la /host/proc
```

### Prometheus không scrape được metrics

```bash
# Kiểm tra targets
curl http://localhost:9090/api/v1/targets

# Kiểm tra metrics endpoint
curl http://localhost:9256/metrics
```

### Grafana không hiển thị data

1. Kiểm tra datasource connection
2. Verify Prometheus có data
3. Check dashboard queries

## 📝 Customization

### Thêm Dashboard mới

1. Tạo file JSON trong `grafana/dashboards/`
2. Restart Grafana: `docker compose restart grafana`

### Thêm Alert Rules

Tạo file `prometheus/alerts.yml` và thêm vào `prometheus.yml`:

```yaml
rule_files:
  - "alerts.yml"
```

### Monitor thêm services

Thêm service mới vào `docker-compose.yml` và cấu hình scrape trong `prometheus.yml`.

## 🤝 Đóng góp

1. Fork repository
2. Tạo feature branch
3. Commit changes
4. Push và tạo Pull Request

## 📄 License

MIT License - xem file [LICENSE](LICENSE) để biết thêm chi tiết.

## 🔗 Liên kết hữu ích

- [Process Exporter Documentation](https://github.com/ncabatoff/process-exporter)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
