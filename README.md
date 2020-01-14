# IMS provisioning with Ansible

This repo provides a set of sample playbooks to provision and deprovision an IMS TM/DB system using Ansible. 

Additional documentation for the Ansible roles used in this playbook can be found in `/roles/<role_name>`

## Document Syntax

This documentation will use **host** to refer to the system used to start Ansible, **target** will refer to the system to be configured.

## Requirements

* Python 2.7.13 or higher must be installed on the target z/OS system. For information consult the "General Help" section. 
* Python 2.7-3.7 must be installed on the host (system starting Ansible). Python 3 or higher is recommended.
  * To install Python 3.6 on a z/OS system, please consult the [Install Python 3.6](#install-python-36) section.
* Ansible packages must be installed on the host (using *pip*, *aptitude*, *yum* or other package install methods, refer to the [install documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) for more information).
* SSH must be enabled on the target z/OS system.
* Z Open Automation Utilities must be installed. For more information consult the [install ZOAU](#install-zoau-z-open-automation-utilities) section.
* __Optional:__ For development of Java applications accessing IMS databases or transactions, the IMS Universal JDBC drivers will be required. 
    * Download from [Universal JDBC driver](https://www-01.ibm.com/marketing/iwm/iwm/web/reg/download.do?source=swg-imsUD&S_PKG=dlUD&lang=en_US&cp=UTF-8) or obtain via the IMS JAVA ON DEMAND FEATURES FMID JMK1406

## Initial setup

A few settings may need to be changed to ensure compatability with your z/OS target.

For more information on python configuration requirements on z/OS, refer to [Ansible FAQ: Running on z/OS](https://docs.ansible.com/ansible/latest/reference_appendices/faq.html).

1. Change the environment variables, provided as key value pairs in the `system_environment` variable, in the file `host_vars/native.yml` to match your target's python installation.
    * If you are using Rocket Software's Python Installation, you will likely need all of the provided variables. Change the paths provided to match those of your installation. For example, my python installation is at `/usr/lpp/rsusr/python27/python-2017-04-12-py27/python27/bin` and my environment variables are as follows:

    ```yaml
    environment:
        # environment variables for python
        MANPATH: /usr/man/%L
        FFI_LIB: /usr/lpp/rsusr/python27/python-2017-04-12-py27/python27/lib/ffi
        RELEASE_NAME: python-2017-04-12
        CURL_CA_BUNDLE: /usr/lpp/rsusr/python27/python-2017-04-12-py27/python27/etc/ssl/cacert.pem
        RELEASE_DIR: /usr/lpp/rsusr/python27/python-2017-04-12-py27
        RELEASE_TYPE: py27
        LIBPATH: /usr/lpp/rsusr/python27/python-2017-04-12-py27/python27/lib:/lib:/usr/lib:.
        NLSPATH: /usr/lib/nls/msg/%L/%N
        PATH: /usr/lpp/rsusr/python27/python-2017-04-12-py27/python27/bin:/bin:.
        MAIL: /usr/mail/OMVSADM
        _: /bin/env
        TERMINFO: /usr/lpp/rsusr/python27/python-2017-04-12-py27/python27/share/terminfo
        PYTHON_ENV: python27
        PKG_CONFIG_PATH: /usr/lpp/rsusr/python27/python-2017-04-12-py27/python27/lib/pkgconfig:/usr/lpp/rsusr/python27/python-2017-04-12-py27/python27/share/pkgconfig
        PYTHON_HOME: /usr/lpp/rsusr/python27/python-2017-04-12-py27/python27
    ```  

2. Update the provisioning configuration
   * `/host_vars/native.yml` holds the variables to be used for provisioning. Change values as necessary to meet your configuration. If you have a working z/OSMF workflow, most variables should translate directly. 
3. Update the hosts file
   * `inventory` contains the information needed to connect to our target. We must specify the following information about our target system:
     * ansible_host: either an IP or URL to the target system.
     * ansible_user: the username used to login with SSH.
     * ansible_python_interpreter: the path on the target to the python interpreter.
   * An example is below, where `z-system` will be the name used to reference our target:
  
    ```yaml
    [z-system]
        host1 ansible_host=ecXXXXXa.vmec.svl.ibm.com ansible_user=omvsadm ansible_python_interpreter="/usr/lpp/rsusr/python27/python-2017-04-12-py27/python27/bin/python"
    ```

    * Multiple systems can be specified in the hosts file, Ansible can run across a cluster.

4. [Bzip2 from Rocket Software](https://www.rocketsoftware.com/zos-open-source) is used to decompress files sent to the z/OS node. A copy of bzip2 should be placed in `roles/install-bzip2/files/` or already be installed in the path set for the variable `uss_utilities_path`.

## Run the playbook

To run the Ansible playbook and begin provisioning, type the following from the root of this repository:

`ansible-playbook -i native-inventory provision-ims-tmdb.yml`

This command assumes you have a public SSH key on the system we will be using as our Ansible host (specified in the host file). To add your publickey, use the `ssh-copy-id` command. 

`ssh-copy-id -i ~/.ssh/mykey.pub usrt001@<hostsystemname>.com`

Alternatively, the command `ansible-playbook -i native-inventory provision-ims-tmdb.yml --ask-pass` can be used without needing a publickey on the host system. On some systems (mac) this may significantly impact performance.

This `ansible-playbook` call tells Ansible to run the `provision-ims-tmdb.yml` playbook, specifying the hosts file with `-i`. The `--ask-pass` flag tells Ansible to ask for the SSH password during runtime instead of failing the playbook. This flag is useful for testing, but is being deprecated. The `ansible_password` can be set in the `hosts` file, or stored in an encrypted file and used with variable substitution.

For debugging output, use `-v`, `-vv`, `-vvv`, or `-vvvv` for increasing levels of verbosity.

### Inventories 

This repository designed to support multiple host types (Native vs z/VM for example).
Each host type has a separate inventory file and variable values.
Examples for both Native (_native-inventory_)and z/VM (_zvm-inventory_) are provided as examples for multiple environment support.
Each host type has a file in `host_vars` that is loaded at runtime automatically, and a folder named `vars_<hostname>` to hold variables which are loaded dynamically during playbook execution.

## Additional resources

[Official Ansible documentation](https://docs.ansible.com/ansible/latest/index.html)

## Known Issues

### Steps are not executing as fast when running on *Mac OSX*

* There is a known issue on Mac computers when using the `--ask-pass` flag. This causes Ansible to use *paramiko* instead of *OpenSSH*. Use SSH keys as a workaround.

### *paramiko* throws **CryptographyDeprecationWarning** messages while running playbook

* These warnings are shown when Python cryptography library v2.4.2 is not installed.
  * Run `pip3 install cryptography==2.4.2`

## General help

### The installation path for Ansible differs depending on installation method and OS

__Mac__:

Recommended install method is: Pip

* To install: `pip install ansible`
* For python 3:  `pip3 install ansible`
* Installation path can be found with `pip/pip3 show ansible`

__Linux__:

Install using package manager for distribution such as `pacman`, `yum` or `aptitude`.

Package manager's generally place Ansible in a standard install path.

### Running playbooks

On mac, using SSH keys is required to get full performance and simultaneous connection support.
Using –ask-pass will cause paramiko (inferior python based SSH) to be used.

### Using SSH keys

To generate an SSH key: `ssh-keygen -t rsa -b 4096`

Copy your public key to the target node: `ssh-copy-id -I ~/.ssh/id_rsa.pub target_username@target_address`

## z/OS specific Ansible help

Rocket Python versions earlier than 2.7.13/3.6.1 will not work with Ansible.
Recent versions represent strings in ASCII.

Python on z/OS requires some environment variables to be set in order to be used.
These need to be provided when running an Ansible playbook with z/OS as the target node.

Environment variables can be set at the task or play level, it is recommended to set these environment variables at the play level so they will be applied to all tasks in the play.

### SSH pipelining

According to the docs, ANSIBLE_SSH_PIPELINING should be set to true when z/OS is the target node.
In addition, enabling pipelining significantly increases Ansible performance.

### EBCDIC vs ASCII Encoding

z/OS systems generally use EBCDIC encoding, this can lead to issues when copying files between Unix and z/OS.

There are two transport methods that can be used to send files with Ansible: SCP and SFTP. Each of these behave differently on z/OS.

* SCP
  * Treats files as text
  * Performs ASCII/EBCDIC conversion on files automatically
* SFTP
  * Files transferred between EBCDIC and ASCII platforms are not converted

Ansible has some variables that can be set to force a particular communication method to be used.
  
* _DEFAULT_SCP_ID_SSH_ – set to True to force SCP, if false will use SFTP
* _DEFAULT_SSH_TRANSFER_METHOD_ – unused according to docs

These can be set as environment variables or in `ansible.cfg`

To convert from ASCII to EBCDIC, the following command can be used in z/OS USS

`iconv -f ISO8859-1 -t IBM-1047 ASCII_file_name`

### Install Python 3.6

To install Python 3.6, it will need to be downloaded from Rocket Software. An account on the Rocket Customer Portal (https://my.rocketsoftware.com) is required. 
1) Click Downloads button at top left of the screen
3) Choose category z/OpenSource on left side panel
4) Scroll to Python
5) Download the binaries, install files, and the README.ZOS on an x86 machine. Transfer the tarball to the target z/OS system and unpack it according the instructions in the install files
6) Additional setup will need to be performed as described in the README.ZOS. The README.ZOS file is only a text file.


### Install ZOAU (Z Open Automation Utilities)
For instructions on installing ZOAU manually, consult the following link:
https://www.ibm.com/support/knowledgecenter/en/SSKFYE_1.0.0/install.html

__NOTE:__ Python 3.6 is required

The FMID for ZOAU is HAL5100. For the program directory and more information, consult this link. 
https://www.ibm.com/support/knowledgecenter/en/SSKFYE_1.0.0/program_directory_zoautil/hal5100.html

