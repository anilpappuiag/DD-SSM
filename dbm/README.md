# DD-SSM
# **Datadog Agent Setup for Oracle on EC2 with Database Monitoring**

This document outlines the steps to set up the Datadog Agent for monitoring a self-hosted Oracle database on EC2. It includes creating the `dd_session` view required for Datadog Database Monitoring (DBM).

---

## **Overview**
This SSM document automates the installation and configuration of the Datadog Agent on Oracle database hosts and performs the following tasks:
1. Installs and configures the Datadog Agent.
2. Sets up the Oracle integration with necessary credentials.
3. Creates the `dd_session` view in the Oracle database.
4. Validates the agent installation and DBM setup.

---

## **Prerequisites**
1. **Datadog API Key**:
   - Store your Datadog API key in AWS Systems Manager Parameter Store under `/datadog/apikey`.

2. **Database Credentials**:
   - Create a Datadog read-only user in Oracle:
     ```sql
     CREATE USER maestro_datadog_ro IDENTIFIED BY [password];
     GRANT CREATE SESSION TO maestro_datadog_ro;
     GRANT SELECT ON v_$session TO maestro_datadog_ro;
     ```
   - Store the username and password in AWS Systems Manager Parameter Store:
     - `/datadog/dbm/[dbname]/username`
     - `/datadog/dbm/[dbname]/password`

3. **Database Admin Credentials**:
   - Store the Oracle admin username and password securely in AWS Systems Manager Parameter Store:
     - `/datadog/dbm/admin/password`

4. **Firewall Rules**:
   - Allow outgoing traffic from the database host to the following Datadog endpoints:
     - `trace.agent.datadoghq.eu`
     - `api.datadoghq.eu`

5. **Access Permissions**:
   - Ensure the IAM role associated with the EC2 instance has permissions to access AWS Systems Manager Parameter Store.

---

## **Execution Steps**
1. **Upload the SSM Document**:
   - Save the SSM document JSON to a file, e.g., `datadog_oracle_setup.json`.
   - In the AWS Management Console, go to **Systems Manager** > **Documents** > **Create Document**.
   - Select **Command document**, upload the JSON file, and save it.

2. **Run the Document**:
   - Navigate to **Run Command** in Systems Manager.
   - Select the uploaded document.
   - Provide the required parameters:
     - `ApiKey`: Path to the Datadog API key in Parameter Store.
     - `DBUsername` and `DBPassword`: Paths to the database monitoring credentials.
     - `DBAdminPassword`: Path to the Oracle admin password in Parameter Store.
     - `DBServiceName`: Oracle service name (e.g., `ORCL`).
     - `Region`: AWS region where the Parameter Store values are located.

3. **Monitor Progress**:
   - Check the **Command Output** in Systems Manager to ensure all steps are executed successfully.

4. **Verify Installation**:
   - Log in to the EC2 instance.
   - Run the following command to validate the Datadog Agent setup:
     ```bash
     sudo datadog-agent status
     ```
   - Confirm Oracle database metrics are visible in the Datadog EU console.

---

## **What the SSM Document Does**
1. **Installs Datadog Agent**:
   - Downloads and installs the agent based on the instance's OS.
   - Configures the agent to monitor Oracle databases.

2. **Configures Oracle Integration**:
   - Fetches credentials from Parameter Store.
   - Updates the `datadog.yaml` configuration file for DBM.

3. **Creates `dd_session` View**:
   - Logs into the Oracle database as `sysdba`.
   - Creates the `dd_session` view and grants required permissions to the Datadog user.

4. **Validates Setup**:
   - Runs `datadog-agent status` to ensure the agent is running and collecting metrics.

---

## **Key Parameters**
| **Parameter**       | **Description**                                                         |
|----------------------|-------------------------------------------------------------------------|
| `ApiKey`            | Datadog API key stored in Parameter Store.                              |
| `DBUsername`        | Read-only Datadog database username stored in Parameter Store.          |
| `DBPassword`        | Password for the Datadog database user stored in Parameter Store.       |
| `DBAdminPassword`   | Admin password for Oracle stored in Parameter Store.                    |
| `DBServiceName`     | Oracle database service name (e.g., `ORCL`).                            |
| `AgentVersion`      | Optional: Version of the Datadog Agent to install (`latest` by default).|
| `Region`            | AWS region where the Parameter Store values are stored.                |

---

## **Troubleshooting**
1. **Agent Installation Issues**:
   - Ensure the EC2 instance has internet access to download the Datadog Agent.
   - Check for OS compatibility with the Datadog Agent.

2. **Oracle Database Connectivity**:
   - Verify the Oracle service name and credentials.
   - Ensure the EC2 instance can connect to the Oracle database.

3. **Datadog Console Visibility**:
   - Check the Datadog EU console for Oracle metrics.
   - Verify the database host can reach the Datadog EU endpoints.

---

## **Additional Notes**
- **Custom View**:
  - The `dd_session` view is customized to provide Datadog with the required data for monitoring Oracle performance.
- **Security**:
  - Use AWS Systems Manager Parameter Store to securely store sensitive information.
  - Restrict IAM permissions to ensure only authorized roles can access the parameters.

---
