---
title : "Launching EC2 spot instances via EC2 fleet"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

EC2 Fleet provides an API that allows to operate and procure capacity with quite granular controls. An EC2 Fleet contains the configuration information to launch a fleet or group of instances. Using EC2 Fleet, you can define separate On-Demand and Spot capacity targets, specify the instance types that work best for your applications, and specify how Amazon EC2 should distribute your fleet capacity within each purchasing model.

Workloads that can benefit from EC2 Fleet API are among other bespoke capacity orchestrators that implement tuned up and optimized logic to provision capacity. Just to name a few, the following projects use EC2 Fleet to manage capacity:

- [Karpenter](https://github.com/aws/karpenter-provider-aws). Karpenter is Kubernetes Cluster Autoscaler. It manages the node lifecycle. It observes incoming pods and launches the right instances for the situation.
- [Atlassian Escalator](https://github.com/atlassian/escalator), yet another Kubernetes Cluster Autoscaler. Designed for large batch or job based workloads that cannot be force-drained and moved when the cluster needs to scale down.

### EC2 Fleet example : Applying instance diversification on HPC tightly coupled workloads with EC2 Fleet instant mode

In this part of the workshop you tackle a common workload for with EC2 Fleet provides benefit when running.

{{% notice warning %}}
Note that while using Spot Instances, most of MPI workloads, specially those that run for hours and do not use checkpointing, are not appropriate for Spot Instances. Remember Spot Instances are suited for fault tolerant applications that can recover from the loss and replacement of one or more instances.
{{% /notice %}}

In this part of the workshop you request an EC2 Fleet using the request type **instant**, which is a feature only available in EC2 Fleet. By doing so, EC2 Fleet places a synchronous one-time request for your desired capacity. In the API response, it returns the instances that launched, along with errors for those instances that could not be launched. More information on request types [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-fleet-configuration-strategies.html#ec2-fleet-request-type).

{{% notice info %}}
Tightly coupled HPC workloads typically suffer from performance degradation when the instances in the cluster are of different instance families and sizes (i.e: c5.large vs c4.large or c5.large vs c5.xlarge). The other characteristic of this workload is that all the instances must be close together (ideally in the same [placement group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html)). To satisfy these constraints you configure the fleet request with same instance type (for example c5.large) in a single Availability Zone. If your HPC application is loosely coupled and you can remove these constraints and use Auto Scaling groups instead.
{{% /notice %}}

1. Create the configuration file to launch the EC2 Fleet with [attribute-based instance type selection (ABIS)](https://docs.aws.amazon.com/autoscaling/ec2/userguide/create-mixed-instances-group-attribute-based-instance-type-selection.html). Run the following:

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

- The EC2 Fleet request specifies separately the target capacity for Spot and On-Demand Instances using the OnDemandTargetCapacity and SpotTargetCapacity fields inside the TargetCapacitySpecification structure. The value for DefaultTargetCapacityType specifies whether Spot or On-Demand Instances should be used to meet the TotalTargetCapacity.

- By setting SingleInstanceType and SingleAvailabilityZone to true, you are forcing the EC2 Fleet request to provision all the instances in the same Availability Zone and of the same type.

2. Copy and paste this command to create the EC2 Fleet and export its identifier to an environment variable to later monitor the status of the fleet.

```
export FLEET_ID=$(aws ec2 create-fleet --cli-input-json file://ec2-fleet-config.json | jq -r '.FleetId')
```

You have now created an EC2 Fleet with request type instance!