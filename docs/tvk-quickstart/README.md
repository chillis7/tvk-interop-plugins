# TrilioVault for Kubernetes tvk-quickstart plugin

The tvk-quickstart kubectl plugin is tested with OCP, RKE, GKE, and DO Kubernetes clusters. The plugin achieves the following:

- Installs TrilioVault for Kubernetes (TVK) - TVK Operator and TVM Application
- Configures the TVK Management Console (UI)
- Creates a target (NFS or S3)
- Executes sample backup and restores TVK operations

## Pre-requisites:

1. All pre-requisites for the TVK installation can be found [here](https://docs.trilio.io/kubernetes/use-triliovault/installing-triliovault#prerequisites-for-triliovault-for-kubernetes).
2. Install S3cmd. from [here](https://s3tools.org/s3cmd).
3. yq must be version 4 or above. Refer to [here](https://github.com/mikefarah/yq) for further information.
4. oc is required if running against OCP cluster - Install oc from [here](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/).
5. Linux or macOS are supported. Windows is not supported at this time.

## What is Executed by the tvk-quickstart?

1. **Preflight check**: Performs prerequisite checks to ensure that all TVK requirements are satisfied.
2. **TVK Installation**: Installs TVK, TVM (UI), and free trial license.
3. **TVK Management Console Configuration**: Configures control access to the UI. Users can choose either Loadbalancer, Nodeport, or PortForwarding.
4. **Target Creation**: Creates and validates the target storage for storing backup images. Users can create an S3 (DigitalOCean Spaces / AWS S3) or an NFS-based target.
5. **Run Backup and Restore of Sample Applications**: Run sample tests for a label, namespace, operator, and helm-based applications. By default, backups and restores are performed against:
   - Label-based MySQL Database application
   - Namespace-based WordPress application
   - Operator-based MySQL Database
   - Helm-based MongoDB based application


## Installation, Upgrade, and Removal of tvk-quickstart kubectl Plugin
Krew is the plugin manager for `kubectl` command-line tool. The tvk-quickstart installation can be executed with or without krew installed, but installion with krew is recommended. However, if there are any issues with your installation of krew, then you may still install the tvk-quickstart kubectl plugin without krew.

### Using krew

| Action                                       | Command   
| :------------------------------------------  |:-------------
| Add the TVK custom plugin index of krew      | `kubectl krew index add tvk-interop-plugin` https://github.com/trilioData/tvk-interop-plugins.git
| Perform the installation                     | `kubectl krew install tvk-interop-plugin/tvk-quickstart`
| Upgrade the tvk-quickstart plugin            | `kubectl krew upgrade tvk-quickstart`
| Upgrade the tvk-quickstart plugin            | `kubectl krew uninstall tvk-quickstart`

### Without krew

If the krew plugin manager is not an option, you may still install the tvk-quickstart kubectl plugin without krew using the following steps.

1. List of available releases: https://github.com/trilioData/tvk-interop-plugins/releases.
2. Choose a version of the tvk-quickstart plugin to install and check if release assets have a tvk-quickstart plugins package[tvk-quickstart.tar.gz].
3. Set env variable `version=[TVK_QS_VERSION]`. If the `version` is not exported, then the `latest` tagged version will be considered.
4. Run this Bash or ZSH shells command to download and install tvk-quickstart without krew

   ```bash
   (
     set -ex; cd "$(mktemp -d)" &&
     if [[ -z ${TVK_QS_VERSION} ]]; then TVK_QS_VERSION=$(curl -s https://api.github.com/repos/trilioData/tvk-interop-plugins/releases/latest | grep -oP '"tag_name": "\K(.*)(?=")'); fi &&
   echo "Installing version=${TVK_QS_VERSION}" &&
   curl -fsSLO "https://github.com/trilioData/tvk-interop-plugins/releases/download/"${TVK_QS_VERSION}"/tvk-quickstart.tar.gz" &&
   tar zxvf tvk-quickstart.tar.gz && sudo mv tvk-quickstart/tvk-quickstart /usr/local/bin/kubectl-tvk_quickstart
   )
   ```
5. Verify installation using the command: `kubectl tvk-quickstart --help`

## Modes of Operation
There are two modes to use the tvk-quickstart plugin:
1. Interactive
2. Non-interactive


### Interactive

1. For interactive installation of TVK operator and manager, configure the TVK UI, create a target and run a sample backup restore, using the command: `kubectl tvk-quickstart [options]`
2. During interactive execution, the user is prompted for input for various options that enable the plugin to perform installation and deployment operations. Use the information provided in the following table to guide you in these options:       

| Parameter                     | Description   
| :---------------------------- |:-------------
| -i, --install_tvk             | Installs TVK and its free trial license. TVK will be installed as upstream operator. A cluster scope TVM custom resource triliovault-manager is also created.
| -c, --configure_ui            | Configures the TVK UI. This option will only work if the cluster has TVM application installed.
| -t, --target                  | Creates target for backup and restore jobs.
| -s, --sample_test		| Creates sample backup and restore jobs.
| -p, --preflight	        | Checks if all the TVK pre-requisites are satisfied

A user may specify more than one option with each tvk-quickstart execution. For example, to install, configure, create a target and run samples quickly, execute the following single command:_ `kubectl tvk-quickstart -i -c -t -s`

###Non-interactive###
In non-interactive mode, tvk-quickstart performs all operations; preflight checks, installation, UI configuration, and runs sample backup and restore tests.  However, all the user input is provided in a configuration file. Use the following steps to invoke tvk-quickstart in the non-interactive mode:

1. Create an `input_config` file, using the sample [input_config file](https://github.com/trilioData/tvk-interop-plugins/blob/main/tests/tvk-quickstart/input_config) as a template. The non-interactive mode leverages values in the `input_config` file.
2. Edit any variables in the configuration file.  Each variable description and the possible values of the variable is described in the following table.
3. Run the following command to execute the plugin in a non-interactive method: `kubectl tvk-quickstart -n` 
4. The user is now prompted to provide the path of the `input_config` file.

## Parameters For `input_config`

| input-config Parameter            | Description   
| :-------------------------------- |:-------------
| PREFLIGHT                         | This parameter checks whether or not preflight should be executed. It accepts one of the following values: [True, False]. More info around this can be found [here](https://github.com/trilioData/tvk-plugins/tree/main/docs/preflight)
| proceed_even_PREFLIGHT_fail       | This option is dependent of PREFLIGHT execution. If the user wishes to proceed despite checks failing in preflight execution, this variable must be set to y/Y. This variable accepts one of the following values [Y,y,n,N].
| TVK_INSTALL                       | Set this option to True to install TVK and TVM. Otherwise, set this value to False.
| CONFIGURE_UI                      | This parameter is to check whether or not TVK UI should be configured. It accepts one of the value from [True, False]
| TARGET		            | This parameter is to check whether or not TVK Target should be created. It accepts one of the value from [True, False]. ***Note***: Target type "Readymade_Minio" requires 4GB per node, else the target creation will fail.
| SAMPLE_TEST		            | This parameter is to check whether or not sample test should be executed. It accepts one of the value from [True, False]
| storage_class		            | This parameter expects storage_class name which should be used across plugin execution. If kept empty, the storage_class annoted with 'default' label would be considered. If there is no such class, the plugin would likely fail.
| operator_version		    | This parameter requires the user to specify the TVK operator version to install as a part of tvk installation process. The compatibility/version can be found [here](https://docs.trilio.io/kubernetes/use-triliovault/compatibility-matrix#triliovaultmanager-and-tvk-application-compatibility). If this parameter is empty, by default TrilioVault operator version  2.1.0 will get installed.
| triliovault_manager_version       | This parameter requires the user to specify the TVK manager version to install as a part of tvk installation process. The compatibility/version can be found [here](https://docs.trilio.io/kubernetes/use-triliovault/compatibility-matrix#triliovaultmanager-and-tvk-application-compatibility). If this parameter is empty, by default TrilioVault operator version  2.1.0 will get installed.
| tvk_ns 		            | This parameter requires the user to specify the namespace in which tvk is to be installed.
| if_resource_exists_still_proceed  | This parameter is to check whether the plugin should proceed for other operations, even if resources exists. It accepts one of the value from [Y,y,n,N]
| ui_access_type		    | Specify the way in which TVK UI should be configured. It accepts one of the value from ['Loadbalancer','Nodeport','PortForwarding']
| domain        		    | The value of this parameter is required when 'ui_access_type == Loadbalancer'. Specify the domain name which has been registered with a registrar and in which you wish to create a record. More information about this parameter can be found [here](https://docs.digitalocean.com/products/networking/dns/).
| tvkhost_name 		            | The value of this parameter is required when 'ui_access_type == Loadbalancer OR ui_access_type == Nodeport'. The value of this parameter is the hostname via which the TVK management console will be accessible using a web browser.
| cluster_name		            | The value of this parameter is required when 'ui_access_type == Loadbalancer OR ui_access_type == Nodeport'. If kept blank, the active cluster name will be used.
| vendor_type		            | The value of this parameter is required to create a target. Specify the vendor name under which the target must be created. Currently supported value is for ['Digital_Ocean','Amazon_AWS'].
| doctl_token		            | Digital Ocean API token, which the user can generate in the control panel [here](https://cloud.digitalocean.com/account/api/tokens) for user authentication.
| target_type		            | Target is a location where TrilioVault stores backup. Specify the type of target to create. It accepts one of the values from ['NFS','S3']. More information can be found [here](https://docs.trilio.io/kubernetes/getting-started/getting-started-1#step-2-create-a-target).
| access_key		            | This parameter is required when 'target_type == S3'. This is used for bucket S3 access/creation. The value should be consistent with the vendor_type that you select.
| secret_key		            | This parameter is required when 'target_type == S3'. This is used for bucket S3 access/creation. The value should be consistent with the vendor_type that you select.
| host_base		            | This parameter is required when 'target_type == S3'. Specify the s3 endpoint for the region that your Spaces/Buckets are in. More information can be found [here](https://docs.digitalocean.com/products/spaces/resources/s3cmd/#enter-the-digitalocean-endpoint).
| host_bucket		            | The value of this parameter should be a URL template to access s3 bucket/spaces. This parameter is required when 'target_type == S3'. Generally its value is '%(bucket)s.<value of host_base>' . This is the URL to access the bucket/space.
| gpg_passphrase		    | This parameter is for an optional encryption password. Unlike HTTPS, which protects files only while in transit, GPG encryption prevents others from reading files both in transit and when they are stored.  More information can be found @[here](https://docs.digitalocean.com/products/spaces/resources/s3cmd/#optional-set-an-encryption-password).
| bucket_location		    | Specify the location where the s3 bucket for the target should be created. This parameter is specific to AWS vendor_type. The value can be one of the follwing: ['us-east-1', 'us-west-1', 'us-west-2', 'eu-west-1', 'eu-central-1', 'ap-northeast-1', 'ap-southeast-1', 'ap-southeast-2', 'sa-east-1'].
| bucket_name		            | The name that a user wants to give to a bucket for target creation.
| target_name		            | Specify the name for the target that needs to be created.
| target_namespace		    | Specify the name of the namespace in which the target should be created in. The user must have permission to create, modify, and access the namespace.
| nfs_server		            | The server IP address or the fully qualified nfs server name for NFS target creation.
| nfs_path		            | Specify the exported path which can be mounted for target creation. This parameter is required when 'target_type == NFS'.
| nfs_options		            | Specify if any other NFS option needs to be set. Additional values for the nfsOptions field can be found [here](https://docs.trilio.io/kubernetes/architecture/apis-and-command-line-reference/custom-resource-definitions-application-1#triliovault.trilio.io/v1.NFSCredentials).
| thresholdCapacity		    | Capacity at which the IO operations are performed on the target. Units supported - [Mi,Gi,Ti]
| bk_plan_name		            | Specify the name for backup plan creation for the sample application. Default value is 'trilio-test-backup'.
| bk_plan_namespace		    | Specify the namespace in which the application should be installed and in which the backup plan will be created. The default value is 'trilio-test-backup'.
| backup_name		            | Specify the name for the backup which will be created for the sample application. The default value is 'trilio-test-backup'.
| backup_namespace		    | Specify the namespace in which the backup should be created. The default value is 'trilio-test-backup'.
| backup_way		            | Specify the way in which the backup should be taken. Supported values are as follows: ['Label_based','Namespace_based','Operator_based','Helm_based']. For Label_based, the MySQL application is installed and the sample backup/restore will be showcased. For Namespace_based, the WordPress application is installed and the sample backup/restore will be showcased. For Operator_based, MySQL operator is installed and the sample backup/restore will be showcased. For Helm_based, the MongoDB application is installed and the sample backup/restore will be showcased.
| restore		            | Specify whether or not restore should be executed. Valid values are [True, False].
| restore_name		            | Specify the name for the restore. The default value is 'tvk-restore'.
| restore_namespace		    | Specify the namespace in which the backup should be restored. The default value is 'tvk-restore'.
