---
title : "Khởi tạo một launch template"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

Là bước tiên quyết trước khi tạo nhóm Auto Scaling EC2 hoặc EC2 Fleet, bạn tạo một launch template. Launch template cho phép bạn xác định các tham số khởi chạy để bạn không phải chỉ định chúng mỗi khi khởi chạy một inctance mới. Nó bao gồm ID của Amazon Machine Image (AMI), loại phiên bản, cặp khóa, nhóm bảo mật và các tham số khác dùng để khởi chạy instance EC2.

{{% notice note %}}
Trong hội thảo này, bạn sử dụng VPC mặc định của tài khoản của mình để tạo phiên bản.
{{% /notice %}}

### Khởi tạo một launch template

1. Tạo tệp cấu hình launch-template-data.json trong CloudShell bằng cách chạy các lệnh bên dưới. Khi kiểm tra cấu hình, bạn nhận thấy rằng tham số ImageId có giá trị thay thế là %ami-id% và UserData chứa các bước cài đặt Webserver được mã hóa base64. Sau khi tạo mẫu khởi chạy, bạn có thể mô tả nó và giải mã dữ liệu người dùng để xem nội dung.

```
cat << EOF > launch-template-data.json
{
  "ImageId": "%ami-id%",
  "TagSpecifications": [
    {
      "ResourceType": "instance",
      "Tags": [
        {
          "Key": "Name",
          "Value": "mySpotWorkshop"
        }
      ]
    }
  ],
  "UserData": "I2Nsb3VkLWNvbmZpZwpyZXBvX3VwZGF0ZTogdHJ1ZQpyZXBvX3VwZ3JhZGU6IGFsbAoKcGFja2FnZXM6CiAgLSBodHRwZAogIC0gY3VybAoKcnVuY21kOgogIC0gWyBzaCwgLWMsICJhbWF6b24tbGludXgtZXh0cmFzIGluc3RhbGwgLXkgZXBlbCIgXQogIC0gWyBzaCwgLWMsICJ5dW0gLXkgaW5zdGFsbCBzdHJlc3MtbmciIF0KICAtIFsgc2gsIC1jLCAiZWNobyBoZWxsbyB3b3JsZC4gTXkgaW5zdGFuY2UtaWQgaXMgJChjdXJsIC1zIGh0dHA6Ly8xNjkuMjU0LjE2OS4yNTQvbGF0ZXN0L21ldGEtZGF0YS9pbnN0YW5jZS1pZCkuIE15IGluc3RhbmNlLXR5cGUgaXMgJChjdXJsIC1zIGh0dHA6Ly8xNjkuMjU0LjE2OS4yNTQvbGF0ZXN0L21ldGEtZGF0YS9pbnN0YW5jZS10eXBlKS4gPiAvdmFyL3d3dy9odG1sL2luZGV4Lmh0bWwiIF0KICAtIFsgc2gsIC1jLCAic3lzdGVtY3RsIGVuYWJsZSBodHRwZCIgXQogIC0gWyBzaCwgLWMsICJzeXN0ZW1jdGwgc3RhcnQgaHR0cGQiIF0KCgo="
}
EOF
```

2. Biến %ami-id% nên được thay thế bằng AMI Amazon Linux 2 mới nhất. Bạn lấy ID AMI của Amazon Linux 2 mới nhất và cập nhật Id AMI trong tệp cấu hình của mình bằng cách chạy các lệnh sau.

```
# Đầu tiên, command này tìm kiếm Amazon Linux 2 AMI mới nhất
export ami_id=$(aws ec2 describe-images --owners amazon --filters "Name=name,Values=amzn2-ami-kernel*gp2" "Name=virtualization-type,Values=hvm" "Name=root-device-type,Values=ebs" --query "sort_by(Images, &CreationDate)[-1].ImageId" --output text)

sed -i.bak -e "s#%instanceProfile%#$instanceProfile#g" -e "s#%ami-id%#$ami_id#g" launch-template-data.json
```

3. Tạo launch template bằng cấu hình bạn vừa cập nhật. Bạn có thể kiểm tra các tham số khác được hỗ trợ bởi mẫu khởi chạy tại [đây](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-launch-template.html).

```
aws ec2 create-launch-template --launch-template-name TemplateForWebServer --launch-template-data file://launch-template-data.json
```

Kết quả ví dụ

```
{
    "LaunchTemplate": {
        "LaunchTemplateId": "lt-0dbe89c266217df42",
        "LaunchTemplateName": "TemplateForWebServer",
        "CreateTime": "2024-04-11T13:11:47+00:00",
        "CreatedBy": "arn:aws:iam::905418000290:user/AdminUser",
        "DefaultVersionNumber": 1,
        "LatestVersionNumber": 1
    }
}
```

4. Hãy xem tập lệnh dữ liệu người dùng được định cấu hình trên launch template để hiểu những gì được cài đặt trên các phiên bản trong khi khởi động.

```
aws ec2 describe-launch-template-versions  --launch-template-name TemplateForWebServer --output json | jq -r '.LaunchTemplateVersions[].LaunchTemplateData.UserData' | base64 --decode
```

Kết quả ví dụ

```
#cloud-config
repo_update: true
repo_upgrade: all

packages:
  - httpd
  - curl

runcmd:
  - [ sh, -c, "amazon-linux-extras install -y epel" ]
  - [ sh, -c, "yum -y install stress-ng" ]
  - [ sh, -c, "echo hello world. My instance-id is $(curl -s http://169.254.169.254/latest/meta-data/instance-id). My instance-type is $(curl -s http://169.254.169.254/latest/meta-data/instance-type). > /var/www/html/index.html" ]
  - [ sh, -c, "systemctl enable httpd" ]
  - [ sh, -c, "systemctl start httpd" ]
```

5. Bước cuối cùng của phần này, bạn sẽ thực hiện lệnh gọi API bổ sung để truy xuất mã nhận dạng của launch template vừa được tạo và lưu trữ nó trong một biến môi trường. Bạn sử dụng ID mẫu khởi chạy này khi tạo nhóm Auto Scaling, Nhóm EC2 và Nhóm Spot.

```
export LAUNCH_TEMPLATE_ID=$(aws ec2 describe-launch-templates --filters Name=launch-template-name,Values=TemplateForWebServer | jq -r '.LaunchTemplates[0].LaunchTemplateId')
```

Tốt lắm! Bạn đã tạo một launch template và lưu trữ vào các biến môi trường tất cả các chi tiết mà bạn cần để tham khảo nó trong các bước tiếp theo.