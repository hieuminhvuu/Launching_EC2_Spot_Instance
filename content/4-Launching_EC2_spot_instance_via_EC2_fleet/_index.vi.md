---
title : "Chạy EC2 spot thông qua via EC2 fleet"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

EC2 Fleet cung cấp API cho phép vận hành và cung cấp với các biện pháp kiểm soát khá chi tiết. Nhóm EC2 chứa thông tin cấu hình để khởi chạy fleet hoặc group of instances. Khi sử dụng EC2 Fleet, bạn có thể xác định các mục tiêu công suất On-Demand và Spot riêng biệt, chỉ định loại phiên bản hoạt động tốt nhất cho ứng dụng của bạn và chỉ định cách Amazon EC2 sẽ phân bổ công suất fleet của bạn trong từng mô hình mua hàng.

Workloads có thể hưởng lợi từ API EC2 Fleet nằm trong số các nhà điều phối công suất riêng biệt khác triển khai logic được điều chỉnh và tối ưu hóa để cung cấp công suất. Chỉ cần nêu tên một số dự án sau đây sử dụng EC2 Fleet để quản lý năng lực:

- [Karpenter](https://github.com/aws/karpenter-provider-aws). Karpenter là Công cụ chia tỷ lệ tự động của cụm Kubernetes. Nó quản lý vòng đời của node. Nó quan sát các pod đến và khởi chạy các instances phù hợp cho tình huống đó.
- [Atlassian Escalator](https://github.com/atlassian/escalator), một Bộ chia tỷ lệ tự động cụm Kubernetes khác. Được thiết kế cho workloads theo lô hoặc theo công việc lớn mà không thể bị ép buộc và di chuyển khi cụm cần thu nhỏ quy mô.

### EC2 Fleet ví dụ : Áp dụng đa dạng hóa phiên bản trên workloads HPC được kết hợp chặt chẽ với chế độ tức thì EC2 Fleet

Trong phần workshop này, bạn giải quyết workload công việc chung mà EC2 Fleet mang lại sẽ mang lại lợi ích khi chạy.

{{% notice warning %}}
Lưu ý rằng khi sử dụng Spot Instances, hầu hết MPI workloads, đặc biệt là những workloads chạy hàng giờ và không sử dụng điểm kiểm tra, đều không phù hợp với Spot Instances. Hãy nhớ rằng Spot Instances phù hợp với các ứng dụng có dung sai cao, có thể phục hồi sau khi mất và thay thế một hoặc nhiều phiên bản.
{{% /notice %}}

Trong phần workshop này, bạn yêu cầu EC2 Fleet bằng cách sử dụng loại yêu cầu **instant**, đây là một tính năng chỉ có trong EC2 Fleet. Bằng cách đó, EC2 Fleet sẽ đưa ra yêu cầu đồng bộ một lần cho công suất mà bạn mong muốn. Trong phản hồi API, nó trả về các phiên bản đã khởi chạy, cùng với lỗi đối với những phiên bản không thể khởi chạy. Thông tin thêm về các loại yêu cầu [tại đây](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-fleet-configuration-strategies.html#ec2-fleet-request-type).

{{% notice info %}}
HPC workloads được kết hợp chặt chẽ thường bị suy giảm hiệu năng khi các phiên bản trong cụm thuộc các họ phiên bản và kích cỡ khác nhau (ví dụ: c5.large so với c4.large hoặc c5.large so với c5.xlarge). Đặc điểm khác của workloads này là tất cả các instance phải ở gần nhau (lý tưởng nhất là trong cùng một [placement group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html)). Để đáp ứng những hạn chế này, bạn đặt cấu hình yêu cầu nhóm với cùng loại instance (ví dụ: c5.large) trong một Availability Zone duy nhất. Nếu ứng dụng HPC của bạn được liên kết lỏng lẻo và bạn có thể loại bỏ các ràng buộc này và thay vào đó hãy sử dụng các nhóm Auto Scaling.
{{% /notice %}}

1. Tạo tệp cấu hình để khởi chạy EC2 Fleet với [attribute-based instance type selection (ABIS)](https://docs.aws.amazon.com/autoscaling/ec2/userguide/create-mixed-instances-group-attribute-based-instance-type-selection.html). Run the following:

```
cat <<EoF > ./ec2-fleet-config.json
{
   "SpotOptions":{
      "SingleInstanceType": true,
      "SingleAvailabilityZone": true,
      "MinTargetCapacity": 4,
      "AllocationStrategy": "price-capacity-optimized",
      "InstanceInterruptionBehavior": "terminate"
   },
   "OnDemandOptions":{
      "AllocationStrategy": "lowest-price",
      "SingleInstanceType": true,
      "SingleAvailabilityZone": true,
      "MinTargetCapacity": 0
   },
   "LaunchTemplateConfigs":[
      {
         "LaunchTemplateSpecification":{
            "LaunchTemplateId":"${LAUNCH_TEMPLATE_ID}",
            "Version":"1"
         },
         "Overrides":[{
            "InstanceRequirements": {
               "VCpuCount": {
                  "Min": 2, 
                  "Max": 4
               },
               "MemoryMiB": {
                  "Min": 0
               },
               "CpuManufacturers": [
                  "intel"
               ]
            }
         }]
      }
   ],
   "TargetCapacitySpecification":{
      "TotalTargetCapacity": 4,
      "OnDemandTargetCapacity": 0,
      "DefaultTargetCapacityType": "spot"
   },
   "Type":"instant"
}
EoF
```

- Yêu cầu EC2 Fleet chỉ định riêng công suất mục tiêu cho Spot Instances và On-Demand Instances bằng cách sử dụng các trường OnDemandTargetCapacity và SpotTargetCapacity bên trong cấu trúc TargetCapacitySpecification. Giá trị của DefaultTargetCapacityType chỉ định nên sử dụng Spot Instances hay On-Demand Instances để đáp ứng TotalTargetCapacity.

- Bằng cách đặt SingleInstanceType và SingleAvailabilityZone thành true, bạn đang buộc EC2 Fleet yêu cầu cung cấp tất cả các instances trong cùng một Availability Zone và cùng loại.

1. Sao chép và dán lệnh này để tạo EC2 Fleet và xuất mã định danh của nó sang một biến môi trường để theo dõi trạng thái của nhóm sau này.

```
export FLEET_ID=$(aws ec2 create-fleet --cli-input-json file://ec2-fleet-config.json | jq -r '.FleetId')
```

Bây giờ bạn đã tạo EC2 Fleet với instance loại yêu cầu!