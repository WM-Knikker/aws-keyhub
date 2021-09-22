# aws-keyhub
CLI login helper for the [AWS CLI](https://aws.amazon.com/cli/) when using SAML based federated login with [Topicus KeyHub](https://www.topicus-keyhub.com).

## Usage

### Installation

#### Release version
Download the latest release from https://github.com/topicuskeyhub/aws-keyhub/releases

##### Linux / macOS
1. Make the binary executable `chmod +x ./kubectl`
2. Move the binary to a file location on your system PATH. `sudo mv ./kubectl /usr/local/bin/kubectl`

##### Windows
1. Todo


### Configuration
To set up the aws-keyhub tool we need the KeyHub url, aws-keyhub ClientId and AWS SAML ClientId. Configuring these properties can be done by running with the `configure` command: `aws-keyhub configure`

### Authenticate
When the application is configured you can run the tool by executing `aws-keyhub login`.
It will open a webpage of KeyHub where you can authorize aws-keyhub. It than retrieves the roles. This roles are the AWS roles that you have access to in one or more AWS accounts.
If you provide the `--role-arn` parameter along with a valid role ARN for your account, that role will be automatically selected and you won't be prompted for a choice. For example `aws-keyhub login --role-arn arn:aws:iam::123456789012:role/MyCustomRole`

### Session duration
Due to [restrictions by Amazon Web Services](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithSAML.html) the maximum duration of the session is 12 hours. If authentication fails when using the AWS CLI please re-run the `aws-keyhub login` command to get a new session. The default session duration is 12 hours (43200 sec). If you need a shorter duration please reconfigure with `aws-keyhub configure`.

## Topicus KeyHub configuration
For optimal usage of this tool your KeyHub instance needs to be configured to send additional SAML payload. The payload helps a user to select the right role if they have access to multiple AWS accounts by displaying a description. Add the custom attribute ```https://github.com/topicuskeyhub/aws-keyhub/groups``` with the following code to build the descriptive array.

```javascript
// Function returns a list of descriptive objects based on a UUID match
var mapping = new Map();
mapping.set('aaabbbcc-2222-aaaa-3333-fffff0000000', {
    'account' : 'example-account-name',
    'role' : 'inzicht',
    'arn' : 'arn:aws:iam::123456789012:role/MyCustomRole,arn:aws:iam::123456789012:saml-provider/keyhub'
});

return groups.filter(function (group) {
    return mapping.has(group.uuid);
  }).map(function (group) {
    return "{\"description\": \"" + mapping.get(group.uuid).account + " - " + mapping.get(group.uuid).role + "\", \"arn\": \"" + mapping.get(group.uuid).arn + "\"}";
  });
```

## FAQ
#### How is my KeyHub password stored?
Your password is no longer stored in version 2 of this tool. It does store a temporary OIDC access token. 

#### Where is the configuration stored?
The configuration is stored in ```~/.aws-keyhub/config-v2.json```

#### Help! The login flow is broken, something seems to be corrupt.
Please verify that you can successfully login to the AWS console in your browser before using this tool.

### Migrating from v1 to v2
There is no migration path as in you have to install and configure aws-keyhub again. Any previous configuration is not persisted. We recommend you remove the old v1 configuration files by deleting the following files and directories:
```
~/.aws-keyhub/config.json
~/.aws-keyhub/puppeteer_profile
```