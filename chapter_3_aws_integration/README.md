# 3. Integration with AWS 
Refs: 
- https://docs.datadoghq.com/integrations/amazon_web_services/?tab=roledelegation
- https://docs.datadoghq.com/integrations/amazon_web_services/?tab=roledelegation#automatic---cloudformation



![alt text](../imgs/aws_integration.png "")


There are three ways to configure AWS integration, and we will use the easiest option.

## 3.1 Automatic - CloudFormation
- Open the Datadog AWS integration tile. Click the Install button to install this integration.
- Under the Configuration tab, choose Automatically Using CloudFormation. If you already have an attached AWS account, click Add another account first. - If you add another account, give it a different name than the IAM Role you have already registered, because specifying the same name results in access denial.
- Login to the AWS console.
- On the CloudFormation page:
    - Provide your Datadog API key.
    - If you would like to enable Resource Collection (required for some products and features), you must set the CloudSecurityPostureManagementPermissions parameter to “true”.
    - Check the two acknowledgement boxes at the bottom
    - Create a new stack
- Update the Datadog AWS integration tile with the IAM role name and account ID used to create the CloudFormation stack.



## 3.2 ELB Health and Performance Metrics
Ref: https://www.datadoghq.com/blog/top-elb-health-and-performance-metrics/#elb-metrics

