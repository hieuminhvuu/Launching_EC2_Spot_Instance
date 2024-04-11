---
title : "Dọn dẹp tài nguyên"
date : "`r Sys.Date()`"
weight : 9
chapter : false
pre : " <b> 9. </b> "
---

Trước khi kết thúc workshop, hãy đảm bảo rằng chúng ta dọn sạch tất cả tài nguyên mà chúng ta đã tạo để không phải gánh chịu những chi phí không mong muốn.

#### Xóa mẫu thử nghiệm AWS FIS

Khi bạn thực hiện xong các thử nghiệm FIS, bạn có thể xóa mẫu thử nghiệm.

```
aws fis delete-experiment-template --id $FIS_TEMPLATE_ID
```

#### Xoá Auto Scaling Group

Để xoá your Auto Scaling group sử dụng CLI

```
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name EC2SpotWorkshopASG --force-delete
```

#### Xoá EC2 Fleet

Khi kết thúc sử dụng EC2 Fleet, bạn có thể xoá EC2 Fleet(s) và chấm dứt tất cả instance.

Để xoá EC2 Fleet và chấm dứt các instance đang chạy sử dụng CLI

```
aws ec2 delete-fleets --fleet-ids "${FLEET_ID}" --terminate-instances
```

#### Chấm dứt Spot instances đã tạo bởi RunInstance

Bạn nhớ lại chúng tôi đã tạo Instance này bằng Name Tag cụ thể. Chúng tôi sẽ sử dụng name tag để tìm kiếm instance và sau đó chuyển instance-id tới lệnh gọi EC2 của Instance kết thúc.

```
export INSTANCE_ID=$(aws ec2 describe-instances --filters "Name=tag-value,Values=EC2SpotWorkshopRunInstance" --query "Reservations[0].Instances[0].InstanceId" | sed s/\"//g)
aws ec2 terminate-instances --instance-ids $INSTANCE_ID
```

#### Xoá Spot Fleet Request

Bây giờ hãy hủy hoặc xóa yêu cầu Spot Fleet. Bạn phải chỉ định liệu Spot Fleet có nên chấm dứt Spot Fleet của nó hay không. Nếu bạn chấm dứt các Instance, yêu cầu Spot Fleet sẽ chuyển sang trạng thái cancelled_terminating. Nếu không, yêu cầu Spot Fleet sẽ chuyển sang trạng thái cancel_running và các instance tiếp tục chạy cho đến khi chúng bị gián đoạn hoặc bạn chấm dứt chúng theo cách thủ công.

Để huỷ Spot Fleet request và chấm dứt instance đang chạy sử dụng CLI

```
aws ec2 cancel-spot-fleet-requests --spot-fleet-request-ids "${SPOT_FLEET_REQUEST_ID}" --terminate-instances
```

#### Xoá Launch Template

Cuối cùng, bây giờ tất cả các phiên bản đã bị chấm dứt, hãy xóa Launch Template.

```
aws ec2 delete-launch-template --launch-template-id "${LAUNCH_TEMPLATE_ID}"
```