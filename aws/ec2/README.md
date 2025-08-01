# AWS EC2 Instance Terraform module

Terraform module which creates an EC2 instance on AWS.

[![SWUbanner](https://raw.githubusercontent.com/vshymanskyy/StandWithUkraine/main/banner2-direct.svg)](https://github.com/vshymanskyy/StandWithUkraine/blob/main/docs/README.md)

## Usage

### Single EC2 Instance

```hcl
module "ec2_instance" {
  source  = "terraform-aws-modules/ec2-instance/aws"

  name = "single-instance"

  instance_type = "t2.micro"
  key_name      = "user1"
  monitoring    = true
  subnet_id     = "subnet-eddcdzz4"

  tags = {
    Terraform   = "true"
    Environment = "dev"
  }
}
```

### Multiple EC2 Instance

```hcl
module "ec2_instance" {
  source  = "terraform-aws-modules/ec2-instance/aws"

  for_each = toset(["one", "two", "three"])

  name = "instance-${each.key}"

  instance_type = "t2.micro"
  key_name      = "user1"
  monitoring    = true
  subnet_id     = "subnet-eddcdzz4"

  tags = {
    Terraform   = "true"
    Environment = "dev"
  }
}
```

### Spot EC2 Instance

```hcl
module "ec2_instance" {
  source  = "terraform-aws-modules/ec2-instance/aws"

  name = "spot-instance"

  create_spot_instance = true
  spot_price           = "0.60"
  spot_type            = "persistent"

  instance_type = "t2.micro"
  key_name      = "user1"
  monitoring    = true
  subnet_id     = "subnet-eddcdzz4"

  tags = {
    Terraform   = "true"
    Environment = "dev"
  }
}
```

## Examples

- [Complete EC2 instance](https://github.com/terraform-aws-modules/terraform-aws-ec2-instance/tree/master/examples/complete)
- [EC2 instance w/ private network access via Session Manager](https://github.com/terraform-aws-modules/terraform-aws-ec2-instance/tree/master/examples/session-manager)

## Make an encrypted AMI for use

This module does not support encrypted AMI's out of the box however it is easy enough for you to generate one for use

This example creates an encrypted image from the latest ubuntu 20.04 base image.

```hcl
provider "aws" {
  region = "us-west-2"
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["679593333241"]

  filter {
    name   = "name"
    values = ["ubuntu-minimal/images/hvm-ssd/ubuntu-focal-20.04-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

resource "aws_ami_copy" "ubuntu_encrypted_ami" {
  name              = "ubuntu-encrypted-ami"
  description       = "An encrypted root ami based off ${data.aws_ami.ubuntu.id}"
  source_ami_id     = data.aws_ami.ubuntu.id
  source_ami_region = "eu-west-2"
  encrypted         = true

  tags = { Name = "ubuntu-encrypted-ami" }
}

data "aws_ami" "encrypted-ami" {
  most_recent = true

  filter {
    name   = "name"
    values = [aws_ami_copy.ubuntu_encrypted_ami.id]
  }

  owners = ["self"]
}
```

## Conditional creation

The following combinations are supported to conditionally create resources:

```hcl
module "ec2_instance" {
  source  = "terraform-aws-modules/ec2-instance/aws"

  # Disable creation of EC2 and all resources
  create = false

  # Enable creation of spot instance
  create_spot_instance = true

  # Enable creation of EC2 IAM instance profile
  create_iam_instance_profile = true

  # Disable creation of security group
  create_security_group = false

  # Enable creation of elastic IP
  create_eip = true

  # ... omitted
}
```

## Notes

- `network_interface` can't be specified together with `vpc_security_group_ids`, `associate_public_ip_address`, `subnet_id`. See [complete example](https://github.com/terraform-aws-modules/terraform-aws-ec2-instance/tree/master/examples/complete) for details.
- In regards to spot instances, you must grant the `AWSServiceRoleForEC2Spot` service-linked role access to any custom KMS keys, otherwise your spot request and instances will fail with `bad parameters`. You can see more details about why the request failed by using the awscli and `aws ec2 describe-spot-instance-requests`

<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.5.7 |
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | >= 6.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | >= 6.0 |

## Modules

No modules.

## Resources

| Name | Type |
|------|------|
| [aws_ebs_volume.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ebs_volume) | resource |
| [aws_ec2_tag.spot_instance](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ec2_tag) | resource |
| [aws_eip.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/eip) | resource |
| [aws_iam_instance_profile.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_instance_profile) | resource |
| [aws_iam_role.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role) | resource |
| [aws_iam_role_policy_attachment.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role_policy_attachment) | resource |
| [aws_instance.ignore_ami](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance) | resource |
| [aws_instance.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance) | resource |
| [aws_security_group.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group) | resource |
| [aws_spot_instance_request.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/spot_instance_request) | resource |
| [aws_volume_attachment.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/volume_attachment) | resource |
| [aws_vpc_security_group_egress_rule.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc_security_group_egress_rule) | resource |
| [aws_vpc_security_group_ingress_rule.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc_security_group_ingress_rule) | resource |
| [aws_iam_policy_document.assume_role_policy](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_policy_document) | data source |
| [aws_partition.current](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/partition) | data source |
| [aws_ssm_parameter.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/ssm_parameter) | data source |
| [aws_subnet.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/subnet) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_ami"></a> [ami](#input\_ami) | ID of AMI to use for the instance | `string` | `null` | no |
| <a name="input_ami_ssm_parameter"></a> [ami\_ssm\_parameter](#input\_ami\_ssm\_parameter) | SSM parameter name for the AMI ID. For Amazon Linux AMI SSM parameters see [reference](https://docs.aws.amazon.com/systems-manager/latest/userguide/parameter-store-public-parameters-ami.html) | `string` | `"/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64"` | no |
| <a name="input_associate_public_ip_address"></a> [associate\_public\_ip\_address](#input\_associate\_public\_ip\_address) | Whether to associate a public IP address with an instance in a VPC | `bool` | `null` | no |
| <a name="input_availability_zone"></a> [availability\_zone](#input\_availability\_zone) | AZ to start the instance in | `string` | `null` | no |
| <a name="input_capacity_reservation_specification"></a> [capacity\_reservation\_specification](#input\_capacity\_reservation\_specification) | Describes an instance's Capacity Reservation targeting option | <pre>object({<br/>    capacity_reservation_preference = optional(string)<br/>    capacity_reservation_target = optional(object({<br/>      capacity_reservation_id                 = optional(string)<br/>      capacity_reservation_resource_group_arn = optional(string)<br/>    }))<br/>  })</pre> | `null` | no |
| <a name="input_cpu_credits"></a> [cpu\_credits](#input\_cpu\_credits) | The credit option for CPU usage (unlimited or standard) | `string` | `null` | no |
| <a name="input_cpu_options"></a> [cpu\_options](#input\_cpu\_options) | Defines CPU options to apply to the instance at launch time. | <pre>object({<br/>    amd_sev_snp      = optional(string)<br/>    core_count       = optional(number)<br/>    threads_per_core = optional(number)<br/>  })</pre> | `null` | no |
| <a name="input_create"></a> [create](#input\_create) | Whether to create an instance | `bool` | `true` | no |
| <a name="input_create_eip"></a> [create\_eip](#input\_create\_eip) | Determines whether a public EIP will be created and associated with the instance. | `bool` | `false` | no |
| <a name="input_create_iam_instance_profile"></a> [create\_iam\_instance\_profile](#input\_create\_iam\_instance\_profile) | Determines whether an IAM instance profile is created or to use an existing IAM instance profile | `bool` | `false` | no |
| <a name="input_create_security_group"></a> [create\_security\_group](#input\_create\_security\_group) | Determines whether a security group will be created | `bool` | `true` | no |
| <a name="input_create_spot_instance"></a> [create\_spot\_instance](#input\_create\_spot\_instance) | Depicts if the instance is a spot instance | `bool` | `false` | no |
| <a name="input_disable_api_stop"></a> [disable\_api\_stop](#input\_disable\_api\_stop) | If true, enables EC2 Instance Stop Protection | `bool` | `null` | no |
| <a name="input_disable_api_termination"></a> [disable\_api\_termination](#input\_disable\_api\_termination) | If true, enables EC2 Instance Termination Protection | `bool` | `null` | no |
| <a name="input_ebs_optimized"></a> [ebs\_optimized](#input\_ebs\_optimized) | If true, the launched EC2 instance will be EBS-optimized | `bool` | `null` | no |
| <a name="input_ebs_volumes"></a> [ebs\_volumes](#input\_ebs\_volumes) | Additional EBS volumes to attach to the instance | <pre>map(object({<br/>    encrypted            = optional(bool)<br/>    final_snapshot       = optional(bool)<br/>    iops                 = optional(number)<br/>    kms_key_id           = optional(string)<br/>    multi_attach_enabled = optional(bool)<br/>    outpost_arn          = optional(string)<br/>    size                 = optional(number)<br/>    snapshot_id          = optional(string)<br/>    tags                 = optional(map(string), {})<br/>    throughput           = optional(number)<br/>    type                 = optional(string, "gp3")<br/>    # Attachment<br/>    device_name                    = optional(string) # Will fall back to use map key as device name<br/>    force_detach                   = optional(bool)<br/>    skip_destroy                   = optional(bool)<br/>    stop_instance_before_detaching = optional(bool)<br/>  }))</pre> | `null` | no |
| <a name="input_eip_domain"></a> [eip\_domain](#input\_eip\_domain) | Indicates if this EIP is for use in VPC | `string` | `"vpc"` | no |
| <a name="input_eip_tags"></a> [eip\_tags](#input\_eip\_tags) | A map of additional tags to add to the eip | `map(string)` | `{}` | no |
| <a name="input_enable_primary_ipv6"></a> [enable\_primary\_ipv6](#input\_enable\_primary\_ipv6) | Whether to assign a primary IPv6 Global Unicast Address (GUA) to the instance when launched in a dual-stack or IPv6-only subnet | `bool` | `null` | no |
| <a name="input_enable_volume_tags"></a> [enable\_volume\_tags](#input\_enable\_volume\_tags) | Whether to enable volume tags (if enabled it conflicts with root\_block\_device tags) | `bool` | `true` | no |
| <a name="input_enclave_options_enabled"></a> [enclave\_options\_enabled](#input\_enclave\_options\_enabled) | Whether Nitro Enclaves will be enabled on the instance. Defaults to `false` | `bool` | `null` | no |
| <a name="input_ephemeral_block_device"></a> [ephemeral\_block\_device](#input\_ephemeral\_block\_device) | Customize Ephemeral (also known as Instance Store) volumes on the instance | <pre>map(object({<br/>    device_name  = string<br/>    no_device    = optional(bool)<br/>    virtual_name = optional(string)<br/>  }))</pre> | `null` | no |
| <a name="input_get_password_data"></a> [get\_password\_data](#input\_get\_password\_data) | If true, wait for password data to become available and retrieve it | `bool` | `null` | no |
| <a name="input_hibernation"></a> [hibernation](#input\_hibernation) | If true, the launched EC2 instance will support hibernation | `bool` | `null` | no |
| <a name="input_host_id"></a> [host\_id](#input\_host\_id) | ID of a dedicated host that the instance will be assigned to. Use when an instance is to be launched on a specific dedicated host | `string` | `null` | no |
| <a name="input_host_resource_group_arn"></a> [host\_resource\_group\_arn](#input\_host\_resource\_group\_arn) | ARN of the host resource group in which to launch the instances. If you specify an ARN, omit the `tenancy` parameter or set it to `host` | `string` | `null` | no |
| <a name="input_iam_instance_profile"></a> [iam\_instance\_profile](#input\_iam\_instance\_profile) | IAM Instance Profile to launch the instance with. Specified as the name of the Instance Profile | `string` | `null` | no |
| <a name="input_iam_role_description"></a> [iam\_role\_description](#input\_iam\_role\_description) | Description of the role | `string` | `null` | no |
| <a name="input_iam_role_name"></a> [iam\_role\_name](#input\_iam\_role\_name) | Name to use on IAM role created | `string` | `null` | no |
| <a name="input_iam_role_path"></a> [iam\_role\_path](#input\_iam\_role\_path) | IAM role path | `string` | `null` | no |
| <a name="input_iam_role_permissions_boundary"></a> [iam\_role\_permissions\_boundary](#input\_iam\_role\_permissions\_boundary) | ARN of the policy that is used to set the permissions boundary for the IAM role | `string` | `null` | no |
| <a name="input_iam_role_policies"></a> [iam\_role\_policies](#input\_iam\_role\_policies) | Policies attached to the IAM role | `map(string)` | `{}` | no |
| <a name="input_iam_role_tags"></a> [iam\_role\_tags](#input\_iam\_role\_tags) | A map of additional tags to add to the IAM role/profile created | `map(string)` | `{}` | no |
| <a name="input_iam_role_use_name_prefix"></a> [iam\_role\_use\_name\_prefix](#input\_iam\_role\_use\_name\_prefix) | Determines whether the IAM role name (`iam_role_name` or `name`) is used as a prefix | `bool` | `true` | no |
| <a name="input_ignore_ami_changes"></a> [ignore\_ami\_changes](#input\_ignore\_ami\_changes) | Whether changes to the AMI ID changes should be ignored by Terraform. Note - changing this value will result in the replacement of the instance | `bool` | `false` | no |
| <a name="input_instance_initiated_shutdown_behavior"></a> [instance\_initiated\_shutdown\_behavior](#input\_instance\_initiated\_shutdown\_behavior) | Shutdown behavior for the instance. Amazon defaults this to stop for EBS-backed instances and terminate for instance-store instances. Cannot be set on instance-store instance | `string` | `null` | no |
| <a name="input_instance_market_options"></a> [instance\_market\_options](#input\_instance\_market\_options) | The market (purchasing) option for the instance. If set, overrides the `create_spot_instance` variable | <pre>object({<br/>    market_type = optional(string)<br/>    spot_options = optional(object({<br/>      instance_interruption_behavior = optional(string)<br/>      max_price                      = optional(string)<br/>      spot_instance_type             = optional(string)<br/>      valid_until                    = optional(string)<br/>    }))<br/>  })</pre> | `null` | no |
| <a name="input_instance_tags"></a> [instance\_tags](#input\_instance\_tags) | Additional tags for the instance | `map(string)` | `{}` | no |
| <a name="input_instance_type"></a> [instance\_type](#input\_instance\_type) | The type of instance to start | `string` | `"t3.micro"` | no |
| <a name="input_ipv6_address_count"></a> [ipv6\_address\_count](#input\_ipv6\_address\_count) | A number of IPv6 addresses to associate with the primary network interface. Amazon EC2 chooses the IPv6 addresses from the range of your subnet | `number` | `null` | no |
| <a name="input_ipv6_addresses"></a> [ipv6\_addresses](#input\_ipv6\_addresses) | Specify one or more IPv6 addresses from the range of the subnet to associate with the primary network interface | `list(string)` | `null` | no |
| <a name="input_key_name"></a> [key\_name](#input\_key\_name) | Key name of the Key Pair to use for the instance; which can be managed using the `aws_key_pair` resource | `string` | `null` | no |
| <a name="input_launch_template"></a> [launch\_template](#input\_launch\_template) | Specifies a Launch Template to configure the instance. Parameters configured on this resource will override the corresponding parameters in the Launch Template | <pre>object({<br/>    id      = optional(string)<br/>    name    = optional(string)<br/>    version = optional(string)<br/>  })</pre> | `null` | no |
| <a name="input_maintenance_options"></a> [maintenance\_options](#input\_maintenance\_options) | The maintenance options for the instance | <pre>object({<br/>    auto_recovery = optional(string)<br/>  })</pre> | `null` | no |
| <a name="input_metadata_options"></a> [metadata\_options](#input\_metadata\_options) | Customize the metadata options of the instance | <pre>object({<br/>    http_endpoint               = optional(string, "enabled")<br/>    http_protocol_ipv6          = optional(string)<br/>    http_put_response_hop_limit = optional(number, 1)<br/>    http_tokens                 = optional(string, "required")<br/>    instance_metadata_tags      = optional(string)<br/>  })</pre> | <pre>{<br/>  "http_endpoint": "enabled",<br/>  "http_put_response_hop_limit": 1,<br/>  "http_tokens": "required"<br/>}</pre> | no |
| <a name="input_monitoring"></a> [monitoring](#input\_monitoring) | If true, the launched EC2 instance will have detailed monitoring enabled | `bool` | `null` | no |
| <a name="input_name"></a> [name](#input\_name) | Name to be used on EC2 instance created | `string` | `""` | no |
| <a name="input_network_interface"></a> [network\_interface](#input\_network\_interface) | Customize network interfaces to be attached at instance boot time | <pre>map(object({<br/>    delete_on_termination = optional(bool)<br/>    device_index          = optional(number) # Will fall back to use map key as device index<br/>    network_card_index    = optional(number)<br/>    network_interface_id  = string<br/>  }))</pre> | `null` | no |
| <a name="input_placement_group"></a> [placement\_group](#input\_placement\_group) | The Placement Group to start the instance in | `string` | `null` | no |
| <a name="input_placement_partition_number"></a> [placement\_partition\_number](#input\_placement\_partition\_number) | Number of the partition the instance is in. Valid only if the `aws_placement_group` resource's `strategy` argument is set to `partition` | `number` | `null` | no |
| <a name="input_private_dns_name_options"></a> [private\_dns\_name\_options](#input\_private\_dns\_name\_options) | Customize the private DNS name options of the instance | <pre>object({<br/>    enable_resource_name_dns_a_record    = optional(bool)<br/>    enable_resource_name_dns_aaaa_record = optional(bool)<br/>    hostname_type                        = optional(string)<br/>  })</pre> | `null` | no |
| <a name="input_private_ip"></a> [private\_ip](#input\_private\_ip) | Private IP address to associate with the instance in a VPC | `string` | `null` | no |
| <a name="input_putin_khuylo"></a> [putin\_khuylo](#input\_putin\_khuylo) | Do you agree that Putin doesn't respect Ukrainian sovereignty and territorial integrity? More info: https://en.wikipedia.org/wiki/Putin_khuylo! | `bool` | `true` | no |
| <a name="input_region"></a> [region](#input\_region) | Region where the resource(s) will be managed. Defaults to the Region set in the provider configuration | `string` | `null` | no |
| <a name="input_root_block_device"></a> [root\_block\_device](#input\_root\_block\_device) | Customize details about the root block device of the instance. See Block Devices below for details | <pre>object({<br/>    delete_on_termination = optional(bool)<br/>    encrypted             = optional(bool)<br/>    iops                  = optional(number)<br/>    kms_key_id            = optional(string)<br/>    tags                  = optional(map(string))<br/>    throughput            = optional(number)<br/>    size                  = optional(number)<br/>    type                  = optional(string)<br/>  })</pre> | `null` | no |
| <a name="input_secondary_private_ips"></a> [secondary\_private\_ips](#input\_secondary\_private\_ips) | A list of secondary private IPv4 addresses to assign to the instance's primary network interface (eth0) in a VPC. Can only be assigned to the primary network interface (eth0) attached at instance creation, not a pre-existing network interface i.e. referenced in a `network_interface block` | `list(string)` | `null` | no |
| <a name="input_security_group_description"></a> [security\_group\_description](#input\_security\_group\_description) | Description of the security group | `string` | `null` | no |
| <a name="input_security_group_egress_rules"></a> [security\_group\_egress\_rules](#input\_security\_group\_egress\_rules) | Egress rules to add to the security group | <pre>map(object({<br/>    cidr_ipv4                    = optional(string)<br/>    cidr_ipv6                    = optional(string)<br/>    description                  = optional(string)<br/>    from_port                    = optional(number)<br/>    ip_protocol                  = optional(string, "tcp")<br/>    prefix_list_id               = optional(string)<br/>    referenced_security_group_id = optional(string)<br/>    tags                         = optional(map(string), {})<br/>    to_port                      = optional(number)<br/>  }))</pre> | <pre>{<br/>  "ipv4_default": {<br/>    "cidr_ipv4": "0.0.0.0/0",<br/>    "description": "Allow all IPv4 traffic",<br/>    "ip_protocol": "-1"<br/>  },<br/>  "ipv6_default": {<br/>    "cidr_ipv6": "::/0",<br/>    "description": "Allow all IPv6 traffic",<br/>    "ip_protocol": "-1"<br/>  }<br/>}</pre> | no |
| <a name="input_security_group_ingress_rules"></a> [security\_group\_ingress\_rules](#input\_security\_group\_ingress\_rules) | Egress rules to add to the security group | <pre>map(object({<br/>    cidr_ipv4                    = optional(string)<br/>    cidr_ipv6                    = optional(string)<br/>    description                  = optional(string)<br/>    from_port                    = optional(number)<br/>    ip_protocol                  = optional(string, "tcp")<br/>    prefix_list_id               = optional(string)<br/>    referenced_security_group_id = optional(string)<br/>    tags                         = optional(map(string), {})<br/>    to_port                      = optional(number)<br/>  }))</pre> | `null` | no |
| <a name="input_security_group_name"></a> [security\_group\_name](#input\_security\_group\_name) | Name to use on security group created | `string` | `null` | no |
| <a name="input_security_group_tags"></a> [security\_group\_tags](#input\_security\_group\_tags) | A map of additional tags to add to the security group created | `map(string)` | `{}` | no |
| <a name="input_security_group_use_name_prefix"></a> [security\_group\_use\_name\_prefix](#input\_security\_group\_use\_name\_prefix) | Determines whether the security group name (`security_group_name` or `name`) is used as a prefix | `bool` | `true` | no |
| <a name="input_security_group_vpc_id"></a> [security\_group\_vpc\_id](#input\_security\_group\_vpc\_id) | VPC ID to create the security group in. If not set, the security group will be created in the default VPC | `string` | `null` | no |
| <a name="input_source_dest_check"></a> [source\_dest\_check](#input\_source\_dest\_check) | Controls if traffic is routed to the instance when the destination address does not match the instance. Used for NAT or VPNs | `bool` | `null` | no |
| <a name="input_spot_instance_interruption_behavior"></a> [spot\_instance\_interruption\_behavior](#input\_spot\_instance\_interruption\_behavior) | Indicates Spot instance behavior when it is interrupted. Valid values are `terminate`, `stop`, or `hibernate` | `string` | `null` | no |
| <a name="input_spot_launch_group"></a> [spot\_launch\_group](#input\_spot\_launch\_group) | A launch group is a group of spot instances that launch together and terminate together. If left empty instances are launched and terminated individually | `string` | `null` | no |
| <a name="input_spot_price"></a> [spot\_price](#input\_spot\_price) | The maximum price to request on the spot market. Defaults to on-demand price | `string` | `null` | no |
| <a name="input_spot_type"></a> [spot\_type](#input\_spot\_type) | If set to one-time, after the instance is terminated, the spot request will be closed. Default `persistent` | `string` | `null` | no |
| <a name="input_spot_valid_from"></a> [spot\_valid\_from](#input\_spot\_valid\_from) | The start date and time of the request, in UTC RFC3339 format(for example, YYYY-MM-DDTHH:MM:SSZ) | `string` | `null` | no |
| <a name="input_spot_valid_until"></a> [spot\_valid\_until](#input\_spot\_valid\_until) | The end date and time of the request, in UTC RFC3339 format(for example, YYYY-MM-DDTHH:MM:SSZ) | `string` | `null` | no |
| <a name="input_spot_wait_for_fulfillment"></a> [spot\_wait\_for\_fulfillment](#input\_spot\_wait\_for\_fulfillment) | If set, Terraform will wait for the Spot Request to be fulfilled, and will throw an error if the timeout of 10m is reached | `bool` | `null` | no |
| <a name="input_subnet_id"></a> [subnet\_id](#input\_subnet\_id) | The VPC Subnet ID to launch in | `string` | `null` | no |
| <a name="input_tags"></a> [tags](#input\_tags) | A mapping of tags to assign to the resource | `map(string)` | `{}` | no |
| <a name="input_tenancy"></a> [tenancy](#input\_tenancy) | The tenancy of the instance (if the instance is running in a VPC). Available values: default, dedicated, host | `string` | `null` | no |
| <a name="input_timeouts"></a> [timeouts](#input\_timeouts) | Define maximum timeout for creating, updating, and deleting EC2 instance resources | `map(string)` | `{}` | no |
| <a name="input_user_data"></a> [user\_data](#input\_user\_data) | The user data to provide when launching the instance. Do not pass gzip-compressed data via this argument; see user\_data\_base64 instead | `string` | `null` | no |
| <a name="input_user_data_base64"></a> [user\_data\_base64](#input\_user\_data\_base64) | Can be used instead of user\_data to pass base64-encoded binary data directly. Use this instead of user\_data whenever the value is not a valid UTF-8 string. For example, gzip-encoded user data must be base64-encoded and passed via this argument to avoid corruption | `string` | `null` | no |
| <a name="input_user_data_replace_on_change"></a> [user\_data\_replace\_on\_change](#input\_user\_data\_replace\_on\_change) | When used in combination with user\_data or user\_data\_base64 will trigger a destroy and recreate when set to true. Defaults to false if not set | `bool` | `null` | no |
| <a name="input_volume_tags"></a> [volume\_tags](#input\_volume\_tags) | A mapping of tags to assign to the devices created by the instance at launch time | `map(string)` | `{}` | no |
| <a name="input_vpc_security_group_ids"></a> [vpc\_security\_group\_ids](#input\_vpc\_security\_group\_ids) | A list of security group IDs to associate with | `list(string)` | `[]` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_ami"></a> [ami](#output\_ami) | AMI ID that was used to create the instance |
| <a name="output_arn"></a> [arn](#output\_arn) | The ARN of the instance |
| <a name="output_availability_zone"></a> [availability\_zone](#output\_availability\_zone) | The availability zone of the created instance |
| <a name="output_capacity_reservation_specification"></a> [capacity\_reservation\_specification](#output\_capacity\_reservation\_specification) | Capacity reservation specification of the instance |
| <a name="output_ebs_block_device"></a> [ebs\_block\_device](#output\_ebs\_block\_device) | EBS block device information |
| <a name="output_ebs_volumes"></a> [ebs\_volumes](#output\_ebs\_volumes) | Map of EBS volumes created and their attributes |
| <a name="output_ephemeral_block_device"></a> [ephemeral\_block\_device](#output\_ephemeral\_block\_device) | Ephemeral block device information |
| <a name="output_iam_instance_profile_arn"></a> [iam\_instance\_profile\_arn](#output\_iam\_instance\_profile\_arn) | ARN assigned by AWS to the instance profile |
| <a name="output_iam_instance_profile_id"></a> [iam\_instance\_profile\_id](#output\_iam\_instance\_profile\_id) | Instance profile's ID |
| <a name="output_iam_instance_profile_unique"></a> [iam\_instance\_profile\_unique](#output\_iam\_instance\_profile\_unique) | Stable and unique string identifying the IAM instance profile |
| <a name="output_iam_role_arn"></a> [iam\_role\_arn](#output\_iam\_role\_arn) | The Amazon Resource Name (ARN) specifying the IAM role |
| <a name="output_iam_role_name"></a> [iam\_role\_name](#output\_iam\_role\_name) | The name of the IAM role |
| <a name="output_iam_role_unique_id"></a> [iam\_role\_unique\_id](#output\_iam\_role\_unique\_id) | Stable and unique string identifying the IAM role |
| <a name="output_id"></a> [id](#output\_id) | The ID of the instance |
| <a name="output_instance_state"></a> [instance\_state](#output\_instance\_state) | The state of the instance |
| <a name="output_ipv6_addresses"></a> [ipv6\_addresses](#output\_ipv6\_addresses) | The IPv6 address assigned to the instance, if applicable |
| <a name="output_outpost_arn"></a> [outpost\_arn](#output\_outpost\_arn) | The ARN of the Outpost the instance is assigned to |
| <a name="output_password_data"></a> [password\_data](#output\_password\_data) | Base-64 encoded encrypted password data for the instance. Useful for getting the administrator password for instances running Microsoft Windows. This attribute is only exported if `get_password_data` is true |
| <a name="output_primary_network_interface_id"></a> [primary\_network\_interface\_id](#output\_primary\_network\_interface\_id) | The ID of the instance's primary network interface |
| <a name="output_private_dns"></a> [private\_dns](#output\_private\_dns) | The private DNS name assigned to the instance. Can only be used inside the Amazon EC2, and only available if you've enabled DNS hostnames for your VPC |
| <a name="output_private_ip"></a> [private\_ip](#output\_private\_ip) | The private IP address assigned to the instance |
| <a name="output_public_dns"></a> [public\_dns](#output\_public\_dns) | The public DNS name assigned to the instance. For EC2-VPC, this is only available if you've enabled DNS hostnames for your VPC |
| <a name="output_public_ip"></a> [public\_ip](#output\_public\_ip) | The public IP address assigned to the instance, if applicable. |
| <a name="output_root_block_device"></a> [root\_block\_device](#output\_root\_block\_device) | Root block device information |
| <a name="output_spot_bid_status"></a> [spot\_bid\_status](#output\_spot\_bid\_status) | The current bid status of the Spot Instance Request |
| <a name="output_spot_instance_id"></a> [spot\_instance\_id](#output\_spot\_instance\_id) | The Instance ID (if any) that is currently fulfilling the Spot Instance request |
| <a name="output_spot_request_state"></a> [spot\_request\_state](#output\_spot\_request\_state) | The current request state of the Spot Instance Request |
| <a name="output_tags_all"></a> [tags\_all](#output\_tags\_all) | A map of tags assigned to the resource, including those inherited from the provider default\_tags configuration block |
<!-- END_TF_DOCS -->

## Authors

Module is maintained by [Anton Babenko](https://github.com/antonbabenko) with help from [these awesome contributors](https://github.com/terraform-aws-modules/terraform-aws-ec2-instance/graphs/contributors).

## License

Apache 2 Licensed. See [LICENSE](https://github.com/terraform-aws-modules/terraform-aws-ec2-instance/tree/master/LICENSE) for full details.

## Additional information for users from Russia and Belarus

* Russia has [illegally annexed Crimea in 2014](https://en.wikipedia.org/wiki/Annexation_of_Crimea_by_the_Russian_Federation) and [brought the war in Donbas](https://en.wikipedia.org/wiki/War_in_Donbas) followed by [full-scale invasion of Ukraine in 2022](https://en.wikipedia.org/wiki/2022_Russian_invasion_of_Ukraine).
* Russia has brought sorrow and devastations to millions of Ukrainians, killed hundreds of innocent people, damaged thousands of buildings, and forced several million people to flee.
* [Putin khuylo!](https://en.wikipedia.org/wiki/Putin_khuylo!)
