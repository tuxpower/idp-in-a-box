# 060-simplesamlphp - ECS service for simpleSAMLphp IdP
This module is used to create an ECS service running simpleSAMLphp.

## What this does

 - Create ALB target group for SSP with hostname based routing
 - Create task definition and ECS service for simpleSAMLphp
 - Create Cloudflare DNS records for SSP

## Required Inputs

 - `app_name` - Application name
 - `app_env` - Application environment
 - `logentries_set_id` - Logentries logset ID for creating new log in
 - `vpc_id` - ID for VPC
 - `alb_https_listener_arn` - ARN for ALB HTTPS listener
 - `subdomain` - Subdomain for SSP IdP
 - `cloudflare_domain` - Top level domain name for use with Cloudflare
 - `docker_image` - URL to Docker image
 - `password_change_url` - URL to change password page
 - `password_forgot_url` - URL to forgot password page
 - `hub_mode` - Whether or not this IdP is in hub mode, default: false
 - `id_broker_access_token` - Access token for calling id-broker
 - `id_broker_assert_valid_ip` - Whether or not to assert valid ip for calling id-broker
 - `id_broker_trusted_ip_ranges` - List of trusted ip blocks for ID Broker
 - `id_broker_base_uri` - Base URL to id-broker API
 - `mfa_learn_more_url` - URL to learn more about MFA
 - `mfa_setup_url` - URL to setup MFA
 - `memcache_host1` - First memcache host
 - `memcache_host2` - Second memcache host
 - `db_name` - Name of MySQL database for ssp
 - `mysql_host` - Address for RDS instance
 - `mysql_user` - MySQL username for id-broker
 - `mysql_pass` - MySQL password for id-broker
 - `recaptcha_key` - Recaptcha site key
 - `recaptcha_secret` - Recaptcha secret
 - `remember_me_secret` - Secret key used in MFA remember me cookie generation
 - `ecs_cluster_id` - ID for ECS Cluster
 - `ecsServiceRole_arn` - ARN for ECS Service Role
 - `alb_dns_name` - DNS name for application load balancer
 - `idp_name` - Short name of IdP for use in logs and email alerts
 - `theme_color_scheme` - The color scheme to use for SSP. Default: `'indigo-purple'`
 - `trusted_ip_addresses` - A list of ip addresses or ranges that should not be rate limited
 - `analytics_id` - The ID used by an analytics provider such as Google Analytics, e.g., "UA-XXXX-YY"
 - `show_saml_errors` - Whether or not to show saml errors. Default: `false`
 - `delete_remember_me_on_logout` - Whether or not to delete remember me cookie on logout. Default: `true`

## Outputs

 - `hostname` - URL to SSP
 - `db_ssp_user` - MySQL Username
 - `admin_pass` - SSP Admin password

## Usage Example

```hcl
module "cf_ips" {
  source = "github.com/silinternational/terraform-modules//cloudflare/ips?ref=2.0.1"
}

module "ssp" {
  source                       = "github.com/silinternational/idp-in-a-box//terraform/060-simplesamlphp"
  memory                       = "${var.memory}"
  cpu                          = "${var.cpu}"
  desired_count                = "${var.desired_count}"
  app_name                     = "${var.app_name}"
  app_env                      = "${var.app_env}"
  logentries_set_id            = "${data.terraform_remote_state.cluster.logentries_set_id}"
  vpc_id                       = "${data.terraform_remote_state.cluster.vpc_id}"
  alb_https_listener_arn       = "${data.terraform_remote_state.cluster.alb_https_listener_arn}"
  subdomain                    = "${var.ssp_subdomain}"
  cloudflare_domain            = "${var.cloudflare_domain}"
  docker_image                 = "${data.terraform_remote_state.ecr.ecr_repo_simplesamlphp}"
  password_change_url          = "https://${data.terraform_remote_state.pwmanager.ui_hostname}/#/change"
  password_forgot_url          = "https://${data.terraform_remote_state.pwmanager.ui_hostname}/#/forgot"
  hub_mode                     = "${var.hub_mode}"
  id_broker_access_token       = "${data.terraform_remote_state.broker.access_token_ssp}"
  id_broker_assert_valid_ip    = "${var.id_broker_assert_valid_ip}"
  id_broker_base_uri           = "https://${data.terraform_remote_state.broker.hostname}"
  id_broker_trusted_ip_ranges  = ["${data.terraform_remote_state.cluster.private_subnet_cidr_blocks}"]
  mfa_learn_more_url           = "${var.mfa_learn_more_url}"
  mfa_setup_url                = "${var.mfa_setup_url}"
  memcache_host1               = "${data.terraform_remote_state.elasticache.cache_nodes.0.address}"
  memcache_host2               = "${data.terraform_remote_state.elasticache.cache_nodes.1.address}"
  db_name                      = "${var.db_ssp_name}"
  mysql_host                   = "${data.terraform_remote_state.database.rds_address}"
  mysql_user                   = "${var.db_ssp_user}"
  mysql_pass                   = "${data.terraform_remote_state.database.db_ssp_pass}"
  recaptcha_key                = "${var.recaptcha_key}"
  recaptcha_secret             = "${var.recaptcha_secret}"
  remember_me_secret           = "${var.remember_me_secret}"
  ecs_cluster_id               = "${data.terraform_remote_state.core.ecs_cluster_id}"
  ecsServiceRole_arn           = "${data.terraform_remote_state.core.ecsServiceRole_arn}"
  alb_dns_name                 = "${data.terraform_remote_state.cluster.alb_dns_name}"
  idp_name                     = "${var.idp_name}"
  theme_color_scheme           = "${var.theme_color_scheme}"
  trusted_ip_addresses         = ["${concat(module.cf_ips.ipv4_cidrs, var.trusted_ip_addresses, data.terraform_remote_state.cluster.public_subnet_cidr_blocks)}"]
  analytics_id                 = "${var.analytics_id}"
  show_saml_errors             = "${var.show_saml_errors}"
  delete_remember_me_on_logout = "${var.delete_remember_me_on_logout}"
}
```
