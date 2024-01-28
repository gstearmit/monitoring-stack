# giám sát ngăn xếp

Ngăn xếp giám sát chứa ELK + Grafana + Prometheus (với nhiều nhà xuất khẩu) và xác thực thông qua proxy ngược (Caddy)

## Tổng quan

Kho lưu trữ này nhằm hỗ trợ triển khai một bộ công cụ để Hợp nhất và giám sát nhật ký.

Ngăn xếp giám sát này được cấu thành bởi các công cụ sau:

- [Elastic stack (trước đây gọi là ELK):](https://www.elastic.co/webinars/introduction-elk-stack) Bao gồm Elaticsearch, Logstash, Kibana và Beats. Mặc dù tất cả chúng đều được xây dựng để phối hợp cực kỳ hiệu quả với nhau, nhưng mỗi dự án là một dự án riêng biệt được điều hành bởi nhà cung cấp nguồn mở Elastic -- vốn ban đầu là nhà cung cấp nền tảng tìm kiếm doanh nghiệp.

    - [Elasticsearch:](https://www.elastic.co/products/elasticsearch) là một công cụ tìm kiếm dựa trên Lucene. Nó cung cấp một công cụ tìm kiếm toàn văn bản có khả năng phân tán, có nhiều đối tượng thuê với giao diện web HTTP và các tài liệu JSON không có lược đồ. Elaticsearch được phát triển bằng Java và được phát hành dưới dạng nguồn mở theo các điều khoản của Giấy phép Apache.

    - [Logstash:](https://www.elastic.co/products/logstash) Là một công cụ nguồn mở để thu thập, phân tích cú pháp và lưu trữ nhật ký để sử dụng trong tương lai. Kibana 3 là một giao diện web có thể được sử dụng để tìm kiếm và xem nhật ký mà Logstash đã lập chỉ mục. Cả hai công cụ này đều dựa trên Elaticsearch. Elaticsearch, Logstash và Kibana khi được sử dụng cùng nhau được gọi là ngăn xếp ELK.

    - [Kibana:](https://www.elastic.co/products/kibana) Là một plugin trực quan hóa dữ liệu nguồn mở cho Elaticsearch. Nó cung cấp khả năng trực quan hóa bên trên nội dung được lập chỉ mục trên cụm Elaticsearch. Người dùng có thể tạo các biểu đồ thanh, đường và phân tán hoặc biểu đồ hình tròn và bản đồ trên khối lượng dữ liệu lớn.

    - [Beats](https://www.elastic.co/products/beats): Là một tác nhân nhẹ của Elastic, rất tốt cho việc thu thập dữ liệu. Họ ngồi trên máy chủ của bạn và tập trung dữ liệu trong Elaticsearch. Ý tưởng là chúng tôi sử dụng nó để tăng cường khả năng xử lý, Beats cũng có thể gửi tới Logstash để chuyển đổi và phân tích cú pháp.

- [Prometheus](https://github.com/prometheus): Prometheus, một dự án của Cloud Native Computing Foundation, là một hệ thống giám sát hệ thống và dịch vụ. Nó thu thập số liệu từ các mục tiêu được định cấu hình theo các khoảng thời gian nhất định, đánh giá các biểu thức quy tắc, hiển thị kết quả và có thể kích hoạt cảnh báo nếu một số điều kiện được coi là đúng.

- [Node_Exporter:](https://github.com/prometheus/node_exporter) Trình xuất Prometheus cho các số liệu phần cứng và hệ điều hành được hiển thị bởi các hạt nhân \*NIX, được viết bằng Go với các bộ thu thập số liệu có thể cắm được.

- [Elasticsearch_Exporter:](https://github.com/justwatchcom/elasticsearch_exporter) Trình xuất Prometheus cho các số liệu khác nhau về ElasticSearch, được viết bằng Go.

- [CAdvisor:](https://github.com/google/cadvisor) (Container Advisor) cung cấp cho người dùng vùng chứa sự hiểu biết về cách sử dụng tài nguyên và đặc điểm hiệu suất của các vùng chứa đang chạy của họ. Nó là một daemon đang chạy có chức năng thu thập, tổng hợp, xử lý và xuất thông tin về các container đang chạy. Cụ thể, đối với mỗi vùng chứa, nó lưu giữ các tham số cách ly tài nguyên, mức sử dụng tài nguyên lịch sử, biểu đồ sử dụng tài nguyên lịch sử hoàn chỉnh và thống kê mạng. Dữ liệu này được xuất theo vùng chứa và trên toàn máy.

- [Grafana:](https://github.com/grafana/grafana) là bộ phân tích và trực quan hóa số liệu nguồn mở. Nó được sử dụng phổ biến nhất để trực quan hóa dữ liệu chuỗi thời gian cho phân tích ứng dụng và cơ sở hạ tầng nhưng nhiều người sử dụng nó trong các lĩnh vực khác bao gồm cảm biến công nghiệp, tự động hóa gia đình, thời tiết và kiểm soát quy trình.

- [Caddy:](https://github.com/stefanprodan/caddy-builder) Caddy là máy chủ web HTTP/2 có HTTPS tự động, bao gồm các plugin bổ sung để giúp nó hoạt động theo cách tương tự như Ingress/Proxy.

## Yêu cầu

- [Docker](https://docs.docker.com/engine/installation/)
- [Docker Compose](https://docs.docker.com/compose/install/)

## Ngành kiến ​​​​trúc

Sau đây là hình ảnh minh họa trực quan về cách tích hợp các công cụ khác nhau với nhau:

![sơ đồ](docs/architecture_diagram.png)

## Các bước triển khai trên Docker Swarm Cluster

**1.** Sau khi cài đặt `docker` và `docker-compose`, như được mô tả trong phần yêu cầu, hãy khởi tạo bầy đàn, như hiển thị ngay bên dưới:

``` vỏ
docker bầy init
```

Một đầu ra tương tự sẽ được hiển thị nếu lệnh thành công:

``` vỏ
Đã khởi tạo Swarm: nút hiện tại (a8pml3unconooa7t7qnsw7knp) hiện là người quản lý.

Để thêm một công nhân vào nhóm này, hãy chạy lệnh sau:

     docker bầy đàn tham gia --token SWMTKN-1-5jk292hj9b12by0byz4xfnntq07nd58ozrqxxyjx91kx03oqhw-8pl8cm17pj1lam8lq10gmuih0 192.168.122.60:2377

Để thêm người quản lý vào bầy này, hãy chạy 'trình quản lý mã thông báo tham gia docker bầy đàn' và làm theo hướng dẫn.
```

**2.** Sao chép kho lưu trữ này vào máy đích nơi ngăn xếp giám sát sẽ được triển khai:

``` vỏ
git clone git@github.com:fsilveir/monitoring-stack.git
```

**3.** Từ thư mục cơ sở của kho lưu trữ, đi vào thư mục `docker` và thực thi lệnh sau