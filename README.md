# Automated Execution Environment (EE) Build for Ansible Automation Platform (AAP)

This Ansible project (playbook) perform the following actions :
- Build a new Execution Environment (EE)
- Push the new EE to the AAP Private Automation Hub
- Create the new EE in the AAP Controller

This code is a slightly improved and self-sufficient version of the redhat-cop [ee_builder role](https://github.com/redhat-cop/ee_utilities/tree/main/roles/ee_builder)

## Requirements

The awx.awx and containers.podman collections MUST be installed in order for this playbook to work. You can install them using the following command :

```
ansible-galaxy collection install -r requirements.yml
```

## Usage

1. Download the code and then install ansible collections requirements if not already installed

```
git clone https://github.com/w4hf/automated-ee-build.git
cd automated-ee-build
mkdir collections
ansible-galaxy collection install -r requirements.yml -p collections/
```

2. Fill in your environment details in vars/ee.yml, all the variables are explained below.
 
3. OPTIONAL : If you have a custom Containerfile Jinja2 template put it in templates/Containerfile.j2 and don't forget to define any new custom variable.

4. Launch the ee-build.yml playbook 

```
ansible-playbook ee-build.yml
```

## Variables 

All the variables should be defined in the vars/ee.yml file
The file is preloaded with sample values that are only there for guidance purpose and should be replaced before usage.
3 Categories of variables :

### 1. Environment Variables

| Variable                  | Sample Value | Description                                                                 |
|---------------------------|--------------|-----------------------------------------------------------------------------|
| build_dir                 | /tmp         | All files will be generated in the build dir file                           |
| ee_name                   | ee-test      | Generated image name                                                        |
| ee_tag                    | 1.0          | Generated image tag                                                         |
| push_image_to_private_hub | True         | Set to true to push the newly generated image to the Private Automation Hub |
| delete_all_when_finished  | True         | Set to true to cleanup the build dir when finished                          |

### 2. Credentials Variables

#### Controller Credentials

The Controller credentials are used to create the Execution Environment in the Controller

| Variable                             | Sample Value                      | Description                                                                                        |
|--------------------------------------|-----------------------------------|----------------------------------------------------------------------------------------------------|
| controller_host                      | controller.internal.domain        | Hostname or IP of the AAP Controller node                                                          |
| controller_username                  | admin                             | Username used to login to AAP Controller                                                           |
| controller_password                  | SomeSecurePassword                | Password of the AAP Controller                                                                     |
| private_hub_credential_in_controller | Automation Hub Container Registry | Name of the credential of the Private Automation Hub in the Controller (type "Container Registry") |


#### Automation Hub Credentials

Automation Hub credentials are used to push newly created image to Automation Hub and to download/install Ansible collection

| Variable                   | Sample Value        | Description                                                   |
|----------------------------|---------------------|---------------------------------------------------------------|
| private_hub_host           | hub.internal.domain | Hostname or IP address of the Private Automation Hub          |
| private_hub_token          | SomeLongTokenString | Automation Hub Token used to pull Ansible collections         |
| private_hub_username       | admin               | Username to connect to Automation Hub                         |
| private_hub_password       | SomeSecurePassword  | Automation Hub password                                       |
| private_hub_validate_certs | False               | Set to false to ignore self signed Automation Hub certificate |

#### Builder Image Source Registry Credentials


| Variable                        | Sample Value                                                                   | Description                                                                                                 |
|---------------------------------|--------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|
| builder_image_registry_username | w4hf                                                                           | Builder image source registry login. Should be your Redhat account username if using registry.redhat.io     |
| builder_image_registry_password | SomeSecurePassword                                                             | Builder image source registry Password. Should be your Redhat account password if using registry.redhat.io  |
| builder_image                   | registry.redhat.io/ansible-automation-platform-22/ansible-builder-rhel8:latest | The builder image used to build the EE                                                                      |
| base_image                      | registry.redhat.io/ansible-automation-platform-22/ee-minimal-rhel8:latest      | The base image on top of which the EE will be built                                                         |

### 3. Image Variables

| Variable                    | Sample Value                                                                                                                                   | Description                                                                                                                           |
|-----------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| create_new_ee_in_controller | True                                                                                                                                           | Set to true to create the EE in the Controller                                                                                        |
| bindeps                     | - "vi [platform:rpm]" - "curl [platform:rpm]"                                                                                                  | List of the binaries that will be installed on the generated image                                                                    |
| collections                 | - name: ansible.posix   version: 1.4.0 - name: community.general   version: 5.6.0 - name: redhat.satellite - name: redhat.satellite_operations | Ansible collections to installed in the generated image                                                                               |
| python_packages             | - netaddr - boto                                                                                                                               | Python packages to be installed in the generated image                                                                                |
| use_custom_container_file   | True                                                                                                                                           | Set to True to use the custom Containerfile templates/Containerfile.j2 Probably should be set to True in most cases                   |
| use_proxy                   | True                                                                                                                                           | Set to True to use a proxy to reach internet. Internet is needed to install binaries, python packages and Ansible Galaxy collections. |
| http_proxy                  | http://proxy_hostname:proxy_port                                                                                                               | HTTP requests proxy                                                                                                                   |
| https_proxy                 | http://proxy_hostname:proxy_port                                                                                                               | HTTPS requests proxy                                                                                                                  |
| no_proxy                    | .internal.domain                                                                                                                               | Avoid using proxy for these domain names. Should be                                                                                   |
| pull_base_image             | True                                                                                                                                           | Set this to False if Base image is already locally present (to speed things up a bit)                                                 |



