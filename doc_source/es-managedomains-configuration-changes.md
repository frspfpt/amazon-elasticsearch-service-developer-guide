# Making configuration changes in Amazon ES<a name="es-managedomains-configuration-changes"></a>

Amazon ES uses a *blue/green* deployment process when updating domains\. Blue/green typically refers to the practice of running two production environments, one live and one idle, and switching the two as you make software changes\. In the case of Amazon ES, it refers to the practice of creating a new environment for domains and routing users to the new environment after those updates are complete\. The practice minimizes downtime and maintains the original environment in the event that deployment to the new environment is unsuccessful\.

## Changes that cause blue/green deployments<a name="es-managedomains-bg"></a>

The following operations cause blue/green deployments:
+ Service software updates
+ Changing instance type
+ If your domain *doesn't* have dedicated master nodes, changing data instance count
+ Enabling or disabling dedicated master nodes
+ Changing dedicated master node count or dedicated master node instance type
+ Enabling or disabling Multi\-AZ
+ Changing storage type, volume type, or volume size
+ Choosing different VPC subnets
+ Adding or removing VPC security groups
+ Enabling or disabling Amazon Cognito authentication for Kibana
+ Choosing a different Amazon Cognito user pool or identity pool
+ Modifying advanced settings
+ Enabling or disabling the publication of error logs, audit logs, or slow logs to CloudWatch
+ Upgrading to a new Elasticsearch version
+ Enabling or disabling **Require HTTPS**
+ Enabling encryption of data at rest or node\-to\-node encryption
+ Enabling or disabling UltraWarm or cold storage
+ Disabling auto\-tune and rolling back its changes

## Changes that don't cause blue/green deployments<a name="es-managedomains-nobg"></a>

In *most* cases, the following operations do not cause blue/green deployments:
+ Changing access policy
+ Changing the automated snapshot hour
+ Enabling auto\-tune or disabling it without rolling back its changes
+ If your domain has dedicated master nodes, changing data node or UltraWarm node count

There are some exceptions\. For example, if you haven't reconfigured your domain since the launch of three Availability Zone support, Amazon ES might perform a one\-time blue/green deployment to redistribute your dedicated master nodes across Availability Zones\.

## Initiating a configuration change<a name="es-managedomains-initiate"></a>

When you initiate a configuration change, the domain state changes to **Processing** until Amazon ES has created a new environment with the latest [service software](es-service-software.md), at which point it changes back to **Active**\. During certain service software updates, the state remains **Active** the whole time\. In both cases, you can review the cluster health and Amazon CloudWatch metrics and see that the number of nodes in the cluster temporarily increases—often doubling—while the domain update occurs\. In the following illustration, you can see the number of nodes doubling from 11 to 22 during a configuration change and returning to 11 when the update is complete\.

![\[Number of nodes doubling from 11 to 22 during a domain configuration change.\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/images/NodesDoubled.png)

This temporary increase can strain the cluster's [dedicated master nodes](es-managedomains-dedicatedmasternodes.md), which suddenly might have many more nodes to manage\. It's important to maintain sufficient capacity on dedicated master nodes to handle the overhead that is associated with these blue/green deployments\.

**Important**  
You do *not* incur any additional charges during configuration changes and service maintenance\. You are billed only for the number of nodes that you request for your cluster\. For specifics, see [Charges for configuration changes](#es-managedomains-config-charges)\.

To prevent overloading dedicated master nodes, you can [monitor usage with the Amazon CloudWatch metrics](es-managedomains-cloudwatchmetrics.md)\. For recommended maximum values, see [Recommended CloudWatch alarms for Amazon Elasticsearch Service](cloudwatch-alarms.md)\.

## Charges for configuration changes<a name="es-managedomains-config-charges"></a>

If you change the configuration for a domain, Amazon ES creates a new cluster as described in [Making configuration changes in Amazon ES](#es-managedomains-configuration-changes)\. During the migration of old to new, you incur the following charges:
+ If you change the instance type, you're charged for both clusters for the first hour\. After the first hour, you're only charged for the new cluster\. EBS volumes aren't charged twice because they're part of your cluster, so their billing follows instance billing\.

  **Example:** You change the configuration from three `m3.xlarge` instances to four `m4.large` instances\. For the first hour, you are charged for both clusters \(3 \* `m3.xlarge` \+ 4 \* `m4.large`\)\. After the first hour, you are charged only for the new cluster \(4 \* `m4.large`\)\.
+ If you don’t change the instance type, you are charged only for the largest cluster for the first hour\. After the first hour, you are charged only for the new cluster\.

  **Example:** You change the configuration from six `m3.xlarge` instances to three `m3.xlarge` instances\. For the first hour, you are charged for the largest cluster \(6 \* `m3.xlarge`\)\. After the first hour, you are charged only for the new cluster \(3 \* `m3.xlarge`\)\.