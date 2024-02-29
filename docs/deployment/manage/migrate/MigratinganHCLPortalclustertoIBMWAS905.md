# Migrating an HCL Digital Experience Portal cluster to IBM WAS 9.0.5


You can migrate HCL Digital Experience (DX) profile to WebSphere Application Server 9.0.5 through local or remote migration process.

## Before you begin

Make sure that you migrate your Search Collection before you migrate your HCL Digital Experience server. After you migrate your search collection, log in to the Deployment Manager WebSphere Integrated Solutions Console and turn off AutoSync.

After you migrate your search collection, log in to the Deployment Manager WebSphere® Integrated Solutions Console and turn off the AutoSync.

## About this task

For these instructions, the following terms are used:

**v85_dmgr_profile_name**
This term refers to the original DMGR profile on the WebSphere® Application Server v8.5.5 or higher installation.

**v90_dmgr_profile_name**
This term refers to the new DMGR profile on the WebSphere® Application Server 9.0.5 or higher installation.

**v85_dmgr_profile_path**
This term refers to the full path to the WebSphere® Application Server v8.5.5.5 or higher deployment manager installation.

**v90_dmgr_profile_path**
This term refers to the full path to the WebSphere® Application Server 9.0.5 or higher deployment manager installation.

**v85_wp_profile_name**
This term refers to the original HCL Digital Experience profile on the WebSphere Application Server.

**v85_wp_profile_path**
This term refers to the full path to the v85 profile.

**v90_wp_profile_path**
This term refers to the full path to the v90 profile.

**v85_was_root_path**
This term refers to the full path to the WebSphere Application Server v8.5.5.5 or higher installation.

**v90_was_root_path**
This term refers to the full path to the WebSphere Application Server 9.0.0.2 or higher installation.

!!! important
    The WebSphere® Application Server 9.0.5 migration is different from the standard HCL Digital Experience migration. The source environment is updated to point to the target WebSphere® Application Server 9.0.5 environment.

## Procedure

1. Install WebSphere® Application Server 9.0.5 on the DMGR target migration server.

    !!! note 
        Ensure that HCL Digital Experience 9.0 and the WebSphere binary files for each cluster node are also installed.

    !!! important
    **For a remote migration only**: During installation on the target system, use the same WebSphere Application Server, Portal Server, and wp_profile directory paths as the source server.

2. Complete the following steps to migrate the Deployment Manager:

    1. On the DMGR target migration server, create a DMGR profile by using the **manageprofiles** command-line option.

        !!! important
            Use the exact same cell name and node name that you used for the WebSphere® Application Server v8.5.5 or higher DMGR installation. If you are running this command on IBM® i, you must add the **-serverName HCL Portal and HCL Web Content Manager** parameter.

        - AIX®: `./manageprofiles.sh -create -profileName v90_dmgr_profile_name -profilePath v90_dmgr_profile_path -templatePath v90_was_root_path/profile Templates/management -nodeName source_node_name -cellName source_cell_name -hostName host_name -isDefault`
            
        - Linux™: `./manageprofiles.sh -create -profileName v90_dmgr_profile_name -profilePath v90_dmgr_profile_path -templatePath v90_was_root_path/profile Templates/management -nodeName source_node_name -cellName source_cell_name -hostName host_name -isDefault`
            
        - Windows™: `manageprofiles.bat -create -profileName v90_dmgr_profile_name -profilePath v90_dmgr_profile_path -templatePath v90_was_root_path/profile Templates/management -nodeName source_node_name -cellName source_cell_name -hostName host_name -isDefault`

        Where the following values are set:
        
        **source_node_name**

        The node name on the source installation.
        
        **source_cell_name**

        The cell name on the source installation.
        
        **host_name**

        The host name of the environment.

    2. Open a command prompt.

        !!! note 
            If you are instructed to open a properties file, they are ASCII files. Open them with the appropriate tool.
    
    3. Copy the **PortalServer/filesForDmgr/filesForDmgr.zip** file from the primary HCL Digital Experience node to the WebSphere® Application Server 9.0 DMGR server and extract it into the **v90_was_root_path** directory.

    4. Ensure OSGi cache was cleared from the new WebSphere® Application Server 9.0 installation by running the following command from the **v90_dmgr_profile_name/bin** directory:
        
        - AIX®: `./osgiCfgInit.sh -all`
        
        - Linux™: `./osgiCfgInit.sh -all`

        - Windows™: `osgiCfgInit -all`

    5. **For a remote migration only**: On the WebSphere® Application Server 9.0 DMGR server, run the following command to create the WAS_V90_RemoteMigrSupport.jar file.

        - AIX®: `cd v90_was_root_path/bin/migration/bin ./createRemoteMigrJar.sh -targetDir remote_zip_dir -includeJava -allPlugins`
        
        - Linux™: `cd v90_was_root_path/bin/migration/bin ./createRemoteMigrJar.sh -targetDir remote_zip_dir -includeJava -allPlugins`
        
        - Windows™: `cd v90_was_root_path\bin\migration\bin createRemoteMigrJar.bat -targetDir remote_zip_dir -includeJava -allPlugins`
    
        Where `remote_zip_dir` is an existing directory that is used to contain the generated file. Make sure that you specify the full path to the directory. Copy the WAS_V90_RemoteMigrSupport.jar file from the WebSphere® Application Server 9.0 DMGR target server to the WebSphere® Application Server DMGR source server. Extract the JAR file into a working directory. For example, run `unzip WAS_V90_RemoteMigrSupport.jar`.

        **Linux only**: Ensure that read and run permissions are set on the extracted files. For example, run chmod -R 755 supp_dir.

    6. The WASPreUpgrade and WASPostUpgrade commands that are used in HCL Digital Experience migration use much memory and might result in an OutOfMemoryException. To prevent this issue from happening, complete the following steps:

        1. Ensure that you are using a large enough heap size. Generally, using 2 GB heap size is sufficient, specified by using the `-javaoption -Xmx2048m` variable when you run **WASPreUpgrade** or **WASPostUpgrade** commands. Increase the Heap Size by increasing the `-Xmx2048m` variable. A possible value might be `-Xmx4096m`.
        
        2. Ensure that the size of the Deployment Manager profile was minimized by deleting unnecessary files. In most cases, the Deployment Manager profile (Dmgr01) should not be larger than 2 GB. It is a good idea to do a search for files larger than 10 MB to find any possible files that can be contributing to a large profile size.
        
        For example:

        - In Linux™ you can use the following command to find files larger than 10 MB: `> find . -type f -size +10000k -exec ls -lh {} \; | awk '{ print $9 ": " $5 }'`.

        - In Windows™, you can run the following search in Windows™ Explorer: `*.* size:> 10MB`

    7. Set the file descriptor limit to at least 20480.
    
        `ulimit -n 20480`
    
    8. Set the stack limit to at least 65536.
    
        `ulimit -s 65536`
    
    9. AIX® only: Use the following steps to increase the maximum length of the command line with environment variables:
        
        1. Run the following command to query the system attributes:

            `lsattr -EH -l sys0 | grep ncargs`

            The command returns a value similar to the following example:
                
            `ncargs  256  ARG/ENV list size in 4K byte blocks  True`

        2. If the ncargs value is less than 512, run the following command to increase the value:
            
            `chdev -l sys0 -a ncargs=1024`

    10. Stop the Deployment Manager server before you run the **WASPreUpgrade** command.
    
    11. **For a local migration only**: Run the **WASPreUpgrade** command from the v90_was_root_path/bin directory. **For a remote migration only**: Run the **WASPreUpgrade** command from the working_directory/bin directory that contains the remote migration package copied earlier. **For a remote migration only**: Add the extra parameter **-machineChange true**.
        
        - AIX®: `./WASPreUpgrade.sh temp_dir v85_was_root_path -javaoption -Xmx2048m -oldProfile v85_dmgr_profile_name -username was_admin_user -password was_admin_user_pswrd`
        
        - Linux™: `./WASPreUpgrade.sh temp_dir v85_was_root_path -javaoption -Xmx2048m -oldProfile v85_dmgr_profile_name -username was_admin_user -password was_admin_user_pswrd`
        
        - Windows™: `WASPreUpgrade.bat temp_dir v85_was_root_path -javaoption -Xmx2048m -oldProfile v85_dmgr_profile_name -username was_admin_user -password was_admin_user_pswrd`
        
        Where the following values are set:

        `temp_dir`
                    Temporary directory where the backup is stored.
        
        !!! note
            For Windows™, Windows™, the temp_dir cannot have spaces in the path.
        
        For example, your command on might look like: `./WASPreUpgrade.sh /opt/HCL/wasMigrateBackup /opt/HCL/WebSphere/AppServer -javaoption -Xmx2048m -oldProfile v85_dmgr_profile_name -username was_admin_user -password was_admin_user_pswrd`

    11. **For a remote migration only**: Compress and copy the backup created by the WASPreUpgrade command from the DMGR source server to the DMGR target server. For a local migration only: Use the backup image made with WasPreUpgrade on the local server. From the v90_was_root_path/bin directory, run the WASPostUpgrade command to migrate the backed-up source profile into the new profile:
    
        !!! note 
            If security is enabled on your profile, add the `-username was_userid -password was_userid_password` parameters to your **WASPostUpgrade** task.
        
        - AIX®: `./WASPostUpgrade.sh temp_dir -profileName v90_dmgr_profile_name -oldProfile v85_dmgr_profile_name -javaoption -Xmx2048m`
            
        - Linux™: `./WASPostUpgrade.sh temp_dir -profileName v90_dmgr_profile_name -oldProfile v85_dmgr_profile_name -javaoption -Xmx2048m`
            
        - Windows™: `WASPostUpgrade.bat temp_dir -profileName v90_dmgr_profile_name -oldProfile v85_dmgr_profile_name -javaoption -Xmx2048m`

        For example, your command might look like: `./WASPostUpgrade.sh /opt/HCL/wasMigratedBackup -profileName v90_dmgr_profile_name -oldProfile v85_dmgr_profile_name -javaoption -Xmx2048m`

        If the **WASPreUpgrade** was successful, but the **WASPostUpgrade** command results in an error, complete the following steps:
        
        1. Verify that only one profile is in the backup profile.
            
        2. Consider deleting Search Collection to decrease the data size.
            
        3. Move JCR Data during migration.

            !!! note
                The Java™ Content Repository might store index-related files in sub-directories in the WebSphere® profile, such as jcr, that can use much space. This data is not altered by the migration process, so it can be temporarily moved out of the old profile or the backup directory in preparation for migration. Move the directory into the new profile after migration completes.
            
        4. Ensure that no paths contain the **":"** character. This character might cause problems for **WASPreUpgrade** and **WASPostUpgrade** commands.
            
        !!! important
            After you run **WASPostUpgrade** on the Deployment Manager or HCL Digital Experience node, log in to the WebSphere® Integrated Solutions Console and verify that AutoSync is off.

    12. Open a command prompt and change to the **v90_dmgr_profile_path/bin/** directory.

    13. Run the following command to start the Deployment Manager:

        - AIX®: `./startManager.sh`
        
        - Linux™: `./startManager.sh`
        
        - Windows™: `startManager.bat`

3. Complete the following steps for each HCL Digital Experience node you have in your cluster.

    1. On the WebSphere® Application Server 9.0 target migration server installation, create a new base profile by using the **manageprofiles** command-line option. **For a remote migration only**: First delete the existing wp_profile that was created during the full installation of HCL Digital Experience 9.0 on the target migration server.
        
        - AIX®: `./manageprofiles.sh -create -defaultPorts -enableAdminSecurity false -profileName v90_wp_profile_name -profilePath v90_wp_profile_path -templatePath v90_was_root_path/profileTemplates/managed -nodeName source_node_name -cellName source_cell_name -hostName host_name -isDefault -omitAction samplesInstallAndConfig default AppDeployAndConfig`
        
        - Linux™: `./manageprofiles.sh -create -defaultPorts -enableAdminSecurity false -profileName v90_wp_profile_name -profilePath v90_wp_profile_path -templatePath v90_was_root_path/profileTemplates/managed -nodeName source_node_name -cellName source_cell_name -hostName host_name -isDefault -omitAction samplesInstallAndConfig default AppDeployAndConfig`
        
        - Windows™: `manageprofiles.bat -create -defaultPorts -enableAdminSecurity false -profileName v90_wp_profile_name -profilePath v90_wp_profile_path -templatePath v90_was_root_path/profileTemplates/managed -nodeName source_node_name -cellName source_cell_name -hostName host_name -isDefault -omitAction samplesInstallAndConfig default AppDeployAndConfig`
        
        Where the following values are set:
        
        **source_node_name**
        
        The node name on the source installation.

        **source_cell_name**
        
        The cell name on the source installation.

        **host_name**
        
        The host name of the environment.

        !!! important
            - Use the exact same cell name and node name that you used for the HCL Digital Experience and WebSphere® Application Server v8.5.5 or higher installation.
            - If you are running this command on IBM® i, you must add the -serverName HCL Portal and HCL Web Content Manager parameter.
            
    2. Open a command prompt.

        !!! note 
            If you are instructed to open a properties file, they are ASCII files. Open them with the appropriate tool.
    
    3. **For a local migration only**: Run the following **install-wp-migration-files** command from the v85_wp_profile_path/ConfigEngine directory to install HCL Digital Experience-specific files into the WebSphere® Application Server 9.0 installation.
        
        !!! note 
            Ensure that the file folder that contains the source binary files is read/write enabled. If the folder is read only, the **install-wp-migration-files** command fails.
        
        - AIX®: `./ConfigEngine.sh install-wp-migration-files -DNewWasLocation=v90_was_root_path -DWasPassword=password`
        
        - Linux™: `./ConfigEngine.sh install-wp-migration-files -DNewWasLocation=v90_was_root_path -DWasPassword=password`
        
        - Windows™: `ConfigEngine.bat install-wp-migration-files -DNewWasLocation=v90_was_root_path -DWasPassword=password`
        
        !!! note
            - If this step fails, **ConfigEngine** might not be working. To fix this error, restore the ConfigEngine.migration.bak file that was created in the ConfigEngine root directory. Change ConfigEngine.migration.bak to ConfigEngine.sh or ConfigEngine.bat, depending on your operating system.
            
            - If ConfigEngine.migration.bak was not created, the failure occurred before the file was changed. Therefore, **ConfigEngine** should work.

    4. **For a local migration only**: Copy the jndi.properties file from the v85_was_root_path/properties directory to the v90_was_root_path/properties directory specified by the **DNewWasLocation** command.
    
    5. Ensure OSGi cache was cleared from the new WebSphere® Application Server 9.0 installation by running the following command from the v90_wp_profile_path/bin directory:
        
        - AIX®: `./osgiCfgInit.sh -all`
        
        - Linux™: `./osgiCfgInit.sh -all`
        
        - Windows™: `osgiCfgInit -all`
    
    6. For a remote migration only: On the HCL Digital Experience 9.0 server, run the following command to create the PORTAL_V8.5.0.0_WAS_V90_OS.arch_RemoteMigrSupport.jar file. For example, the file might be PORTAL_V8.5.0.0_WAS_V90_windows.x86_RemoteMigrSupport.jar.
        
        - AIX®: `cd PortalServer_root/bin ./genRemMigPkg.sh remote_zip_dir`
        
        - Linux™: `cd PortalServer_root/bin ./genRemMigPkg.sh remote_zip_dir`
        
        - Windows™: `cd PortalServer_root\bin genRemMigPkg.bat remote_zip_dir`
    
        Where `remote_zip_dir` is an existing directory that is used to contain the generated file. Make sure that you specify the full path to the directory. Copy thePORTAL_V8.5.0.0_WAS_V90_OS.arch_RemoteMigrSupport.jar file from the HCL Digital Experience 9.0 target server to the HCL Digital Experience 9.0 source server. Extract the JAR file into a working directory. For example, extract PORTAL_V8.5.0.0_WAS_V90_linux.amd64_RemoteMigrSupport.jar.

    7. Complete the following steps to prepare the HCL Digital Experience profile for the **WASPreUpgrade** command.
        
        1. Uninstall any unneeded applications.
        
        2. Delete backup and uninstalled applications.
        
        3. Delete any old large log files or temp files that are no longer needed. If necessary, stop the HCL Portal server. Common temp file locations are:

            - v85_wp_profile_path/temp/

            - v85_wp_profile_path/wstemp/

            - v85_wp_profile_path/config/temp/

    8. The **WASPreUpgrade** and **WASPostUpgrade** commands that are used in HCL Digital Experience migration use much memory and might result in an OutOfMemoryException. To prevent this issue from happening, complete the following steps:
        
        1. Ensure that you are using a large enough heap size. Generally, using 2 GB heap size is sufficient, specified by using the `-javaoption -Xmx2048m` variable when you run **WASPreUpgrade** or **WASPostUpgrade** commands. Increase the Heap Size by increasing the `-Xmx2048m` variable. A possible value might be `-Xmx4096m`.
        
        2. Ensure that the size of the Deployment Manager profile was minimized by deleting unnecessary files. In most cases, the Deployment Manager profile (Dmgr01) should not be larger than 2 GB. It is a good idea to do a search for files larger than 10 MB to find any possible files that can be contributing to a large profile size.
        
            For example:

            - In Linux you can use the following command to find files larger than 10 MB: `> find . -type f -size +10000k -exec ls -lh {} \; | awk '{ print $9 ": " $5 }'.`
                
            - In Windows, you can run the following search in Windows Explorer: `*.* size:> 10MB`
    
    9. Set the file descriptor limit to at least 20480.
        
        `ulimit -n 20480`

    10. Set the stack limit to at least 65536.
        
        `ulimit -s 65536`

    11. AIX® only: Use the following steps to increase the maximum length of the command line with environment variables:

        1. Run the following command to query the system attributes:
            
            `lsattr -EH -l sys0 | grep ncargs`

            The command returns a value similar to the following example:
            
            `ncargs  256  ARG/ENV list size in 4K byte blocks  True`

        2. If the **ncargs** value is less than 512, run the following command to increase the value:
            
            `chdev -l sys0 -a ncargs=1024` 
        
    12. Stop the HCL Portal server before you run the **WASPreUpgrade** command.
    
    13. **For a local migration only**: Run the WASPreUpgrade command from the v90_was_root_path/bin directory. **For a remote migration only**: Run the WASPreUpgrade command from the working_directory/bin directory that contains the remote migration package that is copied earlier. **For a remote migration only**: Add the extra parameter **-machineChange true**.
        
        - AIX®: `./WASPreUpgrade.sh temp_dir v85_was_root_path -javaoption -Xmx2048m -oldProfile v85_wp_profile_name -username was_admin_user -password was_admin_user_pswrd`
        
        - Linux™: `./WASPreUpgrade.sh temp_dir v85_was_root_path -javaoption -Xmx2048m -oldProfile v85_wp_profile_name -username was_admin_user -password was_admin_user_pswrd`
        
        - Windows™: `WASPreUpgrade.bat temp_dir v85_was_root_path -javaoption -Xmx2048m -oldProfile v85_wp_profile_name -username was_admin_user -password was_admin_user_pswrd`
    
        Where the following values are set:
            
        **temp_dir**

        Temporary directory where the backup is stored.
            
        !!! note
            For Windows™, Windows™, the temp_dir cannot have spaces in the path.
        
        For example, your command on might look like: `./WASPreUpgrade.sh /opt/HCL/wasMigrateBackup /opt/HCL/WebSphere/AppServer -javaoption -Xmx2048m -oldProfile v85_wp_profile_name -username was_admin_user -password was_admin_user_pswrd`

    14. **For a remote migration only**: Compress and copy the backup that is created by the WASPreUpgrade command from the source server to the remote target server. Extract the backup into a temporary directory such as temp_dir.

    15. **For a remote migration only**: Update the deployment manager settings.
        
        If the deployment manager host name or soap port changed, update the source deployment manager host name and soap port with the target deployment manager host name and soap port in the serverindex.xml and wsadmin.properties in the backup profile at 
        /tmp/wp_profile_bak/profiles/wp_profile/config/cells/CellName/nodes/dmgr/serverindex.xml and 
        /tmp/wp_profile_bak/profiles/wp_profile/properties/wsadmin.properties.

    16. **For a local migration only**: Run the **WASPostUpgrade** command from `the v90_was_root_path/bin` directory on the local server. **For a remote migration only**: Run the **WASPostUpgrade** command from the `v90_was_root_path/bin` directory on the remote target server. The WasPostUpgrade command migrates the backed-up source profile into the new profile.
        
        !!! note
            if security is enabled on your profile, add the -username was_userid -password was_userid_password parameters to your WASPostUpgrade task.

        - AIX®: `./WASPostUpgrade.sh temp_dir -profileName v90_wp_profile_name -oldProfile v85_wp_profile_name -javaoption -Xmx2048m`
        
        - Linux™: `./WASPostUpgrade.sh temp_dir -profileName v90_wp_profile_name -oldProfile v85_wp_profile_name -javaoption -Xmx2048m`
        
        - Windows™: `WASPostUpgrade.bat temp_dir -profileName v90_wp_profile_name -oldProfile v85_wp_profile_name -javaoption -Xmx2048m`
    
        For example, your command might look like: `./WASPostUpgrade.sh /opt/IBM/wasMigratedBackup -profileName v90_wp_profile_name -oldProfile v85_wp_profile_name -javaoption -Xmx2048m`

        If the **WASPreUpgrade** was successful, but the **WASPostUpgrade** command results in an error, complete the following steps:
        
        1. Check that only one profile exists in the backup profile.
                
        2. Consider deleting Search Collection to decrease the data size.
            
        3. Move JCR Data during migration.
            
            !!! note 
                The Java™ Content Repository might store index-related files in sub-directories in the WebSphere® profile, such as jcr, that can use much space. This data is not altered by the migration process, so it can be temporarily moved out of the old profile or the backup directory in preparation for migration. Move the directory into the new profile after migration completes.
            
        4. Ensure that no paths contain the : character. This character might cause problems for **WASPreUpgrade** and **WASPostUpgrade** commands.

            !!! important
                After you run **WASPostUpgrade** on the Deployment Manager or HCL Digital Experience node, log in to the WebSphere® Integrated Solutions Console and verify that AutoSync is on. Go to **System Administration > Nodes**. Confirm that the node that is being migrated is in sync. If it is not, issue a sync for that node and wait for it to complete.
    
    17. Open a command prompt and change to the v90_wp_profile_path/bin/ directory.

    18. Run the following command to start the node agent:
    
        - AIX®: `./startNode.sh`
        
        - Linux™: `./startNode.sh`
        
        - Windows™: `startNode.bat`
    
    19. Log in to the WebSphere® Integrated Solutions Console.
    
    20. Confirm that the node that is being migrated is in sync. If it is not, run a Full resynch for that node and wait for it to complete.

    21. (Optional): If the source and target profile names are not the same, in the target ConfigEngine path, run the following task **action-copy-ce-script-native-encoding**.

        !!! note
            - Perform this step only if the encodings change  (If on Windows™ for SOURCE and Linux™ for TARGET, WAS migrations allow you to switch. The "lift and shift" of the `wp_profile` may have some incorrect encodings in ConfigEngine.sh as a result).
            - This step only needs to be run with a remote migration. This step is not required in a local migration because the encodings absolutely do not change.
            - The target ConfigEngine path should be in the same path as the PortalServer and AppServer paths. Do not run this command from the ConfigEngine path that is located within the target profile.

        - AIX®: `./ConfigEngine.sh v90_wp_profile_name action-copy-ce-script-native-encoding`
        
        - Linux™: `./ConfigEngine.sh v90_wp_profile_name action-copy-ce-script-native-encoding`
        
        - Windows™: `ConfigEngine.bat v90_wp_profile_name action-copy-ce-script-native-encoding`
    
    22. Disable syndication and Portal search in your source environment.
    
    23. Create a copy of the HCL Digital Experience source database. Refer to your database documentation for instructions on how to create the backup.
    
    24. Modify the wkplc_dbdomain.properties file on the HCL Digital Experience 9.0 target server to point to the database copy.
    
    25. Modify the following properties files so the parameters point to the new HCL Digital Experience 9.0 directory locations:
        
        1. From the v90_wp_profile_path/ConfigEngine/properties directory:
            
            - Verify that the **DbLibrary** parameter in the wkplc_dbtype.properties file points to the correct JDBC driver class. The value is either .zip or .jar. For example: /derby/lib/derby.jar.
            - Verify that the **WpsInstallLocation** parameter in the wkplc.properties file points to the correct installation location of HCL Digital Experience.
            - Verify that the administrative passwords in the wkplc_dbdomain.properties file were not deleted by the fix pack.
            - Verify that the WasRemoteHostName, WasSoapPort, WpsHostName properties in the wkplc.properties are set to the correct values.
        
        2. **For a local migration only**: From the /PortalServer directory, modify the following parameters in the wps.properties file:
            
            - **WasRootDir**=v90_was_root_path/
            
            - **ProfileDirectory**=v90_wp_profile_path
        
        3. **For a remote migration only**: From the v90_wp_profile_path/ConfigEngine/properties directory, modify the following parameters in the wkplc.properties file:
            
            - **WasRemoteHostName**=host name of the Deployment Manager target server
            
            - **WpsHostName**=host name of the HCL Digital Experience target server
    
    26. Open a command prompt and change to the v90_wp_profile_path/ConfigEngine directory.
    
    27. Run the following command:
        
        - AIX®: `./ConfigEngine.sh validate-database -DWasPassword=password -DPortalAdminPwd=password`
        
        - Linux™: `./ConfigEngine.sh validate-database -DWasPassword=password -DPortalAdminPwd=password`
        
        - Windows™: `ConfigEngine.bat validate-database -DWasPassword=password -DPortalAdminPwd=password`
    
    28. Run the following command to recreate your data sources:
        
        - AIX®: `./ConfigEngine.sh connect-database -DWasPassword=password -DPortalAdminPwd=password`
        
        - Linux™: `./ConfigEngine.sh connect-database -DWasPassword=password -DPortalAdminPwd=password`
        
        - Windows™: `ConfigEngine.bat connect-database -DWasPassword=password -DPortalAdminPwd=password`
    
    29. Multiple clustered environments only: Run the following command on the second cluster (Cluster B) that you are migrating. Only run the task if each cluster's database domains use a different database user and password. To learn how to create the custom theme, go to the Roadmaps section and find Roadmap: Multiple clusters.
        
        - AIX®: `./ConfigEngine.sh create-alias-multiple-cluster -DauthDomainList=release,jcr,community,customization,feedback,likeminds -DWasPassword=password -DPortalAdminPwd=password`
        
        - Linux™: `./ConfigEngine.sh create-alias-multiple-cluster -DauthDomainList=release,jcr,community,customization,feedback,likeminds -DWasPassword=password -DPortalAdminPwd=password`
        
        - Windows™: `ConfigEngine.bat create-alias-multiple-cluster -DauthDomainList=release,jcr,community,customization,feedback,likeminds -DWasPassword=password -DPortalAdminPwd=password`
    
    30. Log in to the Deployment Manager/WebSphere Application Server Integrated Solution Console of your newly migrated environment.
    
    31. Navigate to **Environment > WebSphere Variables**.
    
    32. Inspect and validate the **WCM_HOST** and **WCM_PORT** values for each Portal server, to see if they are accurate for your newly migrated environment. Update the values if they are incorrect.
    
    33. Run the following **post-was-migration-update** command:
        
        - AIX®: `./ConfigEngine.sh post-was-migration-update -DWasPassword=password -DPortalAdminPwd=password -DoldProfileLocation=v85_wp_profile_path`
        
        - Linux™: `./ConfigEngine.sh post-was-migration-update -DWasPassword=password -DPortalAdminPwd=password -DoldProfileLocation=v85_wp_profile_path`
        
        - Windows™: `ConfigEngine.bat post-was-migration-update -DWasPassword=password -DPortalAdminPwd=password -DoldProfileLocation=v85_wp_profile_path`
        
        If the **post-was-migration-update** task fails, check the v90_wp_profile_path/logs/HCL Portal and HCL Web Content Manager/SystemOut.log file for errors. Search for error code CWPKI0428I. If you find the following message, complete the instructions that are provided in the log file to add the signer certificate. After the certificate is added, restart the node agent and run the **post-was-migration-update** task again.

4. **For a remote migration only**: Complete the following steps to update the ports for the deployment manager and on every node:
    
    You can still have your source servers running and broadcasting services within your network. After migration, the deployment manager name for example remains unchanged. To avoid conflicts with the new target environment server and services, you need to set unique ports. If you do not run this task, it is possible for the source and target environments to become corrupted.

    As a rule, increase the original port number by 100. You still need to make sure that the same ports are not used in the source environment.

    1. Log in to the WebSphere® Integrated Solutions Console.
    
    2. Click **System administration > Deployment manager > Ports**.
    
    3. Update the following ports for the node:
        
        - CELL_DISCOVERY_ADDRESS
        
        - DCS_UNICAST_ADDRESS
    
    4. Click **System administration > Node agents > node_agent > Ports**.
    
    5. Update the following ports for the node:
        
        - DCS_UNICAST_ADDRESS
        
        - NODE_DISCOVERY_ADDRESS
        
        - NODE_IPV6_MULTICAST_DISCOVERY_ADDRESS
        
        - NODE_MULTICAST_DISCOVERY_ADDRESS

    6. Click **Servers > Server types > WebSphere application servers > application_servers > Ports.**
        
        !!! note 
            The default value for the `application_servers` is HCL Portal and HCL Web Content Manager; however, you can have more servers that are defined. Update the ports on all application servers listed.
    
    7. Update the following port for the node:
        
        - DCS_UNICAST_ADDRESS
    
    8. Click **System administration > Nodes** to resynchronize the primary node.
    
    9. Select the check box for the primary node.
    
    10. Click **Full Resynchronize**.

        !!! note 
            For resynchronization of the primary node, the node agent on the primary node must be running.
    
    11. Wait 30 minutes before you continue on to the next step. It takes several minutes for the synchronization to complete. The WebSphere® Integrated Solutions Console might indicate that synchronization is complete. However, wait 30 minutes before you continue to the next step. If you begin the next step before the completion of the synchronization, applications might not be deployed properly to the nodes.
    
    12. Restart the deployment manager and all of the node agents after you update the ports for every node and complete resynchronization.
    
    13. Continue to run the deployment manager and the node agent of the node that is being updated. However, stop all other node agents before you continue to the next step.
        
        !!! important
        If you update the SOAP port of the target Deployment Manager, then you must update the **WPS_SOAP_PORT** value in the wpscript.sh file. The wpscript.sh file is in the [wp_profile_root/PortalServer/bin/](../../../guide_me/wpsdirstr.md) and [PortalServer_root/bin](../../../guide_me/wpsdirstr.md) directory for each node.
    
    Both the source and target environments can now run simultaneously.

5. Start the HCL Portal server.
    
    If the WebSphere® Application Server migration fails, review the logs to troubleshoot any migration errors. Then, rerun the **install-wp-migration-files** command.

6. Log in to the WebSphere® Integrated Solutions Console and turn on AutoSync.

7. After you migrate your profile to WebSphere® Application Server 9.0.0.2, the Configuration Wizard in the v85_AppServer root/profiles/cw_profile directory is still usable. If you want to have the wizard also running on WebSphere® Application Server 9.0.0.2, run the following command from the /ConfigEngine directory.

    - AIX®: `./ConfigEngine.sh create-config-wizard -DWizardUserid=userID -DWizardPassword=password -DWasPassword=password`
    
    - Linux™: `./ConfigEngine.sh create-config-wizard -DWizardUserid=userID -DWizardPassword=password -DWasPassword=password`
    
    - Windows™: `ConfigEngine.bat create-config-wizard -DWizardUserid=userID -DWizardPassword=password -DWasPassword=password`