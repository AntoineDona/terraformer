# Use Terraformer with [Datadog](https://www.datadoghq.com/)

This provider uses the [terraform-provider-datadog](https://registry.terraform.io/providers/DataDog/datadog/latest).

##  Usage
### 1. Installation
First you will need to install Terraformer with the Datadog provider. See the [README](https://github.com/GoogleCloudPlatform/terraformer#installation).

### 2. Set up a template Terraform workspace
Before you can use Terraformer, you need to create a template workspace so that Terraformer
can access the [DataDog/datadog](https://registry.terraform.io/providers/DataDog/datadog/latest) provider.

To do this, create a new directory with a basic `provider.tf` file:
```hcl
terraform {
  required_providers {
    datadog = {
      source  = "DataDog/datadog"
      version = "~> 3.67.0"         # Find the latest version here: https://github.com/DataDog/terraform-provider-datadog/releases/latest
    }
  }
}

provider "datadog" {
  # Configuration options
}
```

then run:
```bash
$ terraform init
````

You should see the output: `Terraform has been successfully initialized!`

### 3. Run Terraformer

```bash
export DATADOG_API_KEY=Datadog API key. More information on this at https://docs.datadoghq.com/account_management/api-app-keys/ 
export DATADOG_HOST=Datadog API host i.e. https://api.datadoghq.eu which can be found at https://docs.datadoghq.com/getting_started/site/#access-the-datadog-site
export DATADOG_APP_KEY=Datadog APP key. More information on this at https://docs.datadoghq.com/account_management/api-app-keys/ 

./terraformer import datadog --resources=* 
```

You can also specify only certain kinds of resources to import as well, i.e. `--resources=dashboard`.

### 4. Inspect the imported Terraform files

You should now see a `generated/` subdirectory with generated files.

You can now initialize and use your new generated resources:
```bash
$ terraform init
$ terraform plan # No changes. Your infrastructure matches the configuration.
```

### Filtering Resources

You can use the `filter` argument to restrict the import of Terraform resources.

Filtering based on Tags follows the convention `--filter="Name=tags;Value='your tag'"`.

```bash
# Import monitors based on multiple tags
./terraformer import datadog --resources=monitor --filter="Name=tags;Value='foo:bar'" --filter="Name=tags;Value='env:production'"

# Import monitor where tag doesn't include colon
./terraformer import datadog --resources=monitor --filter="Name=tags;Value=anExampleTag"
```

Filtering based on resource ID:

```bash
# Import dashboard based on the dashboard ID
./terraformer import datadog --resources=dashboard --filter=dashboard=some-id

# Import based on multiple resource IDs
 ./terraformer import datadog --resources=monitor --filter=monitor=id1:id2:id4
```

Tag filters are order specific. For example, if your monitor has tags (in the order) `atag: atagvalue`, `foo:bar` but you filter for `--filter="Name=tags;Value='foo:bar'" --filter="Name=tags;Value='atag: atagvalue'"`, the monitor would not be imported.

### Preserving Resource Order with `--no-sort`

When importing certain Datadog resources like dashboards or synthetic tests, you may need to use the `--no-sort` option to preserve the correct order of elements (e.g., dashboard widgets, synthetic test steps).

Without `--no-sort`, the imported resources may have their internal ordering altered, which can cause issues when applying the Terraform configuration.

**Recommended usage:**
```bash
# For dashboards
./terraformer import datadog --resources=dashboard --no-sort

# For synthetic tests  
./terraformer import datadog --resources=synthetics_test --no-sort

# For all resources (recommended)
./terraformer import datadog --resources=* --no-sort
```

This ensures that the imported Terraform configuration maintains the same order as your existing Datadog resources.

## Supported Datadog resources

*   `dashboard`
    * `datadog_dashboard`
*   `dashboard_json`
    * `datadog_dashboard_json`
*   `dashboard_list`
    * `datadog_dashboard_list`
*   `downtime`
    * `datadog_downtime`
*   `integration_aws`
    * `datadog_integration_aws`
*   `integration_aws_lambda_arn`
    * `datadog_integration_aws_lambda_arn`
*   `integration_aws_log_collection`
    * `datadog_integration_aws_log_collection`
*   `integration_azure`
    * `datadog_integration_azure`
        * **_NOTE:_** Sensitive field `client_secret` is not generated and needs to be manually set
*   `integration_gcp`
    * `datadog_integration_gcp`
        * **_NOTE:_** Sensitive fields `private_key, private_key_id, client_id` is not generated and needs to be manually set
*   `integration_pagerduty`
    * `datadog_integration_pagerduty`
*   `integration_pagerduty_service_object`
    * `datadog_integration_pagerduty_service_object`
*   `integration_slack_channel`
    * `datadog_integration_slack_channel`
        * **_NOTE:_** Importing resource requires resource ID or `account_name` to be passed via [Filter][1] option
*   `logs_archive`
    * `datadog_logs_archive`
*   `logs_archive_order`
    * `datadog_logs_archive_order`
*   `logs_custom_pipeline`
    * `datadog_logs_custom_pipeline`
*   `logs_index`
    * `datadog_logs_index`
*   `logs_index_order`
    * `datadog_logs_index_order`
*   `logs_integration_pipeline`
    * `datadog_logs_integration_pipeline`
*   `logs_pipeline_order`
    * `datadog_logs_pipeline_order`
*   `metric_metadata`
    * `datadog_metric_metadata`
        * **_NOTE:_** Importing resource requires resource ID's to be passed via [Filter][1] option
*   `monitor`
    * `datadog_monitor`
*   `role`
    * `datadog_role`
*   `security_monitoring_default_rule`
    * `datadog_security_monitoring_default_rule`
*   `security_monitoring_rule`
    * `datadog_security_monitoring_rule`
*   `service_level_objective`
    * `datadog_service_level_objective`
*   `synthetics_global_variable`
    * `datadog_synthetics_global_variable`
        * **_NOTE:_** Importing resource requires resource ID's to be passed via [Filter][1] option
*   `synthetics_private_location`
    * `datadog_synthetics_private_location`
*   `synthetics_test`
    * `datadog_synthetics_test`
*   `user`
    * `datadog_user`

[1]: https://github.com/GoogleCloudPlatform/terraformer/blob/master/README.md#filtering
