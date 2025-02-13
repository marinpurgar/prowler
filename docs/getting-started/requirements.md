# Requirements

Prowler has been written in Python using the [AWS SDK (Boto3)](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html#) and [Azure SDK](https://learn.microsoft.com/en-us/python/api/overview/azure/?view=azure-python).
## AWS

Since Prowler uses AWS Credentials under the hood, you can follow any authentication method as described [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-precedence).

### AWS Authentication

Make sure you have properly configured your AWS-CLI with a valid Access Key and Region or declare AWS variables properly (or instance profile/role):

```console
aws configure
```

or

```console
export AWS_ACCESS_KEY_ID="ASXXXXXXX"
export AWS_SECRET_ACCESS_KEY="XXXXXXXXX"
export AWS_SESSION_TOKEN="XXXXXXXXX"
```

Those credentials must be associated to a user or role with proper permissions to do all checks. To make sure, add the following AWS managed policies to the user or role being used:

  - arn:aws:iam::aws:policy/SecurityAudit
  - arn:aws:iam::aws:policy/job-function/ViewOnlyAccess

  > Moreover, some read-only additional permissions are needed for several checks, make sure you attach also the custom policy [prowler-additions-policy.json](https://github.com/prowler-cloud/prowler/blob/master/iam/prowler-additions-policy.json) to the role you are using.

  > If you want Prowler to send findings to [AWS Security Hub](https://aws.amazon.com/security-hub), make sure you also attach the custom policy [prowler-security-hub.json](https://github.com/prowler-cloud/prowler/blob/master/iam/prowler-security-hub.json).

## Azure

Prowler for azure supports the following authentication types:

- Service principal authentication by environment variables (Enterprise Application)
- Current az cli credentials stored
- Interactive browser authentication
- Managed identity authentication

### Service Principal authentication

To allow Prowler assume the service principal identity to start the scan it is needed to configure the following environment variables:

```console
export AZURE_CLIENT_ID="XXXXXXXXX"
export AZURE_TENANT_ID="XXXXXXXXX"
export AZURE_CLIENT_SECRET="XXXXXXX"
```

If you try to execute Prowler with the `--sp-env-auth` flag and those variables are empty or not exported, the execution is going to fail.
### AZ CLI / Browser / Managed Identity authentication

The other three cases does not need additional configuration, `--az-cli-auth` and `--managed-identity-auth` are automated options, `--browser-auth` needs the user to authenticate using the default browser to start the scan.

### Permissions

To use each one you need to pass the proper flag to the execution. Prowler fro Azure handles two types of permission scopes, which are:

- **Azure Active Directory permissions**: Used to retrieve metadata from the identity assumed by Prowler and future AAD checks (not mandatory to have access to execute the tool)
- **Subscription scope permissions**: Required to launch the checks against your resources, mandatory to launch the tool.


#### Azure Active Directory scope

Azure Active Directory (AAD) permissions required by the tool are the following:

- `Directory.Read.All`
- `Policy.Read.All`

The best way to assign it is through the azure web console:

![AAD Permissions](../img/AAD-permissions.png)

#### Subscriptions scope

Regarding the subscription scope, Prowler by default scans all the subscriptions that is able to list, so it is required to add the following RBAC builtin roles per subscription  to the entity that is going to be assumed by the tool:

- `Security Reader`
- `Reader`
