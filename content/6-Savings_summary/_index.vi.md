---
title : "Saving summary"
date : "`r Sys.Date()`"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

### Spot Savings

Cho đến nay, chúng tôi đã khởi chạy Spot Instance theo một số cách. Đây là một câu hỏi: Bạn nghĩ chúng ta đã tiết kiệm được bao nhiêu cho workloads này?

Bạn có thể kiểm tra xem mình đã tiết kiệm được bao nhiêu với Spot Instance bằng cách đi tới bảng **Savings Summary**. Để xem khoản tiết kiệm của bạn, hãy làm như sau:

1. Mở Amazon EC2 console tại [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/).
2. Trong ngăn điều hướng, chọn **Spot Requests**.
3. Ở góc trên bên phải của màn hình, chọn **Savings summary**

![Saving](/images/s1/104.png) 

### Spot price history & Spot notifications

Có nhiều API hơn mà bạn có thể sử dụng để tìm hiểu thêm về Spot.

Một số dự án, như [the EC2 Spot Interruption Dashboard](https://github.com/aws-samples/ec2-spot-interruption-dashboard) có thể được sử dụng làm điểm ban đầu để hiểu Spot Instance nào đang bị chấm dứt và điều chỉnh cấu hình của bạn bằng cách tăng cường đa dạng hóa.

Một số API khác có thể hữu ích để hiểu giá Spot thay đổi như thế nào theo thời gian. Bạn cũng có thể xem thông tin này trong AWS Console bằng cách:

1. Mở Amazon EC2 console tại [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/).
2. Trong ngăn điều hướng, chọn **Spot Requests**.
3. Chọn **Pricing history** ở góc trên cùng bên phải.

Nếu quan tâm đến cách sử dụng Spot API theo chương trình, bạn có thể sử dụng [describe-spot-price-history](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-spot-price-history.html) để truy xuất thông tin bạn cần.

### Công cụ quản lý chi phí

Bạn cũng có thể xem Spot Savings bằng cách sử dụng [AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/), có giao diện dễ sử dụng cho phép bạn trực quan hóa, hiểu rõ và quản lý chi phí cũng như mức sử dụng AWS của bạn theo thời gian, bao gồm cả Spot Instances. Bạn có thể sử dụng Cost Explorer được lọc theo “Tùy chọn mua hàng” để xem mẫu số tiền bạn chi tiêu cho Phiên bản Spot theo thời gian và xem xu hướng mà bạn có thể sử dụng để hiểu chi phí của mình. Bạn có thể xem dữ liệu trong 12 tháng qua và dự báo ba tháng tiếp theo.

![Cost_explorer](/images/s1/104.png) 

Khách hàng AWS có quyền truy cập vào dữ liệu sử dụng và chi phí thô thông qua báo cáo AWS Cost and Usage (AWS CUR). Những báo cáo này chứa thông tin toàn diện nhất về mức sử dụng và chi phí AWS của bạn. Nếu bạn đang sử dụng Spot Instance cho nhu cầu điện toán của mình thì AWS CUR sẽ điền vào các cột giá sử dụng Spot của Amazon EC2 và các cột sản phẩm. Với dữ liệu này, bạn có thể tính toán mức tiết kiệm trước đây đạt được với Spot thông qua AWS CUR.