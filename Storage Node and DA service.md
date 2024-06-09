# Nút Lưu Trữ và Dịch Vụ DA

README này cung cấp hướng dẫn chi tiết để triển khai và hoàn thiện hệ thống 0G, bao gồm nhiều thành phần, mỗi thành phần có chức năng riêng.

## Điều Kiện Tiên Quyết
Các dịch vụ lưu trữ và DA 0G tương tác với các hợp đồng trên chuỗi để xác nhận gốc blob và khai thác PoRA.

### Hợp Đồng Trên Môi Trường Testnet
- **Flow Contract**: `0x22C1CaF8cbb671F220789184fda68BfD7eaA2eE1`
- **Mine Contract**: `0x8B9221eE2287aFBb34A7a1Ef72eB00fdD853FFC2`

*Lưu ý: Tutorial này sử dụng trên môi trường Linux, cụ thể là Ubuntu 22.04 LTS.*

## Storage Node

### Bước 1: Cài Đặt Dependencies
```
sudo apt-get update
sudo apt-get install clang cmake build-essential
```
### Bước 2: Cài Đặt Rustup
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
### Bước 3: Cài Đặt Go
```
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
```
### Bước 4: Tải Xuống Source Code
```
git clone https://github.com/0glabs/0g-storage-node.git
```
### Bước 5: Build Từ Source Code
```
cd 0g-storage-node
git submodule update --init

# Build ở chế độ release
cargo build --release
```
### Bước 6: Cập Nhật File Cấu Hình
Cập nhật file run/config.toml như sau:

```
# cổng p2p
network_libp2p_port

# endpoint rpc
rpc_listen_address

# các nút peer, chúng tôi cung cấp hai nút, bạn có thể sửa đổi thành các địa chỉ IP của riêng bạn
network_boot_nodes = ["/ip4/54.219.26.22/udp/1234/p2p/16Uiu2HAmPxGNWu9eVAQPJww79J32pTJLKGcpjRMb4Qb8xxKkyuG1","/ip4/52.52.127.117/udp/1234/p2p/16Uiu2HAm93Hd5azfhkGBbkx1zero3nYHvfjQYM2NtiW4R3r5bE2g"]

# địa chỉ hợp đồng flow
log_contract_address

# địa chỉ hợp đồng mine
mine_contract_address

# endpoint rpc của blockchain layer one
blockchain_rpc_endpoint

# block number để bắt đầu đồng bộ
log_sync_start_block_number

# vị trí cho db, network logs
db_dir
network_dir

# thiết lập hai trường này nếu bạn muốn trở thành thợ mỏ
# id thợ mỏ của bạn, có thể là chuỗi hex bất kỳ dài 64 ký tự
# không bao gồm 0x dẫn đầu
# cần thiết lập một id duy nhất, nếu không sẽ gây lỗi khi bạn gửi tx nhận thưởng
miner_id
# khóa riêng của bạn dài 64 ký tự
# không bao gồm 0x dẫn đầu
# không bỏ qua số 0 dẫn đầu
miner_key
```

### Bước 7: Chạy Service
Mở tmux để chạy nền

```
Sao chép mã
tmux
cd run
../target/release/zgs_node --config config.toml
```
## Dịch Vụ Lưu Trữ KV

### Bước 1: Cài Đặt Dependencies

Cài đặt các dependencies, Go và Rust tương tự như phần Storage Node.

### Bước 2: Tải Xuống Mã Nguồn

```
git clone https://github.com/0glabs/0g-storage-kv.git
```
### Bước 3: Build Mã Nguồn
```
cd 0g-storage-kv
git submodule update --init

# Build ở chế độ release
cargo build --release
```
### Bước 4: Cập Nhật File Cấu Hình
Copy config_example.toml thành config.toml và cập nhật các tham số.
```
# endpoint rpc
rpc_listen_address
# địa chỉ IP của dịch vụ lưu trữ, phân tách bằng dấu ","
zgs_node_urls = "http://ip1:port1,http://ip2:port2,..."

# endpoint rpc của blockchain layer one
blockchain_rpc_endpoint

# địa chỉ hợp đồng flow
log_contract_address

# block number để bắt đầu đồng bộ, nên đồng bộ với cấu hình trong dịch vụ lưu trữ
log_sync_start_block_number
```
### Bước 5: Chạy Dịch Vụ KV
Chạy trong tmux để dịch vụ chạy nền
```
tmux
cd run
../target/release/zgs_kv --config config.toml
```
Lưu ý: Cấu hình hệ thống đề xuất giống như nút lưu trữ.

## Data Availability service
### Bước 1: Cài Đặt Dependencies
Cài đặt các dependencies, Go và Rust tương tự như phần Storage Node.

### Bước 2: Tải Mã Nguồn
```
git clone https://github.com/0glabs/0g-data-avail.git
```
### Bước 3: Dịch Vụ Phân Tán
Cập nhật file 0g-data-avail/disperser/Makefile

Cấu Hình Encoder
```
# cổng grpc
--disperser-encoder.grpc-port 34000

# cổng metric
--disperser-encoder.metrics-http-port 9109

# số lượng worker, có thể dựa vào số lõi
--kzg.num-workers

# yêu cầu đồng thời tối đa
--disperser-encoder.max-concurrent-requests

# kích thước pool yêu cầu, có thể lớn hơn số lõi
--disperser-encoder.request-pool-size
```
Cấu Hình Batcher
```
# endpoint rpc của blockchain layer one
--chain.rpc

# khóa riêng của tài khoản ví, có thể đặt làm biến môi trường
--chain.private-key

# sửa đổi giới hạn gas cho các chuỗi khác nhau
--chain.gas-limit

# giới hạn kích thước batch, có thể là một số lớn như 1000
--batcher.batch-size-limit

# số lượng segment để tải lên trong một yêu cầu rpc
--batcher.storage.upload-task-size

# khoảng thời gian để phân tán tính cuối cùng
--batcher.finalizer-interval

# cấu hình aws, có thể đặt làm biến môi trường
--batcher.aws.region
--batcher.aws.access-key-id
--batcher.aws.secret-access-key
--batcher.s3-bucket-name
--batcher.dynamodb-table-name

# endpoints của dịch vụ lưu trữ, đối với nhiều endpoints, phân tách từng cái một
--batcher.storage.node-url
--batcher.storage.node-url

# endpoint của dịch vụ kv
--batcher.storage.kv-url

# địa chỉ hợp đồng flow
--batcher.storage.flow-contract

# timeout cho mã hóa, thiết lập dựa trên công suất instance
--encoding-timeout 10s
```
Cấu Hình Disperser
```
# cổng để lắng nghe yêu cầu
--disperser-server.grpc-port

# cấu hình aws, có thể đặt làm biến môi trường
# lưu ý rằng các key khác với batcher
--disperser-server.aws.region
--disperser-server.aws.access-key-id
--disperser-server.aws.secret-access-key
--disperser-server.s3-bucket-name
--disperser-server.dynamodb-table-name
```
### Bước 4: Build Mã Nguồn
```
cd 0g-data-avail/disperser
make build
```
### Bước 5: Chạy Encoder, Batcher và Disperser
```
# encoder
make run_encoder

# batcher
make run_batcher

# disperser
make run_server
```
### Bước 6: Chạy Dịch Vụ Kết Hợp
```
make run_combined
```
Lưu ý: Cấu hình cho máy chủ kết hợp giống như các máy chủ riêng biệt ngoại trừ việc tiền tố của một số tham số được thiết lập thành combined-server. Vui lòng tham khảo tệp Makefile để biết các cấu hình chi tiết.

Bây giờ chúng tôi cũng cung cấp một tùy chọn để sử dụng bộ nhớ làm cơ sở dữ liệu metadata thay vì AWS DynamoDB. Đặt --combined-server.use-memory-db để chỉ định cơ sở dữ liệu bạn muốn sử dụng.
## Dịch Vụ Truy Xuất
Cập nhật file 0g-data-avail/retriever/Makefile
```
# cổng grpc để lắng nghe yêu cầu
--retriever-server.grpc-port

# địa chỉ endpoint của dịch vụ kv
--retriever-server.storage-kv-url

# địa chỉ endpoint của dịch vụ lưu trữ
--retriever-server.storage-url
```
### Bước 1: Build Mã Nguồn
```
cd 0g-data-avail/retriever
make build
```
### Bước 2: Chạy Dịch Vụ Truy Xuất
```
make run
```
Lưu ý: Cấu hình hệ thống đề xuất giống như nút lưu trữ.


## Đóng Góp và Phát Triển

### Yêu Cầu Hệ Thống
Đảm bảo hệ thống của bạn đáp ứng các yêu cầu sau:

Hệ điều hành: Ubuntu 22.04 LTS hoặc tương đương
Bộ xử lý: Tối thiểu 4 lõi, đề xuất 8 lõi
Bộ nhớ: Tối thiểu 16GB RAM, đề xuất 32GB RAM
Lưu trữ: SSD với tối thiểu 500GB dung lượng trống
Kết nối mạng: Băng thông cao với độ trễ thấp

### Quy Trình Đóng Góp
Fork repository.
Tạo branch cho tính năng hoặc bugfix của bạn.
Commit và push các thay đổi lên branch của bạn.
Tạo pull request để yêu cầu review và merge.

### Hướng Dẫn Code Style
Tuân thủ các quy tắc về code style của dự án:
Sử dụng 4 khoảng trắng để thụt lề.
Viết các hàm và biến bằng tiếng Anh, sử dụng camelCase cho biến và PascalCase cho tên hàm.
Viết các comment rõ ràng, mô tả chức năng và mục đích của các đoạn code phức tạp.
### Liên Hệ
Nếu có bất kỳ câu hỏi hoặc yêu cầu hỗ trợ, vui lòng liên hệ với chúng tôi qua email: support@0glabs.io
