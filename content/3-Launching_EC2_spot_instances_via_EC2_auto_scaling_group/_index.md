---
title : "Launching EC2 spot instances via EC2 auto scaling group"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

{{% notice note %}}
When adopting EC2 Spot Instances, we recommend you to consider Amazon EC2 Auto Scaling group (ASG) since it offers the most up to date EC2 features such as attribute-based instance type selection, capacity rebalancing, scaling policies and many more functionalities.
{{% /notice %}}

[Amazon EC2 Auto Scaling groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-groups.html) contain a collection of Amazon EC2 Instances that are treated as a logical grouping for the purposes of automatic scaling and management. Auto Scaling groups also enable you to use Amazon EC2 Auto Scaling features such as health check replacements and scaling policies. Both maintaining the number of instances in an Auto Scaling group and automatic scaling are the core functionality of the Amazon EC2 Auto Scaling service.

{{% notice info %}}
In the past, Auto Scaling groups used launch configurations. Applications using launch configurations should migrate to launch templates so that you can leverage the latest features. With launch templates you can provision capacity across multiple instance types using both Spot Instances and On-Demand Instances to achieve the desired scale, performance, and cost optimization.
{{% /notice %}}

### Using attribute-based instance type selection and mixed instance groups

Being instance flexible is an important [Spot best practice](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-best-practices.html), you can use [attribute-based instance type selection (ABIS)](https://docs.aws.amazon.com/autoscaling/ec2/userguide/create-mixed-instances-group-attribute-based-instance-type-selection.html) to automatically select multiple instance types matching your requirements. A common case when using Auto Scaling groups, is to use it with workloads that require a mix of Spot and On-Demand capacity.

In this step you create a json file for creating Auto Scaling groups using AWS CLI. The configuration uses the launch template that you created in the previous steps and ABIS to pick any current generation non-GPU instance types with 2 vCPU and no limit on memory. OnDemandBaseCapacity allows you to set an initial capacity of 1 On-Demand Instance. Remaining capacity is mix of 25% On-Demand Instances and 75% Spot Instances defined by the OnDemandPercentageAboveBaseCapacity.

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
In this configuration you set the SpotAllocationStrategy to price-capacity-optimized. The price-capacity-optimized allocation strategy allocates instances from the Spot Instance pools that offer low prices and high capacity availability. You can read more about the price-capacity-optimized allocation strategy in Introducing the price-capacity-optimized allocation strategy for EC2 Spot Instances blog post.
{{% /notice %}}

1. Run the following commands to retrieve your default VPC and then its subnets.

```
export VPC_ID=$(aws ec2 describe-vpcs --filters Name=isDefault,Values=true | jq -r '.Vpcs[0].VpcId')
export SUBNETS=$(aws ec2 describe-subnets --filters Name=vpc-id,Values="${VPC_ID}")
export SUBNET_1=$((echo $SUBNETS) | jq -r '.Subnets[0].SubnetId')
export SUBNET_2=$((echo $SUBNETS) | jq -r '.Subnets[1].SubnetId')
```

2. Run the following commands to create an Auto Scaling group across 2 Availability Zones, min-size 2, max-size 20, and desired-capacity 10 vCPU units.

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name EC2SpotWorkshopASG --min-size 2 --max-size 20 --desired-capacity 10 --desired-capacity-type vcpu --vpc-zone-identifier "${SUBNET_1},${SUBNET_2}" --capacity-rebalance --mixed-instances-policy file://asg-policy.json
```

You have now created a mixed instances Auto Scaling group!