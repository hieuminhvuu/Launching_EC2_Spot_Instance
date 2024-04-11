---
title : "Spot Blueprint"
date : "`r Sys.Date()`"
weight : 8
chapter : false
pre : " <b> 8. </b> "
---

Spot Blueprints là một chức năng được cung cấp trong Bảng điều khiển web AWS, trong phần Yêu cầu Spot, giúp tạo ra một số kiến trúc phổ biến nhất cho Spot, sử dụng Cơ sở hạ tầng làm mã và tuân thủ các biện pháp thực hành Spot Best. Có Bản thiết kế giao ngay cho các dịch vụ phổ biến nhất bao gồm Amazon EC2 Auto Scaling, Amazon EMR, AWS Batch và Amazon Elastic Kubernetes Service (Amazon EKS).

Spot Blueprints giúp bạn có bước khởi đầu thuận lợi trong việc sử dụng kiến trúc Spot bằng cách cung cấp trình tạo mẫu mã cơ sở hạ tầng dễ thực hiện, được thiết kế để thu thập các yêu cầu về khối lượng công việc của bạn, đồng thời giải thích và định cấu hình các biện pháp thực hành tốt nhất của Spot trong quá trình thực hiện. Đầu ra của quy trình là tệp Cloudformation hoặc Terraform IaaC (Cơ sở hạ tầng dưới dạng mã) mà bạn có thể sử dụng và điều chỉnh cho các dự án của mình.

##### Getting started with Spot Blueprints

1. Mở Amazon EC2 console at [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/).
2. Trong ngăn điều hướng, chọn **Spot Requests**.
3. Chọn Spot Blueprints ở góc trên bên phải.

![Spot Blueprints](/images/s1/108.png) 

Bạn sẽ thấy các danh mục khác nhau để bắt đầu. Từ đó, bạn có thể tải xuống bản thiết kế được định cấu hình sẵn ở định dạng AWS CloudFormation hoặc Terraform hoặc chọn định cấu hình bản thiết kế đó. Bạn có thể tìm hiểu thêm về Spot Blueprints bằng cách đọc [launch blog post](https://aws.amazon.com/blogs/compute/introducing-spot-blueprints-a-template-generator-for-frameworks-like-kubernetes-and-apache-spark/).