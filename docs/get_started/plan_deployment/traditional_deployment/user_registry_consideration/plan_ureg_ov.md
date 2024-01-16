# User registry options

With HCL Digital Experience, you can complete various security configuration tasks. In previous releases, only one task was available and you might not recover if errors occurred. Also, you were unable to expand your user registry to meet growing business needs. With HCL DX version 9.5 and later, there are multiple tasks and you can fine-tune your system to meet your business needs.

You can choose from the following general security options:

|Security option|Explanation|
|---------------|-----------|
|Federated security|With this option, you can create virtual portals with multiple realms. You can also use multiple repositories \(LDAP, database, custom\), and you can add application groups to your system. This option is works well if you must merge multiple LDAP servers into one cohesive structure. **Attention:** If you plan to enable the transient user feature, you must choose the federated user registry configuration. <br>**Important:** You must ensure that no duplicate names exist on the various repositories. For example, if you installed the product with a portal administrator of admin1, then admin1 must not exist on the corporate LDAP server.|
|Custom security|This option provides you with the ability to write a fully controlled WebSphere Security environment. There is a custom user registry and a custom member adapter for Virtual Member Manager \(VMM\). The abilities of this option depend on your implementation.|

## Federated security

HCL Portal is configured with a default federated repository with a built-in file repository. The federated repository offers the richest number of options to meet your business needs. You can easily expand your business as your needs grow. For example, your company acquires a new business that has an existing LDAP user registry. You can add that LDAP server to your federated repository. Choose one of the following tasks to enable a production repository:

|Task|Description|
|----|-----------|
|Add a federated LDAP repository to the VMM configuration|Select this option to add an LDAP server to the federated repository. This task does not change the current security assignment. Therefore, the administrative user that is defined during installation is still active.|
|Add a federated database repository to the VMM configuration|Select this option to add a database to the federated repository. This task does not change the current security assignment. Therefore, the administrative user that is defined during installation is still active.|
|Add a federated custom user registry|Select this option to add a custom user registry that your company created to the federated repository. This task does not change the current security assignment. Therefore, the administrative user that is defined during installation is still active.|

After you add your initial user registry, you can add more user registries to the repository to create a multiple user registry configuration. After you configure your repository, you must remove the default file-based repository. You do not have to remove the file-based repository in a development environment or if you are using [HCL Connections](https://www.hcltech.com/software/acquisition-updates). The following tasks are required to remove the default file-based repository:

|Task|Description|
|----|-----------|
|Change the user registry where users and groups are stored|This task changes the default repository where new users and groups are stored.|
|Change WebSphere Application Server administrator|This task changes the WebSphere® Application Server administrator user ID and password.|
|Change HCL Portal Server administrator|This task changes the HCL Portal administrator user ID and password.|
|Delete a federated repository from the VMM configuration|This task deleted the default file-based repository from your configuration.|

After you use your federated repository, you might need to manage your user registry. You can run any of the following optional tasks to fine-tune your federated repository:

|Task|Description|
|----|-----------|
|Updating the federated LDAP user registry|Choose this option to update certain parameters such as your bind ID and password to fix issues with your LDAP user registry.|
|Updating the federated database user registry|Choose this option to update certain parameters such as the data source name, database URL, and database type to fix issues with your database user registry.|
|Create a realm|Choose this option to create a realm, which is a group of users from one or more user registries that form a coherent group within HCL Portal. Realms and provides flexible user management with various configuration options. A realm must be mapped to a virtual portal to allow the defined users to log in to the virtual portal. In a federated repository, you can create multiple realms.|


