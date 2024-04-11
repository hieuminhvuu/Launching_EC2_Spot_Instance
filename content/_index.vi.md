---
title : "Khởi chạy EC2 Spot Instance"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
---
# Khởi chạy EC2 Spot Instance

### Tổng quan

Trong workshop này, chúng ta tìm hiểu về các nguyên tắc cơ bản của EC2 Spot Instance và các công cụ được đề xuất để khởi chạy EC2 Spot Instance, kiểm tra khả năng phục hồi và xem lịch sử giá cả.

Bạn đóng vai một kỹ sư DevOps, người được cung cấp một ứng dụng mẫu và được giao nhiệm vụ triển khai ứng dụng đó một cách tiết kiệm chi phí bằng cách sử dụng các nhóm EC2 Auto Scaling và Nhóm EC2 trong Khu vực AWS tối ưu. Sau khi triển khai, bạn mô phỏng tình trạng gián đoạn của Phiên bản dùng ngay để kiểm tra khả năng phục hồi của các nhóm Tự động thay đổi quy mô và cuối cùng tính toán mức tiết kiệm chi phí ước tính.

### Dự kiến thời gian và chi phí để tổ chức workshop này

Thời gian ước tính để hoàn thành workshop là 45 đến 60 phút. Chi phí ước tính là dưới 5 đô la.

### Nội dung
 1. [Bắt đầu workshop](1-Starting_the_workshop)
 2. [Tạo launch template](2-Creating_a_launch_template)
 3. [Chạy EC2 spot instances thông qua EC2 auto scaling group](3-Launching_EC2_spot_instances_via_EC2_auto_scaling_group)
 4. [Chạy EC2 spot instances thông qua EC2 fleet](4-Launching_EC2_spot_instance_via_EC2_fleet)
 5. [Tạo một điểm gián đoạn](5-Creating_a_spot_interruption_experiment)
 6. [Tổng hợp về tiết kiệm](6-Savings_summary)
 7. [Spot placement score](7-Spot_placement_score)
 8. [Spot Blueprint](8-Spot_Blueprints)
 9. [Dọn dẹp tài nguyên](9-Clean_up)