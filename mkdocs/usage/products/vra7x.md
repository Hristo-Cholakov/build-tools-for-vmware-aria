---
title: vRealize Automation 7.x
---

# vRealize Automation 7.x Project

## About
vRA project is a filesystem representation of vRA content into human friendly YAML or JSON format. The project consist of content descriptor and content container.

- *Content Descriptor* defines what vRA content will be part of this project.
- *Content Container* holds the actual content representation.

## Prerequisites
- Install and Configure #TO DO: link to workstation setup

## Crate New vRA 7 Project
{{ general.bta_name }} provides ready to use vRA 7 project templates using *maven archetypes*.

To create a new vRA 7 project use the following command:
```Bash
args=(
    "-DinteractiveMode=false"
    "-DarchetypeGroupId=com.vmware.pscoe.vra.archetypes"
    "-DarchetypeArtifactId=package-vra-archetype"
    "-DarchetypeVersion={{ iac.latest_release }}"
    "-DgroupId={{ archetype.customer_project.group_id}}" # (1)!
    "-DartifactId={{ archetype.customer_project.artifact_id}}" # (2)!
)
mvn archetype:generate "${args[@]}"
```

1.  {{ archetype.customer_project.group_id_hint}}
2.  {{ archetype.customer_project.artifact_id_hint}}
   
The result of the command will produce the following file structure:
```
.
├── README.md
├── content.yaml
├── license_data
│   ├── _license
│   │   ├── template_header.txt.ftl
│   │   └── template_license.txt.ftl
│   └── licenses.properties
├── pom.xml
├── release.sh
└── src
    └── main
        └── resources
            └── README.md
```

Content Descriptor is implemented by content.yaml file with the following defaults:
```
---
# Example describing for export Composite blueprints by their names
#
# composite-blueprint:
#   - SQL 2016 Managed
#   - Kubernates 1.9.0

property-group:
property-definition:
software-component:
composite-blueprint:
xaas-blueprint:
xaas-resource-action:
xaas-resource-type:
xaas-resource-mapping:
workflow-subscription:
...
```
!!! note
    vRA Project supports only content types outlined into Content Descriptor.

To capture the state of your vRA environment simply fill in the names of the content objects and follow the [Pull](#pull) section.

<!-- Build Section -->
{% include-markdown "../../assets/docs/mvn/build-project.md" %}
The output of the command will result in **{{ archetype.customer_project.group_id}}.{{ archetype.customer_project.artifact_id}}-1.0.0-SNAPSHOT.vra** file generated in the target folder of the project.

## Environment Connection Parameters
There are two ways to pass the vRA connection parameters to maven pull/push command:
=== "Use Maven Profiles"
    Append the following profile section in your maven settings.xml file.

    ``` xml
    ...<!--# (1)! -->
    <profiles>
      ...
      <profile>
          <id>vra-env</id>
          <properties>
              <!--vRA Connection-->
              <vra.host>vra-fqdn</vra.host>
              <vra.port>443</vra.port>
              <vra.username>configurationadmin@vsphere.local</vra.username>
              <vra.password>*****</vra.password>
              <vra.tenant>vsphere.local</vra.tenant>
          </properties>
      </profile>
    </profiles>
    ```

    1.  {{ archetype.customer_project.maven_settings_location_hint}}

    Use the profile by passing it with:
    ``` bash
    mvn vra:pull -Pvra-env
    ```

=== "Directly Pass The Parameters"

    ``` sh
    mvn vra:pull -Dvra.host=vra-fqdn -Dvra.port=443 -Dvra.username=configurationadmin@vsphere.local -Dvra.password=***** -Dvra.tenant=vsphere.local
    ```



## Pull Content
When working on a vRA project, you mainly make changes on a live server using the vRA Console and then you need to capture those changes in the maven project on your filesystem.

To support this use case, the toolchain comes with a custom goal "vra:pull". The following command will "pull" the content outlined into *Content Descriptor* file to the current project from a specified server and expand its content in the local filesystem overriding any local content:
```bash
mvn vra:pull -Dvra.host=10.29.26.18 -Dvra.port=443 -Dvra.username=configurationadmin@vsphere.local -Dvra.password=*** -Dvra.tenant=vsphere.local
```
A better approach is to have the different vRO/vRA development environments specified as profiles in the local
settings.xml file by adding the following snippet under "profiles":
```xml
<servers>
    <server>
        <username>configurationadmin@vsphere.local</username>
        <password>{native+maven+encrypted+pass}</password>
        <id>corp-dev-vra</id>
    </server>
</servers>
....
<profile>
    <id>corp-dev</id>
    <properties>
        <!--vRA Connection-->
        <vra.host>10.29.26.18</vra.host>
        <vra.port>443</vra.port>
        <vra.tenant>vsphere.local</vra.tenant>
        <vra.serverId>corp-dev-vra</vra.serverId>
    </properties>
</profile>
```
Then, you can sync content back to your local sources by simply activating the profile:
```bash
mvn vra:pull -Pcorp-env
```

> Note that ```vra:pull``` will fail if the content.yaml is empty or it cannot find some of the described content 
on the target vRA server.

## Push Content
To deploy the code developed in the local project or checked out from source control to a live server, you can use
the ```vrealize:push``` command:
```bash
mvn package vrealize:push -Pcorp-env
```
This will build the package and deploy it to the environment described in the ```corp-env``` profile. There are a few
additional options.

## Include Dependencies
By default, the ```vrealize:push``` goal will deploy all dependencies of the current project to the target 
environment. You can control that by the ```-DincludeDependencies``` flag. The value is ```true``` by default, so you
skip the dependencies by executing the following:
```bash
mvn package vrealize:push -Pcorp-env -DincludeDependencies=false
```

Note that dependencies will not be deployed if the server has a newer version of the same package deployed. For example,
if the current project depends on ```com.vmware.pscoe.example-2.4.0``` and on the server there is ```com.vmware.pscoe.example-2.4.2```,
the package will not be downgraded. You can force that by adding the ````-Dvra.importOldVersions``` flag:
```bash
mvn package vrealize:push -Pcorp-env -Dvra.importOldVersions
```
The command above will forcefully deploy the exact versions of the dependent packages, downgrading anything it finds on the server.

### Ignore Certificate Errors (Not recommended)
> This section describes how to bypass a security feature in development/testing environment. **Do not use those flags when targeting production servers.** Instead, make sure the certificates have the correct CN, use FQDN to access the servers and add the certificates to Java's key store (i.e. cacerts).

You can ignore certificate errors, i.e. the certificate is not trusted, by adding the flag ```-Dvrealize.ssl.ignore.certificate```:
```bash
mvn package vrealize:push -Pcorp-env -Dvrealize.ssl.ignore.certificate
```

You can ignore certificate hostname error, i.e. the CN does not match the actual hostname, by adding the flag ```-Dvrealize.ssl.ignore.certificate```:
```bash
mvn package vrealize:push -Pcorp-env -Dvrealize.ssl.ignore.hostname
```

You can also combine the two options above.

The other option is to set the flags in your Maven's settings.xml file for a specific **development** environment.
```xml
<profile>
    <id>corp-dev</id>
    <properties>
        <!--vRO Connection-->
        <vro.host>10.29.26.18</vro.host>
        <vro.port>8281</vro.port>
        <vro.username>administrator@vsphere.local</vro.username>
        <vro.password>***</vro.password>
        <!--The id of maven element containing the encrypted password and username-->
        <vro.serverId>${serverid}</vro.serverId>
        <vro.auth>vra</vro.auth>
        <vro.tenant>vsphere.local</vro.tenant>
        <vro.authHost>{auth_host}</vro.authHost>
        <vro.authPort>{auth_port}</vro.authPort> 
        <vro.refresh.token>{refresh_token}</vro.refresh.token> 
        <vro.proxy>http://proxy.host:80</vro.proxy>
        <vrealize.ssl.ignore.hostname>true</vrealize.ssl.ignore.hostname>
        <vrealize.ssl.ignore.certificate>true</vrealize.ssl.ignore.certificate>
    </properties>
</profile>
```

## Bundling
To produce a bundle.zip containing the package and all its dependencies, use:
```
$ mvn clean deploy -Pbundle
```
Refer to [Build Tools for VMware Aria](setup-workstation-maven.md)/Bundling for more information.

## Clean Up
To clean up a version of vRA package from the server use:
- Clean up only curent package version from the server
    ```
    mvn vrealize:clean -DcleanUpLastVersion=true -DcleanUpOldVersions=false -DincludeDependencies=false
    ```
- Clean up curent package version from the server and its dependencies. This is a force removal operation.
    ```
    mvn vrealize:clean -DcleanUpLastVersion=true -DcleanUpOldVersions=false -DincludeDependencies=true
    ```
- Clean up old package versions and the old vertions of package dependencies.
    ```
    mvn vrealize:clean -DcleanUpLastVersion=false -DcleanUpOldVersions=true -DincludeDependencies=true
    ```

## Troubleshooting
* If Maven error does not contain enough information rerun it with *-X* debug flag.
```Bash
mvn -X <rest of the command>
```
* Sometimes Maven might cache old artifats. Force fetching new artifacts with *-U*. Alternativelly remove *<home>/.m2/repository* folder.
```Bash
mvn -U <rest of the command>
```
