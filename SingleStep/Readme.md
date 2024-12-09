# Install Datadog Agent via AWS Systems Manager

This AWS Systems Manager document installs the Datadog Agent on Linux instances. It supports configurable parameters for the Datadog API key, site, APM instrumentation, environment, and optional tags.

## Prerequisites

1. **AWS Systems Manager Agent (SSM Agent)**:
   - Ensure the SSM Agent is installed and running on your instances.
2. **IAM Role**:
   - Attach an IAM role with `ssm:SendCommand` and other necessary permissions to your instance profile.
3. **Datadog Account**:
   - Obtain a valid Datadog API Key from your Datadog account.
4. **Outbound Network Access**:
   - Ensure instances can access `https://install.datadoghq.com` and Datadog endpoints.

---

## Parameters

| Parameter Name             | Type    | Default         | Description                                                                                  |
|----------------------------|---------|-----------------|----------------------------------------------------------------------------------------------|
| `DDAPIKEY`                 | String  | Required        | The Datadog API Key for authentication.                                                     |
| `DDSITE`                   | String  | `datadoghq.eu`  | The Datadog site to use (e.g., `datadoghq.com` or `datadoghq.eu`).                          |
| `DDAPMINSTRUMENTATION`     | String  | `host`          | The APM instrumentation type (e.g., `host` or `container`).                                 |
| `DDAPMLIBRARIES`           | String  | `java:1,python:2,js:5,dotnet:3` | APM libraries to enable with version mappings.                                              |
| `DDENV`                    | String  | Required        | The environment to tag the Datadog Agent (e.g., `production`, `staging`).                   |
| `DDTAGS`                   | String  | `""` (optional) | Optional tags to categorize the host in Datadog (comma-separated, e.g., `team:web,env:prod`).|

---

## Usage

1. **Create the Systems Manager Document**:
   - Use the AWS Management Console or AWS CLI to create the document with the content provided.

   Example CLI Command:
   ```bash
   aws ssm create-document \
     --name "InstallDatadogAgent" \
     --document-type "Command" \
     --content file://InstallDatadogAgent.json
   ```

2. **Run the Document**:
   - Use the `Run Command` feature in AWS Systems Manager to execute the document.

   Example CLI Command:
   ```bash
   aws ssm send-command \
     --document-name "InstallDatadogAgent" \
     --targets "Key=instanceIds,Values=<INSTANCE_ID>" \
     --parameters 'DDAPIKEY=<YOUR_API_KEY>,DDSITE=datadoghq.com,DDENV=production,DDTAGS=team:web,env:prod'
   ```

3. **Monitor Execution**:
   - View the execution output in the Systems Manager Console under the "Command history" section.

---

## Key Features

- **Flexible Configuration**:
  - Customize Datadog site, environment, tags, and APM instrumentation.
- **Error Handling**:
  - Validates required inputs and reports installation success or failure.
- **Automation**:
  - Streamlines Datadog Agent installation without manual intervention.

---

## Known Limitations

- This document supports Linux-based instances only.
- Ensure that all required network endpoints are accessible from the target instance.

---

## Troubleshooting

- **Error: `Datadog Agent installation failed.`**:
  - Verify the provided API Key and network connectivity to Datadog endpoints.
- **SSM Command Not Executing**:
  - Ensure the instance has the appropriate IAM role with necessary SSM permissions.

---

## References

- [Datadog Agent Installation Guide](https://docs.datadoghq.com/agent/)
- [AWS Systems Manager Documentation](https://docs.aws.amazon.com/systems-manager/)

--- 
