# Migrate Azure Resources between tenants

This article provides guidance on how to transfer Azure resources to a different tenant. The process can be executed using various tools, including the Azure portal, Azure PowerShell, Azure CLI, or the REST API.

It's important to note that this operation is complex and should only be undertaken when necessary. Our primary recommendation is to rebuild your resources in the target tenant. However, there are scenarios where a rebuild might not be feasible or desirable, leaving the transfer of Azure resources as the only viable option. Additionally, not all Azure resources are eligible for transfer across tenants. In such cases, these resources need to be recreated.

Azure resources can't be migrated to a different tenant on their own. While it's possible to migrate resources between subscriptions and resource groups, these migrations are restricted to movements within the same tenant. To transfer Azure resources, the subscription in which they reside must be moved to the new target tenant. This migration can significantly affect and potentially interrupt any workloads running in the subscription or dependent resources. The purpose of this article is to offer guidance on identifying resources that can support a tenant-to-tenant subscription move and to outline the necessary migration steps.

## Checklist

Before initiating the subscription move, it's crucial to establish a comprehensive migration project plan. Utilize project management tools, such as Microsoft Project, to meticulously track all aspects of the migration. Ensure that every element, including dependencies, conditions, recovery steps, risks, and validations, is thoroughly documented. Incorporate the following checkpoints into your planning and expand them as necessary:

- **Action Item:** Define specific tasks to be completed.
- **Remarks and Considerations:** Note any special notes or considerations for each action item.
- **Tenant Environment:** Detail the current and target tenant environments.
- **Owner:** Assign an individual responsible for each action item.
- **Status:** Update the current status of each task (for example, Not Started, In Progress, Completed).
- **Notes:** Include any additional information or observations.
- **Links:** Provide relevant links to resources or documentation.
- **Dependencies:** Identify any dependencies between tasks.
- **Pre-conditions:** List conditions that must be met before starting a task.
- **Post-conditions:** Specify conditions that indicate the successful completion of a task.
- **Recovery Steps:** Outline steps to revert changes if necessary.
- **Risks:** Document potential risks and mitigation strategies.
- **Validations:** Define how each task is validated upon completion.
- **Stakeholders Notified:** Ensure all relevant stakeholders are informed.
- **Attachments:** Include any pertinent documents or files.
- **Start Date:** Specify the planned start date for each task.
- **Start Time:** Indicate the start time for tasks, if applicable.
- **Duration:** Estimate how long each task takes.
- **End Date:** Note the completion date for tasks, if applicable.
- **End Time:** Note the completion time for tasks, if applicable.

This structured approach helps ensure a well-organized and effective migration process.

### Action Plan

| Action | Description |
| --- | --- |
| Assess source and target Azure Policies | <ul><li>Baseline knowledge - Understand Azure Policy configurations that currently exist, or a baseline document, to help identify deviations or anomalies during the assessment.</li><li>Clear scope of assessment, focus on entire subscription and inheritance policies if applicable</li></ul> |
| Assess source Azure RBAC Role Assignments | <ul><li>Existing Role Assignment Documentation: An updated list or documentation of all existing Azure RBAC role assignments in the source tenant.</li><li>Knowledge of Target Directory: A clear understanding or initial assessment of the target directory structure and objects to guide the mapping process.</li></ul> |
| Identify Custom Roles and Managed Identities in the source tenant | <ul><li>Access Rights: Necessary permissions to view and export custom roles in the source tenant.</li><li>Data Collection Tools: Tools or scripts, if any, for identifying and documenting custom roles should be pretested.</li></ul> |
| Assess Azure Key Vaults with Certs, Secrets, and Keys | <ul><li>Knowledge of Target Tenant: An understanding of the target tenant ID that needs to be associated with the Key Vaults post-migration.</li></ul> |
| Assess any Active Directory authentication integrated resources (for example, databases) | <ul><li>Existing Documentation: A current list or documentation of all databases that utilize Active Directory authentication in the source tenant.</li><li>Knowledge of Target Directory: An understanding of the target AD to identify how authentication is handled post-migration.</li></ul> |
| Azure File Services & Data Lake Storage | <ul><li>Existing Documentation: A current list or documentation of all resources in Azure File Services and Data Lake Storage that utilize NTFS/SAS ACLs in the source tenant.</li></ul> |
| Assess the AKS environments | <ul><li>Access Rights: Necessary permissions to access, view, and manage the AKS clusters in the source tenant.</li><li>Data Collection Tools: Any tools or scripts used for the assessment should be pretested to ensure they can pull all AKS configurations, workloads, associated resources, and settings.</li><li>Existing Documentation: A current list or documentation of all AKS clusters, their configurations, node pools, network settings, storage integrations, and other associated resources.</li><li>Target Directory Knowledge: An understanding of the target directory and how the recreated AKS environment should be configured there.</li></ul> |
| Assess any Entra application registrations | <ul><li>Document all application registrations in the source Microsoft Entra ID environment and any other related configurations.</li></ul> |
| (Managed) Disk encryptions | <ul><li>Encryption Keys Backup: Back up or secure storage of customer-managed encryption keys, ensuring that they're retrievable for usage in the target directory.</li></ul> |
| Assess DNS and Prep for DNS delegation | <ul><li>Existing Documentation: A current list or documentation of all DNS zones, their records, associated settings, and any potential subdelegations in the source tenant.</li><li>Backup: A backup of all DNS configurations, including zone files, to ensure recovery if needed.</li></ul> |
| Validate Blockers - Azure Marketplace | <ul><li>Azure Marketplace Precheck Tool: Access to any Azure tools or scripts that can run prechecks to validate the transfer eligibility of Azure Marketplace deployments.</li></ul> |
| Validate Blockers - Azure Reservations | <ul><li>Existing Documentation: A list or documentation detailing all current Reserved Instances, including the applied discounts, duration left, and associated resources.</li><li>Cost Analysis Tools: Tools or scripts to perform a cost-benefit analysis considering the migration and potential loss of discounts.</li></ul> |
| Assess Virtual Network and Security Groups | <ul><li>Existing Documentation: Current documentation of all Virtual Networks, their subnet allocations, associated NSGs, rules, and any hybrid connection setups in the source tenant.</li><li>Target Environment Details: Knowledge about the target environment's network configurations to ensure no conflicts.</li></ul> |
| Assess Azure Resource Locks | <ul><li>Resource Listing: A list or inventory of all resources in the source tenant that could potentially have resource locks.</li></ul> |
| Azure Monitor and Alerts | <ul><li>Resource Inventory: A list or inventory of all resources in the source tenant that have monitoring configurations and alerts.</li><li>Data Collection Tools: Tools or scripts validated to extract details about Azure Monitor configurations, metrics, log analytics workspace configurations, and associated alerts.</li></ul> |
| Billing and Cost Management | <ul><li>Resource List: An exhaustive list of resources/services used in the source tenant and their associated costs.</li><li>Cost Management Tools: Access to tools and dashboards (like Microsoft Cost Management and Billing) that provide insight into the current cost configuration and breakdown.</li></ul> |
| Evaluate External Dependencies | <ul><li>Dependency Inventory: A comprehensive list of all resources and services in the source tenant that communicate or integrate with external systems, be they on-premises, in other clouds, or other non-Microsoft services.</li><li>Access Rights: Necessary permissions to view configurations and logs related to external communications and integrations within the source tenant.</li><li>Data Collection Tools: Tools, scripts, or services (like Azure Network Watcher) that can help detail and evaluate outgoing and incoming connections.</li></ul> |
| Azure DevOps and CI/CD Pipelines | <ul><li>Access to Azure DevOps: Necessary permissions to view all CI/CD configurations, repositories, pipelines, service connections, variable groups, and associated resources within the source tenant's Azure DevOps Projects.</li><li>Existing Pipelines: A documented inventory of all CI/CD pipelines, including build, release, YAML pipelines, and their associated triggers, agents, and stages.</li><li>Service Connections: A list of all service connections that pipelines utilize to connect to Azure services or other external resources.</li></ul> |
| Review Shared Dashboard and Reports | <ul><li>Access: Necessary permissions to view all shared dashboards, Power BI reports, and any other associated reporting resources within the source tenant.</li><li>Inventory: A comprehensive list of all shared dashboards, reports, data sources, and associated permissions.</li><li>Data Source Configurations: Knowledge of where and how these dashboards and reports pull data, especially if it's from resources within the same Azure tenant.</li></ul> |
| List other known resources | <ul><li>Azure CLI with az graph: Ensure you have the Azure Command-Line Interface (CLI) installed, with the 'az graph' extension added. The extension enables querying Azure resources using the Azure Resource Graph.</li><li>Permissions: The user must have permissions on the resources they intend to query. Typically, a Reader role at a minimum is required.</li><li>Query Preparedness: Familiarity with the Kusto Query Language (KQL), as KQL is the language used for Azure Graph Queries.</li></ul> |
| Match and recreate Azure Polices | <ul><li>All Azure policies in the source environment are fully documented and available for review.</li><li>Access to the target environment is established, and any conflicting or redundant policies are identified.</li><li>Necessary credentials or roles are available to recreate the policies in the target environment.</li></ul> |
| Recreate and Assign any Azure RBAC Role Assignments | <ul><li>All Azure RBAC role assignments in the source environment are fully documented and available for review.</li><li>Access to the target environment is established.</li><li>The necessary credentials or roles are available to recreate and assign the roles in the target environment.</li><li>Users, groups, and service principals from the source environment have corresponding objects in the target directory or a mapped counterpart.</li></ul> |
| Recreate and Assign Custom roles | <ul><li>Azure CLI with az graph: Ensure you have the Azure Command-Line Interface (CLI) installed, with the 'az graph' extension added. This extension enables querying Azure resources using the Azure Resource Graph.</li><li>Permissions: The user must have permissions on the resources they intend to query. Typically, a Reader role at a minimum is required.</li><li>Query Preparedness: Familiarity with the Kusto Query Language (KQL), as KQL is the language used with 'az graph query'.</li></ul> |
| Recreate and assign System Assigned Managed Identities | <ul><li>All system-assigned managed identities in the source environment are documented, including the roles they hold.</li><li>Access to the target environment is established.</li><li>Necessary credentials or roles to manage system-assigned managed identities are available for use in the target environment.</li></ul> |
| Recreate and assign User-assigned Managed Identities | <ul><li>All user-assigned managed identities in the source environment are documented, including the roles they hold and the resources they're associated with.</li><li>Access to the target environment is established.</li><li>Necessary credentials or roles to manage user-assigned managed identities are available for use in the target environment.</li></ul> |
| Recreate Azure Key Vaults with Certs, Secrets, and Keys | <ul><li>Source Azure Key Vaults are accessible and are fully documented.</li><li>Access to the target environment is secured.</li><li>The necessary permissions to manage Key Vaults in the target environment are in place.</li><li>There are no conflicting Key Vault names or configurations in the target environment.</li></ul> |
| Recreate the AKS environments | <ul><li>The AKS clusters in the source environment are documented and backed up.</li><li>The target environment is ready for AKS deployment, with the necessary permissions in place.</li><li>Supporting Azure services (like networking or storage) that AKS depends on are also replicated or set up in the target environment.</li></ul> |
| Assess any Entra application registrations | <ul><li>The Microsoft Entra ID in the source environment is accessible and has the application registrations that need to be assessed.</li><li>Clear documentation or understanding of the role each application registration plays within the architecture.</li></ul> |
| Validate the overview of the used AD users and objects | <ul><li>Source Entra ID is populated with users, groups, and other entities that are used throughout the migration process.</li><li>The intended structure or list of necessary AD objects for the target environment is clearly defined.</li></ul> |
| Backup configuration data | <ul><li>A list or inventory of all critical Azure resources whose configurations need to be backed up is prepared.</li><li>Backup strategies for each type of resource are defined, determining frequency, retention period, etc.</li><li>Necessary tools and storage solutions are set up and ready for use.</li></ul> |

## Technical Assessment

### Evaluating Source and Target Azure Policies

Thoroughly assess all Azure Policy objects in both the source and target environments. This assessment should encompass custom definitions, assignments, exemptions, and compliance data. It's crucial to ensure that the policies in the target environment align with the policies in the source tenant and don't introduce any conflicts. A significant risk to be aware of is the potential for existing policies in the target environment to disrupt the workload post-migration.

Key focus areas include:

- **Policy Alignment:** Verify that the policies in the target environment are compatible with those in the source. Look for any discrepancies that might affect the migrated resources.
- **Conflict Identification:** Identify any policies in the target environment that could conflict with the incoming resources. This step is vital to prevent any operational issues after the migration.
- **Comprehensive Coverage:** Ensure the assessment covers the entire subscription, paying special attention to inherited policies that might affect the migrated resources.
- **Documentation and Backup:** Document all relevant policies and their settings for both the source and target environments. Consider creating backups of policy configurations for reference and recovery purposes.

### Reviewing Source Azure RBAC Role Assignments Before Tenant Migration

When moving a subscription to a different tenant, it's important to note that all existing role assignments, including those for users, groups, and service principals, are permanently deleted. To ensure a smooth transition and maintain appropriate access controls in the new tenant, follow these steps:

- **Mapping of Roles:** Create a comprehensive mapping of all current users, groups, and service principals in the source tenant. This mapping should correlate each entity with its counterpart in the target tenant's directory.
- **Re-creation of Role Assignments:** After the migration, you'll need to re-establish these role assignments in the target tenant. This process includes assigning roles to the corresponding users, groups, and service principals based on the mapping created.
- **Inclusion of All Roles:** Ensure that this process encompasses all types of roles. This includes not only standard roles but also any custom roles and managed identities that are in use in the source tenant.
- **Documentation:** Document the existing role assignments before the migration. This record is crucial for accurately re-creating the roles in the new tenant.

### Evaluating Azure Key Vaults Containing Certificates, Secrets, and Keys

When preparing for a tenant migration, it's important to recognize that Azure Key Vault services can't be transferred between tenants. To manage this effectively, you should document the following:

- **Applications and Services Dependencies:** Identify all applications or services that rely on these key vaults for credentials or encryption purposes. This includes any software components that directly interact with the key vaults to retrieve necessary information for their operations.
- **Automation and Deployment Scripts:** Make a list of all automation scripts or deployment pipelines that are configured to pull secrets or keys from these key vaults. This often includes CI/CD pipelines, automated deployment tools, and any scripts used for infrastructure management or application deployment.

Once this documentation has been done:

- **Recreate Key Vaults:** In the target tenant, you need to set up new Azure Key Vaults. This involves replicating the structure and contents of the existing key vaults, including all certificates, secrets, and keys.
- **Update Applications and Scripts:** Modify the applications and scripts you've identified to reference the new key vaults in the target environment. This step is crucial to ensure that these components continue to function correctly after the migration.

### Evaluating Resources Integrated with Active Directory Authentication

When preparing for a tenant migration, it's crucial to assess resources that are integrated with Active Directory (AD) authentication. These resources might be linked to a different AD and require special attention, as you can't transfer a Microsoft Entra Domain Services managed domain to a different directory. Here are the key steps to take:

- **Identify AD-Integrated Resources:** Determine which applications, services, or databases are currently using Active Directory for authentication. This includes any resource that relies on AD for user identity verification or access control.
- **Recreate Identities in New AD:** For these identified resources, plan to recreate the necessary user identities and service accounts in the new Active Directory environment. This step is essential because these identities can't be directly transferred between different AD domains.
- **Backup and Disaster Recovery:** Review and document the backup and disaster recovery mechanisms linked to these AD-integrated resources. Ensure that these mechanisms are either replicated or suitably modified in the target environment to maintain data integrity and availability.
- **Fallback Authentication Mechanisms:** Given the complexities involved in migrating AD-integrated resources, ensure that local authentication options are available as a fallback. This is important if Microsoft Entra ID Authentication (or any primary authentication method) encounters issues post-migration.
- **Update Applications and Services:** Modify applications, users, or services that authenticate using these AD-integrated resources to align with the new AD setup. This might involve updating connection strings, authentication settings, and other relevant configurations.

### Azure File Services & Data Lake Storage

It's important to address NTFS/SAS Access Control Lists (ACLs) as they need to be recreated. This involves:

- **Services Interaction:** Identify all services that read from or write data to these storage solutions. This includes applications, web services, and any other systems that interact directly with the storage where NTFS/SAS ACLs are applied.
- **Associated Data Management Tools:** Consider data pipelines, analytics tools, or backup mechanisms associated with the storage account. These tools might have specific permissions or configurations based on the existing NTFS/SAS ACLs, which need to be replicated in the new setup.

### Evaluating AKS Environments for Tenant Migration

When preparing for a tenant migration, it's important to note that Azure Kubernetes Service (AKS) clusters and their associated resources can't be transferred directly to a different directory. These elements must be recreated in the new environment. This process involves:

- **Documenting Current AKS Configuration:** Thoroughly document the existing AKS cluster configurations, including node sizes, numbers, networking setup, and any custom configurations or integrations.
- **Recreating AKS Clusters:** In the target tenant, set up new AKS clusters mirroring the configurations of the existing ones. This includes replicating the node configurations, networking setups, and any other specific settings.
- **Migrating Workloads:** Plan for the migration of workloads running on the AKS clusters. This might involve container image transfers, configuration updates, and ensuring compatibility with the new environment.
- **Updating CI/CD Pipelines:** If there's a CI/CD pipeline linked to the AKS clusters, update these pipelines to point to the newly created clusters in the target tenant.
- **Testing and Validation:** Before decommissioning the old AKS clusters, thoroughly test the new setup to ensure that all services are running as expected and that there are no disruptions to operations.

### Evaluating Entra Application Registrations for Tenant Migration

When migrating to a new tenant, it's crucial to assess Microsoft Entra application registrations, as these are bound to the specific Microsoft Entra ID and must be recreated in the new environment. This process involves:

- **Identifying Applications and Services:** Determine which applications or services are currently using these Entra (AAD) application registrations for authentication. This includes any software components that rely on Microsoft Entra ID for user identity management or access control.
- **Recreating Application Registrations:** In the target tenant's Microsoft Entra environment, recreate the application registrations. This step involves setting up new registrations that mirror the configurations of the existing ones, including redirect URIs, permissions, and any other relevant settings.
- **Updating Applications and Services:** Modify the identified applications and services to authenticate using the newly created registrations in the target tenant. This might involve updating client IDs, secrets, and other authentication parameters.

### Evaluating (Managed) Disk Encryptions for Tenant Migration

When migrating to a new tenant, special attention is required for Managed Disks that use Disk Encryption Sets for encryption with customer-managed keys. The process involves:

- **Disabling Current Encryption:** Initially, disable the Disk Encryption Set that's currently encrypting the Managed Disks. This step is necessary to prepare for the migration and reconfiguration in the new tenant.
- **Managing System-Assigned Identities:** System-assigned identities associated with the Disk Encryption Sets must be carefully managed. After disabling the current encryption, these identities need to be re-enabled or appropriately configured in the new tenant to maintain encryption continuity.
- **Backup or Snapshot Configurations:** Review and document any backup or snapshot configurations associated with these disks. Ensure that these configurations are replicated or suitably adjusted in the target environment to maintain data protection and recovery capabilities.

### Evaluating Migration Account Requirements for Tenant Migration

For a successful tenant migration, it's essential to have a designated migration account that is present in both the source and target tenants. The setup for this account involves:

- **Establishing Migration Account in Target Tenant:** The migration account should originate in the target tenant. This means it should be a native account within the target tenant's domain.
- **Inviting Migration Account to Source Tenant:** Once established in the target tenant, this account must be invited to the source tenant. It's crucial to assign the appropriate roles and permissions to this account in the source tenant, ensuring it has the necessary access to perform the migration tasks.

### Evaluating DNS and Preparing for DNS Delegation in Tenant Migration

During a tenant migration, special attention needs to be given to DNS configurations, particularly because creating the same DNS Zone in the target subscription can be challenging due to reserved zone names. Additionally, the process of reparenting for Azure DNS might not be clearly defined at the current moment. Key considerations include:

- **Identifying Dependent Services and Applications:** Determine which services or applications rely on the DNS entries in the current setup. This includes any system that uses these DNS records for locating services, load balancing, or other network-related functions.
- **Reviewing Network Configurations:** Assess network configurations or routing mechanisms that are dependent on the current DNS settings. This could involve internal routing within your network, external access configurations, or integrations with other cloud services.

### Validating Blockers for Azure Marketplace in Tenant Migration

When planning a tenant migration, it's important to consider the implications for Azure Marketplace deployments, as they might not be eligible for transfer between tenants. To address this, you should:

- **Run Precheck Report:** Conduct a precheck report for your Azure Marketplace deployments. This step is crucial to identify any potential issues or limitations that could arise during the tenant transfer process.
- **Review Private Offers:** If you have private offers from the Microsoft marketplace, ensure to precheck your account specifically for these offers. This can be done through the Microsoft Learn platform or other relevant Microsoft resources.

### Validating Blockers for Azure Reservations in Tenant Migration

When migrating tenants, it's crucial to address the status of Azure Reserved Instances (RIs), as they present specific considerations:

- **Transfer of Reserved Instances:** Machines with Reserved Instances discount are treated as normal instances in the new tenant, resulting in the loss of any reserved pricing discounts.
- **Requesting New Reserved Instances:** To maintain the cost benefits, it's advisable to request new Reserved Instances in the target tenant. This ensures that you continue to enjoy the discounted rates associated with RIs.
Canceling Current Reserved Instances: Concurrently, plan to cancel the current Reserved Instances in the source tenant. This step is necessary to avoid unnecessary charges and to align your resource management with the new tenant setup.

### Assessing Virtual Network and Network Security Groups for Tenant Migration

When migrating to a new tenant, it's important to evaluate the Virtual Network and Network Security Groups (NSGs) to ensure a smooth transition. Key aspects to consider include:

- **Non-Conflicting NSG Configurations:** Verify that the Network Security Group configurations in your current environment don't conflict with those in the target environment. This involves checking rules, policies, and any specific security settings to ensure compatibility and avoid potential network security issues post-migration.
- **Mapping Network Traffic Flows:** It's highly recommended to map out the network traffic flows, especially if there are hybrid connections involved. Understanding how data moves within and outside your network, including connections to on-premises data centers or other cloud services, is crucial for replicating or reconfiguring these flows in the target tenant.

### Assessing Azure Resource Locks for Tenant Migration

When preparing for a tenant migration, it's important to evaluate any Azure Resource Locks that are currently in place. These locks can affect the migration process and require careful handling:

- **Identify Existing Resource Locks:** Review all resources to identify any existing locks. This includes checking various resource types like virtual machines, storage accounts, and databases for any 'Delete' or 'Read-Only' locks that might be applied.
- **Plan for Removal and Re-application:** In many cases, these resource locks might need to be temporarily removed to facilitate the migration process. Plan for their removal before starting the migration and outline a strategy for reapplying them in the target environment post-migration.
Document Lock Configurations: Ensure to document the details of each lock, including the resources they're applied to and the type of lock. This documentation will be crucial for accurately re-establishing the locks after the migration.

### Evaluating Azure Monitor and Alerts for Tenant Migration

When migrating to a new tenant, it's crucial to assess the configurations of Azure Monitor and any associated alerts. This involves:

- **Identifying Monitoring Configurations:** Review all Azure Monitor configurations, including metrics, logs, and any other monitoring settings currently in place. This step is essential to ensure that you have a complete understanding of the monitoring landscape in your current environment.
- **Reviewing Alert Configurations:** Examine all alert rules and conditions set up in Azure Monitor. This includes alerts for resource performance, cost management, service health, and any custom alerts tailored to specific needs.
- **Documenting Incident Response Mechanisms:** Pay special attention to any incident response or on-call mechanisms that are tied to these alerts. This might include integrations with ticketing systems, notification services, or automated response workflows.

### Evaluating Billing and Cost Management for Tenant Migration

When transitioning to a new tenant, it's important to assess all aspects of billing and cost management to ensure a smooth financial transition. This includes:

- **Identifying Current Configurations:** Review all settings and configurations related to cost management and billing in your current environment. This encompasses budget alerts, cost analysis tools, and any custom reports or dashboards you have set up.
- **Planning for Re-configuration:** Recognize that these configurations need to be re-established in the post-migration environment. This step is crucial, especially if there isn't an overarching policy that automatically transfers these settings.
- **Understanding Billing Ownership:** The steps for transitioning billing and cost management settings might vary depending on who owns the billing account (for example, self-service or enterprise agreement). Understanding the nature of your billing ownership will guide the specific actions required during the migration.

### Evaluating External Dependencies for Tenant Migration

In the context of migrating to a new Azure tenant, it's crucial to assess any external dependencies that might be impacted by the move. This involves:

- **Identifying Third-Party Services:** Determine if there are any non-Microsoft services that interact with your Azure resources. This could include SaaS applications, external APIs, or other cloud services that are integrated with your Azure environment.
- **Reviewing On-Premises Infrastructure:** Consider any on-premises infrastructure that is connected to your Azure resources. This might involve direct connections, VPNs, or data synchronization setups that need to be adjusted or reconfigured for the new tenant.
- **Assessing Other Cloud Interactions:** If your Azure resources are part of a multicloud strategy, evaluate how they interact with resources in other cloud environments. Ensure that these connections and integrations are accounted for and planned in the migration process.

### Evaluating Azure DevOps and CI/CD Pipelines for Tenant Migration

When migrating to a new Azure tenant, it's important to assess the impact on Azure DevOps and any Continuous Integration/Continuous Deployment (CI/CD) pipelines. This involves:

- **Reviewing Pipelines and Integrations:** Examine all CI/CD pipelines and their integration with Azure DevOps. Determine if changes or reconfigurations are necessary post-migration. This includes reviewing build and release pipelines, and any automated testing or deployment processes.
- **Assessing Source Code Repositories:** Identify any source code repositories linked to these pipelines. Ensure that access to these repositories is maintained or appropriately modified in the new tenant environment.
- **Evaluating Deployment Targets:** Consider the deployment targets of these pipelines. This might involve Azure resources, such as web apps, functions, or containers, which will be affected by the tenant migration.
- **Reviewing Integrated Tools:** Look into any additional tools or services integrated with your CI/CD pipelines. This could include code analysis tools, third-party deployment services, or monitoring solutions.
- **Considering Teams and Processes:** Understand how teams or processes rely on these pipelines and DevOps data. Ensure that the necessary teams are informed about the migration and are prepared for any temporary disruptions or changes in workflow.

### Reviewing Shared Dashboards and Reports for Tenant Migration

In the context of migrating to a new Azure tenant, it's important to assess the shared dashboards and reports, such as those created with Power BI or other analytics tools. This involves:

- **Identifying Shared Dashboards:** Determine if there are any shared dashboards within your current Azure environment. This includes dashboards used for monitoring, management, or data visualization purposes.
- **Evaluating PowerBI Reports:** Review any Power BI reports or other analytics reports that are being used. Assess how these reports are integrated with your Azure resources and data sources.
- **Understanding Impact Post-Migration:** Recognize that these resources might be affected after the migration. This could be due to changes in data sources, permissions, or integrations with other Azure services.
- **Planning for Reconfiguration or Updates:** Prepare to update or reconfigure these dashboards and reports in the new tenant. This might involve reconnecting data sources, updating permissions, or modifying integrations.

### List other known resources

To list other Azure resources with known Microsoft Entra dependencies, you can utilize the Azure Resource Graph, which is accessible through the az graph extension in Azure CLI. This tool allows you to query and explore your Azure resources across multiple subscriptions efficiently.

### Documenting a Plan for Full Diagnostic System and Application Testing After Reparenting

After reparenting resources during a tenant migration, it's crucial to have a comprehensive plan for diagnostic system and application testing. This ensures that all transferred resources are functioning correctly in the new environment. The plan should include:

- **Functionality Tests:** Develop a suite of functionality tests that cover all aspects of the system and applications. These tests should be designed to verify that each component is operating as expected post-migration.
- **Automation of Tests:** Wherever possible, automate these tests. Automated testing provides a consistent and efficient way to validate the functionality of each resource. It also allows for repeated testing without additional manual effort.
- **Dependency Coverage:** Ensure that the tests touch upon all dependencies. This includes internal dependencies within the system and external dependencies, such as non-Microsoft services, databases, and network connections.
- **Resource-Specific Testing:** Tailor tests to cover each resource that has been transferred. Different resources might require different testing approaches. For instance, testing a database migration might involve data integrity checks, while testing a web application might focus on user interface and API interactions.
- **Performance and Load Testing:* Include performance and load testing to ensure that the system operates effectively under expected workloads in the new environment.
- **Security Testing:** Conduct security testing to ensure that all security measures are functioning correctly post-migration. This includes testing access controls, encryption, and other security protocols.
- **Documentation of Test Results:** Document the results of all tests. This documentation should include details of the test scenarios, execution process, and any issues identified. It's crucial for diagnosing any problems and for audit purposes.
- **Plan for Addressing Issues:** Establish a process for addressing any issues uncovered during testing. This might involve bug fixes, configuration changes, or additional migrations.

## Premigration steps

| Action | Description |
| --- | --- |
| Match and recreate Azure Polices | <ul><li>If certain policies conflict in the target environment, temporarily disable them and address the conflict before reapplying. Not accurately recreating policies can lead to compliance issues or other operational problems in the target environment.</li><li>Compare the policy definitions and assignments between the source and target environments to ensure they match.</li><li>Perform tests by attempting to deploy resources that should either comply or conflict with the recreated policies to validate their effectiveness.</li></ul> |
| Recreate and Assign any Azure RBAC Role Assignments | <ul><li>Users, groups, and service principals in the target environment have the appropriate roles assigned to them, matching the source environment. If misconfigured, use saved documentation from the source environment as a reference to correct assignments.</li><li>Not accurately recreating and assigning roles can lead to access issues, either granting too much or too little access to resources in the target environment.</li><li>Overlooking certain custom roles or unique assignments can compromise security or functionality.</li><li>Compare role assignments between the source and target environments to ensure they match.</li><li>Regularly audit the role assignments in the target environment for a period after migration to catch any lingering issues.</li></ul> |
| Recreate and assign Managed Identities | <ul><li>If managed identities face authentication issues, verify the identity's status and ensure it's properly linked to its associated resource.</li><li>Failure to accurately recreate managed identities can result in authentication or authorization issues in applications or services relying on these identities.</li><li>Misconfigured permissions or roles for the managed identities can lead to access issues or potential security vulnerabilities.</li><li>List all system-assigned managed identities in the target environment and ensure they match the documented identities from the source environment.</li><li>Test resources associated with managed identities to ensure they can authenticate and authorize correctly.</li><li>Continuously monitor for any authentication or authorization failures related to the managed identities post-migration.</li></ul> |
| Recreate Azure Key Vaults with Certs, Secrets, and Keys | <ul><li>If a secret, key, or certificate is missing or fails, import or add it manually from the source documentation or backup.</li><li>Missing or misconfigured secrets, keys, or certificates can lead to application or service failures.</li><li>Incorrect access policies can result in unauthorized access or potential security vulnerabilities.</li><li>List all Key Vaults in the target environment and ensure their structure and content match the source documentation.</li><li>Test applications or services relying on these Key Vaults to ensure they can access and retrieve required configurations.</li><li>Verify the correct access policies for each Key Vault, ensuring that only authorized entities can access the contents.</li></ul> |
| Recreate the AKS environments | <ul><li>If the AKS cluster fails to create or is misconfigured, refer to the backup or exported configuration from the source and try the setup process again.</li><li>For failed workloads, review the deployment configurations and ensure they match the source settings. Address any missing dependencies or configurations.</li><li>If networking or storage issues arise, double-check their configurations against the source environment and adjust as necessary.</li><li>Workloads not being correctly configured can lead to application failures or degraded performance.</li><li>Networking misconfigurations can result in accessibility issues or potential security vulnerabilities.</li><li>If storage is misconfigured, data persistence might be affected, leading to potential data loss or inconsistencies.</li><li>Deploy a test workload or use existing workloads to ensure they operate as expected.</li><li>Validate networking and storage by interacting with applications hosted on the AKS cluster and ensuring data persistence and network communication function as expected.</li></ul> |
| Assess any Entra application registrations | <ul><li>If any application registration details are missed or incorrectly documented, revisit the source Microsoft Entra ID environment to correct the oversight.</li><li>Overlooking an application registration could result in services or applications not functioning correctly post-migration.</li><li>Incorrectly setting up permissions or roles for an application registration in the target environment could pose security risks.</li><li>Test the applications or services associated with these registrations to ensure they're functioning as expected in the new environment.</li></ul> |
| Validate the overview of the used AD users and objects | <ul><li>If discrepancies are found, reference the overview obtained from the source AD and make necessary corrections in the target AD.</li><li>Missing any AD user or object might lead to permission issues, broken integrations, or inaccessible resources post-migration.</li><li>Ensure no critical user or object is missed or incorrectly mapped.</li><li>Verify the roles, group memberships, and permissions of key users to ensure they match the expectations for the new environment.</li></ul> |
| Backup configuration data | <ul><li>If any backup fails, check the logs or error messages, rectify the issue (for example, insufficient permissions, storage issues), and rerun the backup.</li><li>If a backup storage solution becomes inaccessible, ensure you have redundancy in place or switch to an alternative backup storage solution.</li><li>Failure to back up configuration might lead to longer recovery times if there's any post-migration issue or misconfiguration.</li><li>Verify the completeness of backups against the inventory of critical resources.</li><li>Ensure backup metadata (for example, timestamps, locations) matches expectations and the data can be retrieved quickly when needed.</li></ul> |

### Go/No-Go Decision for Migration

As the critical juncture of the migration process approaches, the emphasis shifts to the Go/No-Go decision, a pivotal moment where stakeholders evaluate whether to proceed with the migration or to postpone it. This decision is contingent on thorough preparation, comprehensive stakeholder involvement, and meticulous assessment of all premigration tasks and potential risks.

#### Stakeholder Involvement and Communication

Stakeholders play a crucial role in the decision-making process. Their insights, derived from their respective domains, contribute significantly to a well-informed decision.
Effective communication is essential. This includes notifying stakeholders about planned downtime and any changes, utilizing established communication channels to ensure clarity and transparency throughout the process.

#### Pre-Migration Task Completion

The Go/No-Go decision is fundamentally based on the completion of all premigration tasks. This ensures that every aspect of the migration is accounted for and adequately prepared.
It's also vital to address or mitigate any potential blockers or significant risks that could affect the migration.
Making the Decision:

The decision should be clear-cut: either proceed with the migration or delay/postpone it ("No go"). This decision is based on reports and validations from all prior steps.
In the event of a "No go" decision, it's crucial to establish feedback loops to identify and address the reasons behind this choice.

#### Post-Decision Actions

If "No go" is the chosen path, immediately define a plan to tackle the highlighted concerns or blockers. This includes rescheduling another "Go/No go" meeting after resolving these issues.
Ensure that all teams are aligned and ready for the rescheduled migration, maintaining cohesion and readiness across the board.
Risks and Considerations:

A premature "Go" decision might lead to unforeseen complications during the migration, whereas repeated "No go" decisions could have financial or operational repercussions.
It's imperative to avoid miscommunication or lack of clarity during this process, as the lack of clarity could lead to incorrect assumptions and misguided actions.

#### Final Checks

Prior to the final decision, it's essential to consider feedback from all involved teams and stakeholders.
Confirm that all technical checks, backup verifications, and other preparatory tasks are successfully completed.
Finally, cross-verify with contingency plans to ensure comprehensive readiness for any complexities that might arise during the migration.
This process underscores the importance of a balanced and well-informed Go/No-Go decision, setting the stage for a smooth and successful migration.

### Test Migration Strategy

Implementing a test migration is a strategic approach in certain scenarios. It allows for testing the resource move, gaining insights into the potential impact, and proactively identifying and addressing blockers before the actual migration.

#### Preparation for Test Migration

**Shutdown of Resources:** If feasible, shut down resources to simulate the migration environment. This step includes ceasing connectivity to resources outside the subscription environment.

**Dependency Awareness:** Understand the applications/services running on these resources and their dependencies to ensure a comprehensive approach.

#### Precheck for Test Migration

- Compile a list of all resources that can be safely shut down without disrupting essential services.
- Gain a deep understanding of the interdependencies of Azure resources, including the current network topology and connectivity routes to external resources.
- Ensure that a backup or snapshot of the configurations and data of the resources is taken to safeguard against potential data loss.
- Inform stakeholders and resource owners about the planned downtime, emphasizing the nature and purpose of the test migration.
- Establish and communicate a maintenance window, ensuring all parties are aware and prepared.

#### Post-check for Test Migration

Confirm that no services or features relying on these resources are accessible, indicating a successful simulation of the migration environment.

If a resource fails to shut down properly, assess the cause, such as any locks, dependencies, or errors, and take corrective actions based on the type of resource and error identified.

Resources interconnected with others might cause cascading failures or inaccessibility.
Acknowledge the potential for data loss or inconsistencies if resources are shut down without proper preparation.
Consider the risk of potential business disruption, particularly if critical resources are inadvertently shut down.

Upon completion of the test migration process, check the status of each resource in the Azure portal or Azure CLI to ensure they are in the desired state. Validation is crucial to confirm the effectiveness of the test migration and to identify any areas that require further attention before proceeding with the actual migration.

### Migrate subscriptions

- Announce maintenance window.
- Remove resources from subscription that aren't transferable, see list of supported resources.
- Shut down or pause (dependent) resources if possible.
- Cease connectivity to resources inside and outside the source subscription environment.
- Validate successful transfer.
- Check Ownership of Azure Subscription in new tenant.
- Start resources that were shut down or pauses.
- Validate and Assign any Azure RBAC (Custom) Role Assignments and Managed Identities.
- Assign any Active Directory objects to resources (if necessary).
- Validate Azure Key Vaults with Certs, Secrets, and Keys.
- Assign Certs, Secrets, and Keys to resources.
- Validate encryption with Customer keys on disk and other resources.
- Deploy new resources that weren't supported for transfer.
- Assign/Configure any Entra application registrations.
- Run full diagnostics system and application test plan.

### Roll back

There's no rollback mechanism in place, instead, performs the same actions in reverse if needed.

### Post-migration Steps

#### Draft Migration Report

All stages of the migration are complete, from premigration assessments to post-migration validations.

- A comprehensive migration report is available detailing every phase, action, outcome, challenges faced, and recommendations.
- The report is shared with stakeholders for review and feedback.
- If feedback from stakeholders highlights gaps or missed areas in the report, revisit the migration process to fill in those gaps.
- Incorrect or misleading information in the report can affect future decision-making or stakeholder trust.
- Ensure the report covers every phase of the migration.
- Review the report for accuracy and clarity.
- Validate the data in the report with logs, system outputs, and feedback from involved teams.
- Obtain sign-offs from key personnel or stakeholders affirming the report's accuracy and completeness.
- Check with Product Owners on any open items.

### Supported resources

| Resource type  | Transferable to new tenant |
| ------------- | ------------- |
| Virtual Machines  | Yes  |
| Virtual Network  | Yes  |
| Azure Kubernetes Service  | No  |
| Key Vault  | No  |
| Managed Identities  | No  |
| Azure Policy Definitions  | No  |
| Load Balancer  | Yes  |
| Application Gateway  | Yes  |
| Alerts | No  |
| Billing Accounts  | No  |
