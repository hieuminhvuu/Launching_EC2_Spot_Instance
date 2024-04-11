---
title : "Chạy EC2 spot instances thông qua EC2 auto scaling group"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

{{% notice note %}}
Khi sử dụng EC2 Spot Instances, chúng tôi khuyên bạn nên xem xét nhóm Amazon EC2 Auto Scaling (ASG) vì nhóm này cung cấp các tính năng EC2 cập nhật mới nhất như lựa chọn loại phiên bản dựa trên thuộc tính, tái cân bằng công suất, chính sách thay đổi quy mô và nhiều chức năng khác.
{{% /notice %}}

[Amazon EC2 Auto Scaling groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-groups.html) chứa một tập hợp các Phiên bản Amazon EC2 được coi là một nhóm logic nhằm mục đích quản lý và điều chỉnh quy mô tự động. Các nhóm Auto Scaling cũng cho phép bạn sử dụng các tính năng Auto Scaling của Amazon EC2 như thay thế kiểm tra tình trạng và chính sách mở rộng. Cả việc duy trì số lượng phiên bản trong nhóm Auto Scaling và tự động thay đổi quy mô đều là chức năng cốt lõi của dịch vụ Auto Scaling của Amazon EC2.

{{% notice info %}}
Trước đây, các nhóm Auto Scaling đã sử dụng cấu hình khởi chạy (launch configurations). Các ứng dụng sử dụng cấu hình khởi chạy sẽ di chuyển sang các launch templates để bạn có thể tận dụng các tính năng mới nhất. Với các launch templates, bạn có thể cung cấp năng lực cho nhiều loại phiên bản bằng cách sử dụng cả Spot Instances và On-Demand Instances để đạt được quy mô, hiệu suất và tối ưu hóa chi phí như mong muốn.
{{% /notice %}}

### Sử dụng lựa chọn loại phiên bản dựa trên thuộc tính và các nhóm phiên bản hỗn hợp

Việc linh hoạt phiên bản là một phương pháp quan trọng nhất của Spot - [Spot best practice](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-best-practices.html), bạn có thể sử dụng lựa chọn loại phiên bản dựa trên thuộc tính - [attribute-based instance type selection (ABIS)](https://docs.aws.amazon.com/autoscaling/ec2/userguide/create-mixed-instances-group-attribute-based-instance-type-selection.html) để tự động chọn nhiều loại phiên bản phù hợp với yêu cầu của mình. Một trường hợp phổ biến khi sử dụng nhóm Auto Scaling là sử dụng nhóm này với khối lượng công việc yêu cầu kết hợp dung lượng Spot và On-Demand.

Ở bước này, bạn tạo tệp json để tạo nhóm Auto Scaling bằng AWS CLI. Cấu hình này sử dụng launch template mà bạn đã tạo ở các bước trước và ABIS để chọn mọi loại phiên bản không có GPU thế hệ hiện tại có 2 vCPU và không có giới hạn về bộ nhớ. OnDemandBaseCapacity cho phép bạn đặt công suất ban đầu là 1 On-Demand Instances. Dung lượng còn lại là sự kết hợp giữa 25% On-Demand Instances và 75% Spot Instances được xác định bởi OnDemandPercentageAboveBaseCapacity.

```
cat <<EoF > ./asg-policy.json
{
   "LaunchTemplate":{
      "LaunchTemplateSpecification":{
         "LaunchTemplateId":"${LAUNCH_TEMPLATE_ID}",
         "Version":"1"
      },
      "Overrides":[{
         "InstanceRequirements": {
            "VCpuCount": {
               "Min": 2, 
               "Max": 2
            },
            "MemoryMiB": {
               "Min": 0
            },
            "CpuManufacturers": [
               "intel",
               "amd"
            ],
            "InstanceGenerations": [
               "current"
            ],
            "AcceleratorCount": {
               "Max": 0
            }
         }
      }]
   },
   "InstancesDistribution":{
      "OnDemandBaseCapacity":1,
      "OnDemandPercentageAboveBaseCapacity":25,
      "SpotAllocationStrategy":"price-capacity-optimized"
   }
}
EoF
```

{{% notice info %}}
Trong cấu hình này, bạn đặt SpotAllocationStrategy thành mức giá được tối ưu hóa theo khả năng. Chiến lược phân bổ được tối ưu hóa theo công suất giá sẽ phân bổ các phiên bản từ nhóm Spot Instance có mức giá thấp và độ khả dụng công suất cao. Bạn có thể đọc thêm về chiến lược phân bổ được tối ưu hóa theo khả năng về giá trong bài đăng trên blog [Giới thiệu chiến lược phân bổ được tối ưu hóa theo khả năng về giá cho Phiên bản Spot EC2](https://aws.amazon.com/blogs/compute/introducing-price-capacity-optimized-allocation-strategy-for-ec2-spot-instances/).
{{% /notice %}}

1. Chạy các lệnh sau để truy xuất VPC mặc định của bạn và sau đó là các subnet của nó.

```
export VPC_ID=$(aws ec2 describe-vpcs --filters Name=isDefault,Values=true | jq -r '.Vpcs[0].VpcId')
export SUBNETS=$(aws ec2 describe-subnets --filters Name=vpc-id,Values="${VPC_ID}")
export SUBNET_1=$((echo $SUBNETS) | jq -r '.Subnets[0].SubnetId')
export SUBNET_2=$((echo $SUBNETS) | jq -r '.Subnets[1].SubnetId')
```

2. Chạy các lệnh sau để tạo nhóm Tự động thay đổi quy mô trên 2 Vùng sẵn sàng, kích thước tối thiểu 2, kích thước tối đa 20 và 10 đơn vị vCPU có công suất mong muốn.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name EC2SpotWorkshopASG --min-size 2 --max-size 20 --desired-capacity 10 --desired-capacity-type vcpu --vpc-zone-identifier "${SUBNET_1},${SUBNET_2}" --capacity-rebalance --mixed-instances-policy file://asg-policy.json
```

Bây giờ bạn đã tạo một nhóm Auto Scaling hỗn hợp!