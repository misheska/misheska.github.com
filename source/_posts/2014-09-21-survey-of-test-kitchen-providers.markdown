---
layout: post
title: "Survey of Test Kitchen providers"
date: 2014-09-21 20:52
comments: true
categories: [Chef, Test Kitchen]
---

* list element with functor item
{:toc}

# Introduction

Test Kitchen supports a wide variety of different providers via Test Kitchen drivers besides the default `kitchen-vagrant` driver.  In this post, we'll cover several popular alternatives.

Test Kitchen drivers are gem libraries available for download from http://rubygems.org .  Use the `kitchen driver discover` command to list all the Test Kitchen gems currently available.  Here is a list of all the Test Kitchen drivers as of this writing:

    $ kitchen driver discover
        Gem Name                          Latest Stable Release
        kitchen-all                       0.2.0
        kitchen-ansible                   0.0.1
        kitchen-azure                     0.1.0
        kitchen-bluebox                   0.6.2
        kitchen-cabinet                   3.0.0
        kitchen-cloudstack                0.10.0
        kitchen-digital_ocean             0.3.0
        kitchen-digitalocean              0.8.0
        kitchen-docker                    1.5.0
        kitchen-docker-api                0.4.0
        kitchen-driver-vagrant_provision  1.0.0
        kitchen-ec2                       0.8.0
        kitchen-fifo                      0.1.0
        kitchen-fog                       0.7.3
        kitchen-gce                       0.2.0
        kitchen-goiardi                   0.1.1
        kitchen-inspector                 1.3.0
        kitchen-joyent                    0.1.1
        kitchen-libvirtlxc                0.4.0
        kitchen-local                     0.0.1
        kitchen-lxc                       0.0.1
        kitchen-openstack                 1.6.0
        kitchen-puppet                    0.0.13
        kitchen-rackspace                 0.12.0
        kitchen-rightscale                0.1.0
        kitchen-salt                      0.0.19
        kitchen-scribe                    0.3.1
        kitchen-sharedtests               0.2.0
        kitchen-ssh                       0.0.4
        kitchen-sshgzip                   0.0.3
        kitchen-sync                      1.0.1
        kitchen-vagrant                   0.15.0
        kitchen-vagrant_sandbox           0.1.1
        kitchen-vagrant_winrm             0.1.1
        kitchen-zcloudjp                  0.5.0
        test-kitchen-provisioners         0.1


By default, Test Kitchen defaults to using the `kitchen-vagrant` driver.  When you run the `kitchen init` command to add Test Kitchen support to a project, you can add the `--driver=<gem_name>` option to have Test Kitchen generate configuration files using another driver of your choice.  For example, the following command would use the `kitchen-azure` driver:

    kitchen init --create-gemfile --driver=kitchen-azure

As shown in the following diagram the environments supported by Chef-releated drivers fall into four different categories: desktop virtual machines, public/private cloud providers, Linux containers and physical machines.  We'll cover representative examples from each category in this appendix.

{% img center /images/chapa01/tkdriver.004.png [Test Kitchen driver architecture] %}

# Desktop Virtualization

Test Kitchen uses the `kitchen-vagrant` driver to work with desktop virtualization providers, like VirtualBox, VMWare Fusion, VMWare Workstation and Hyper-V.  Since the `kitchen-vagrant` driver is just a shim on top of Vagrant for Test Kitchen, any provider that Vagrant supports should be supported by the `kitchen-vagrant` driver.

It is important to clarify that as of this writing, the `kitchen-vagrant` driver assumes that the virtualization provider is installed locally on the host machine.  As shown in the following diagram, using the `kitchen-vagrant` driver, Test Kitchen creates a sandbox environment virtual machine locally on your host:

1. Test Kitchen invokes the `kitchen-vagrant` driver to create a virtual machine instance.
2. In the case of the `kitchen-vagrant` driver, Vagrant itself contains all the logic to work with different types of virtualization software.  The `kitchen-vagrant` is just a small shim to allow Test Kitchen to use Vagrant to work with virtual machine instances.  In this example, Vagrant uses the VirtualBox API to spin up a virtual machine instance for our sandbox environment.
3. Once the sandbox environment is running, Test Kitchen links the instance for communication.

{% img center /images/chapa01/tkdriver.001.png [Sandbox environment creation with kitchen-vagrant] %}

Test Kitchen treats the data center versions of VMware, like vCenter/vSphere/ESXi as a cloud provider.  To Test Kitchen the data center editions are handled as if there were cloud instances, as vCenter/vSphere/ESXi merely a private cloud on a local LAN or corporate WAN instead of a public cloud over the Internet.  As of this writing, the `kitchen-openstack` and `kitchen-ssh` drivers support vSphere data center virtualization with Test Kitchen.

## kitchen-vagrant with VMware Fusion/VMware Workstation desktop virtualization

You can use VMware desktop virutalization with `kitchen-varant` instead of Oracle VM VirtualBox.  It requires the purchase of the Vagrant VMware plugin from https://www.vagrantup.com/vmware which, at the time of this writing, costs USD $79 per seat.  The VMware plugin works with VMware Workstation 9 and 10 on Windows/Linux and VMware Fusion 5, 6 and 7 on Mac OS X.

On Mac OS X/Linux, you may have multiple virtualization solutions installed alongside VMware.  On these platforms, you can use both VMware and VirtualBox baseboxes at the same time, for example, if you have enough system resources.  On Windows, you must make a choice, as only one virtualization solution can be installed at a time.

Once you have purchased the VMware plugin and received a license file, you can install the Vagrant plugin and license with the following:

For VMware Workstation (on Windows/Linux):

    $ vagrant plugin install vagrant-vmware-workstation
    $ vagrant plugin license vagrant-vmware-workstation license.lic

For VMware Fusion (on Mac OS X):

    $ vagrant plugin install vagrant-vmware-fusion
    $ vagrant plugin license vagrant-vmware-fusion license.lic

After you install the VMware plugin and license file and want to use VMware, you'll need to get VMware baseboxes.  Currently VirtualBox and VMware baseboxes are not interchangeable.

Once the VMware plugin and license has been installed, you'll need to change your `.kitchen.yml` files slightly for VMware.  You can specify the VMware provider name in the `platforms` section of your `.kitchen.yml` file.

Modify the `.kitchen.yml` file, adding a `provider:` line to the `platforms` `driver` section.  If you are using VMware Workstation, use the `vmware_workstation` provider name.  For VMware Fusion, the provider name should be `vmware_fusion`.  You'll also need to change the `box_url` line to point at a box file which has Vmware Tools installed, as box files are not guest tool agnostic.  For this book, box files have been provided for both VMware and VirtualBox via VagrantCloud, so you can use the same `box_url` line.

Synced folders work the same as with VirtualBox.  Just add a `synced_folders:` block to the `driver:` section with a list of folders to map between the guest and the host.  Each entry in the list contains an array with two parameters.  The first parameter is a path to the directory on the host machine.  If the path is relative, it is relative to the `.kitchen.yml` file.  The second parameter is an absolute path specifying where the folder is shared on the guest machine.  The `.kitchen.yml` examples that follow map the current working directory on the host to the directory `/vagrant` on the guest, like so:

    ...
           synced_folders:
           - [".", "/vagrant"]
    ...

VMware Workstation `.kitchen.yml` example:

{% codeblock vmware/workstation/.kitchen.yml lang:ruby %}
 ---
driver:
  name: vagrant

provisioner:
  name: chef_solo

platforms:
  - name: centos65
    driver:
      provider: vmware_workstation
      box: learningchef/centos65
      box_url: learningchef/centos65
      synced_folders:
        - [".", "/vagrant"]

suites:
  - name: default
    run_list:
    attributes:
{% endcodeblock %}

VMware Fusion `.kitchen.yml` example:

{% codeblock vmware/fusion/.kitchen.yml lang:ruby %}
 ---
driver:
  name: vagrant

provisioner:
  name: chef_solo

platforms:
  - name: centos65
    driver:
      provider: vmware_fusion
      box: learningchef/centos65
      box_url: learningchef/centos65
      synced_folders:
        - [".", "/vagrant"]

suites:
  - name: default
    run_list:
    attributes:
{% endcodeblock %}

Once you modify the `.kitchen.yml` file appropriately the `kitchen create`, `kitchen converge`, etc. commands will use VMware instead of VirtualBox:

    $ kitchen create default-centos65
    -----> Starting Kitchen (v1.2.2.dev)
    -----> Creating <default-centos65>...
           Bringing machine 'default' up with 'vmware_fusion' provider...
           ==> default: Cloning VMware VM: 'learningchef/centos65'. This can take some time...
           ==> default: Checking if box 'learningchef/centos65' is up to date...
           ==> default: Verifying vmnet devices are healthy...
           ==> default: Preparing network adapters...
           ==> default: Fixed port collision for 22 => 2222. Now on port 2200.
           ==> default: Starting the VMware VM...
           ==> default: Waiting for the VM to finish booting...
           ==> default: The machine is booted and ready!
           ==> default: Forwarding ports...
               default: -- 22 => 2200
           ==> default: Setting hostname...
           ==> default: Configuring network adapters within the VM...
           ==> default: Waiting for HGFS kernel module to load...
           ==> default: Enabling and configuring shared folders...
               default: -- /Users/misheska/github/learningchef/learningchef-code/chapa01/vmware/fusion: /vagrant
           ==> default: Machine not provisioning because `--no-provision` is specified.
           Vagrant instance <default-centos65> created.
           Finished creating <default-centos65> (0m39.42s).
    -----> Kitchen is finished. (0m39.66s)

# Test Kitchen Cloud Drivers

The following diagram shows how the Test Kitchen cloud drivers create a sandbox environment.  The main difference between using a cloud provider and desktop virtualization is that the sandbox environment lives remotely on another machine.  Test Kitchen communicates with the sandbox environment remotely over SSH, usually on the Internet.

1. Test Kitchen invokes the specified driver (like `kitchen-ec2`) to create an instance on the cloud provider.  Cloud provider drivers communicate with the cloud provider using the appropiate cloud API.  Normally this is an HTTP API. 
2. The cloud provider spins up an instance to serve as our sandbox environment.
3. Once the sandbox environment is running, Test Kitchen links the instance to your local development workstation for remote communication, usually over SSH.  All Test Kitchen commands work with the remote sandbox environment transparently.  As far as the user experience with Test Kitchen goes, it behaves as if it were a local desktop virtualization environment.

{% img center /images/chapa01/tkdriver.005.png [Test Kitchen driver architecture] %}

As of this writing, all of the Test Kitchen Cloud drivers do not support synchronized folders.  All `kitchen` commands automatically copy your project files to the sandbox environment, as Test Kitchen uses `scp` to transfer files from your host to the remote cloud instance.  For any other file sharing beyond what is supported by Test Kitchen, you'll need to use a Cloud Provider-specific mechanism, such as Amazon Elastic Block Store (EBS).

## DigitalOcean Cloud Provider (kitchen-digitalocean)

### kitchen-digitalocean Setup

Go to https://cloud.digitalocean.com/api_access to get your Client ID and API Key, as shown in the following diagram.  As of this writing, the `kitchen-digitalocean` provider still uses the 1.x DigitalOcean API, not the v2.0 API.  So you must use a v1.0 Client ID/API pair to work with the provider, not a v2.0 Personal Access Token. Record both the Client ID and API key values.  Click on the _Generate New Key_ button if your API Key is hidden or unavailable.

{% img center /images/chapa01/digitalocean_api_access.png [DigitalOcean API Access] %}

Collect SSH public keys from the computers which need access to your sandbox instances.  Visit https://cloud.digitalocean.com/ssh_keys and add the SSH keys.  Once you've added the SSH key(s), visit the URL in the following form to get your SSH Key IDs, replacing _<your_client_id>_ and _<your_api_key>_ with your Client API and API Key, respectively.

    http://api.digitalocean.com/ssh_keys/?client_id=<your_client_id>&api_key=<your_api_key>

The following screenshot shows an example of the output from the site.  Record the SSH Key ID fields.

{% img center /images/chapa01/digitalocean_ssh_key_ids.png [Digital Ocean SSH Key IDs] %}

Run the following `kitchen init` command to add Test Kitchen support to your project using the `kitchen-digitalocean` driver:

    $ kitchen init --driver=kitchen-digitalocean --create-gemfile
          create  .kitchen.yml
          create  test/integration/default
          create  Gemfile
          append  Gemfile
          append  Gemfile
    You must run `bundle install' to fetch any new gems.

Run `bundle install` to download and install any required gems.

### kitchen-digitalocean .kitchen.yml Example

Since the Client ID, API Key and SSH Key IDs contain sensitive information, it is recommended that you store them in environment variables instead of directly in your `.kitchen.yml` file.  This way, you can share your `.kitchen.yml` file with others and store it in source control.  You can use embedded Ruby templates in a `.kitchen.yml` to load values from the environment.  Here is an example `kitchen.yml` which spins up a CentOS 6.5 sandbox environment, loading the Client ID, API Key and SSH Key IDs from corresponding environment variables:

{% codeblock digitalocean/.kitchen.yml lang:ruby %}
 ---
driver:
  require_chef_omnibus: true
  name: digitalocean
  digitalocean_client_id: "<%= ENV['DIGITALOCEAN_CLIENT_ID']%>"
  digitalocean_api_key: "<%= ENV['DIGITALOCEAN_API_KEY']%>"
  ssh_key_ids: "<%= ENV['DIGITALOCEAN_SSH_KEY_IDS']%>"

provisioner:
  name: chef_solo

platforms:
  - name: centos65
    driver: digitalocean
      image_id: 3448641
      region_id: 4

suites:
  - name: default
    run_list:
    attributes:
{% endcodeblock %}

Before running any Test Kitchen commands, make sure you set the appropriate environment variables as shown below (with your own values):

Linux and Mac OS X:

    export DIGITALOCEAN_CLIENT_ID="abcdef01234567890abcdef0123456789"
    export DIGITALOCEAN_API_KEY="01234567890abcdef01234567890abcdef"
    export DIGITALOCEAN_SSH_KEY_IDS="12345, 67890"

Windows Command Prompt:

    set DIGITALOCEAN_CLIENT_ID=abcdef01234567890abcdef0123456789
    set DIGITALOCEAN_API_KEY=01234567890abcdef01234567890abcdef
    set DIGITALOCEAN_SSH_KEY_IDS=12345, 67890

Windows Powershell:

    $env:DIGITALOCEAN_CLIENT_ID="abcdef01234567890abcdef0123456789"
    $env:DIGITALOCEAN_API_KEY="01234567890abcdef01234567890abcdef"
    $env:DIGITALOCEAN_SSH_KEY_IDS="12345, 67890"

The output of `kitchen list` should resemble the following:

    $ kitchen list
    Instance           Driver        Provisioner  Last Action
    default-centos65  Digitalocean  ChefSolo     <Not Created>

Spin up the node with `kitchen create`:

    $ kitchen create default-centos65
    -----> Starting Kitchen (v1.2.2.dev)
    -----> Creating <default-centos65>...
           Digital Ocean instance 2016149 created.
    .........................................................................
           Waiting for 104.131.234.140:22...
           (ssh ready)

           Finished creating <default-centos65> (2m22.61s).
    -----> Kitchen is finished. (2m23.04s)

Install Chef Client with `kitchen setup`.  `kitchen destroy` will delete your Droplet on DigitalOcean.

Refer to the `kitchen-digitalocean` driver documentation on https://github.com/test-kitchen/kitchen-digitalocean for more information on additional `.kitchen.yml` settings.

## Amazon EC2 Cloud Provider (kitchen-ec2)

### kitchen-ec2 Setup

In order to use the `kitchen-ec2` driver, you'll need to create an Amazon Web Services access key, consisting of an _access key ID_ plus a _secret key_.  You can create a new _access key ID_ and _secret _key_ or retrieve an existing _access key ID_ on the AWS Identity and Access Management (IAM) page in the AWS Console.  Once you select a user, click on the _Manage Access Keys_ button as shown in the following:

{% img center /images/chapa01/ec2_manage_access_keys.png [AWS IAM Manage Access Keys] %}

In the Manage Access keys dialog, click on the _Create Access Key_ button to create a new _access key ID_ and _secret _key_ as shown in the following:

{% img center /images/chapa01/ec2_create_access_key.png [AWS Create Access Key] %}

AWS will create your access key.  You can click on _Show User Security Credentials_ to display the _Access Key ID_ and the _Secret Access Key_.  Make note of these as this is the last time they will be displayed.  You can also click on the _Download Credentials_ button to download the credentials as a `.csv` file as shown below:

{% img center /images/chapa01/ec2_download_security_credentials.png [Download Credentials] %}

Create a key pair to use when you launch instances.  Amazon EC2 supports a variety of ways to work with key pairs.  Refer to http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html for more information.

Make sure you set permissions on the key pair.  Otherwise `kitchen-ec2` will ignore the file.

    chmod 400 my-key-pair.pem

Run the following `kitchen init` command to add Test Kitchen support to your project using the `kitchen-ec2` driver:

    $ kitchen init --driver=kitchen-ec2 --create-gemfile
          create  .kitchen.yml
          create  test/integration/default
          create  Gemfile
          append  Gemfile
          append  Gemfile
    You must run `bundle install' to fetch any new gems.

Run `bundle install` to fetch any new gems.

### kitchen-ec2 .kitchen.yml Example

Since the Access Key ID, Secret Access Key and SSH Key ID contain sensitive information, it is recommended that you store these values in environment variables instead of directly in your `.kitchen.yml` file.  This way, you can share your `.kitchen.yml` file with others and store it in source control.  You can use embedded Ruby templates in a `.kitchen.yml` file to load values from the environment.  Here is an example `kitchen.yml` which spins up a CentOS 6.5 sandbox environment, loading the Access Key ID, Secret Acces Key and SSH Key ID from corresponding environment variables:

{% codeblock ec2/.kitchen.yml lang:ruby %}
 ---
driver:
  require_chef_omnibus: true
  name: ec2
  aws_access_key_id: "<%= ENV['AWS_ACCESS_KEY_ID']%>"
  aws_secret_access_key: "<%= ENV['AWS_SECRET_ACCESS_KEY']%>"
  aws_ssh_key_id: "<%= ENV['AWS_SSH_KEY_ID']%>"
  ssh_key: "<%= ENV['AWS_SSH_KEY']%>"

provisioner:
  name: chef_solo

platforms:
  - name: centos65
    driver:
      image_id: ami-8997afe0
      username: root
      region: us-east-1
      availability_zone: us-east-1c

suites:
  - name: default
    run_list:
    attributes:
{% endcodeblock %}

Before running any Test Kitchen commands, make sure you set the appropriate environment variables as shown below (with your own values):

Linux and Mac OS X:

    export AWS_ACCESS_KEY_ID="ABCDEFGHI123JKLMNOPQ"
    export AWS_SECRET_ACCESS_KEY="abcdefghijklmnopqrstuvwyz"
    export AWS_SSH_KEY_ID="keyid1234"
    export AWS_SSH_KEY="$HOME/ec2/$AWS_SSH_KEY_ID.pem"

Windows Command Prompt:

    set AWS_ACCESS_KEY_ID=ABCDEFGHI123JKLMNOPQ
    set AWS_SECRET_ACCESS_KEY=abcdefghijklmnopqrstuvwyz
    set AWS_SSH_KEY_ID=keyid1234
    set AWS_SSH_KEY=%USERPROFILE%/ec2/%AWS_SSH_KEY_ID%.pem

Windows Powershell:

    $env:AWS_ACCESS_KEY_ID="ABCDEFGHI123JKLMNOPQ"
    $env:AWS_SECRET_ACCESS_KEY="abcdefghijklmnopqrstuvwyz"
    $env:AWS_SSH_KEY_ID="keyid1234"
    $env:AWS_SSH_KEY="$env:userprofile/ec2/$env:aws_ssh_key_id.pem"

The output of `kitchen list` should resemble the following:

    $ kitchen list
    Instance           Driver  Provisioner  Last Action
    default-centos65  Ec2     ChefSolo     <Not Created>

Spin up the node with `kitchen create`:

    $ kitchen create default-centos65
    -----> Starting Kitchen (v1.2.2.dev)
    -----> Creating <default-centos65>...
           EC2 instance <i-5b6f2b70> created.
    ...........       (server ready)
           Waiting for ec2-54-197-34-184.compute-1.amazonaws.com:22...
           Waiting for ec2-54-197-34-184.compute-1.amazonaws.com:22...
           Waiting for ec2-54-197-34-184.compute-1.amazonaws.com:22...
           Waiting for ec2-54-197-34-184.compute-1.amazonaws.com:22...
           (ssh ready)\n
           Finished creating <default-centos65> (3m2.97s).
    -----> Kitchen is finished. (3m3.40s)

NOTE:

You may be prompted to opt in and accept the terms and subscribe to using the AWS Marketplace CentOS image the first time you spin up an image.  The `kitchen-ec2` driver will provide you with a link to the opt in URL.

NOTE:

You might not be able to create CentOS images in all availability zones.  The `kitchen-ec2` driver will advice you of your availability zone options if there is an issue with your availability zone choice.

Install Chef Client with `kitchen setup`.  `kitchen destroy` will delete your EC2 instance.

Refer to the `kitchen-ec2` driver documentation on https://github.com/test-kitchen/kitchen-ec2 for more information additional `.kitchen.yml` settings.

## Google Compute Engine Cloud Provider (kitchen-gce)

### kitchen-gce Setup

Create a Google Compute Engine project in the Google Developers Console at https://console.developers.google.com.  Create a Service Account Key by navigating to _APIs & auth_ > _Credentials_.  Under OAuth start the process by clicking on the _CREATE NEW CLIENT ID_ button as shown here:

{% img center /images/chapa01/gce_create_new_client_id.png [Create New Client ID] %}

On the _Create Client ID_ dialog, choose _Service account_ then click on _Create Client ID_ as shown below.  This will generate a private key file along with a password.  Record this information, as it is the only time it will be displayed.

{% img center /images/chapa01/gce_create_service_account.png [Create Service Account] %}

Make note of the _Email address_ field for the Service Account (not to be confused with the project owner's Email Address at the top of the page) as shown in the following.  You'll be recording this in the `google_client_email` field in the `.kitchen.yml`.

{% img center /images/chapa01/gce_client_email.png [Google Client Email] %}

If you do not already have an SSH key pair to login, create them using `ssh-keygen` or an equivalent tool.  Register the public key in the Google Developer Console.  The default file name for a public key is `$HOME/.ssh/id_rsa.pub`.  Navigate to _Compute_ > _Compute Engine_ > _Metadata_ on the Google Developers Console.  Make sure the _SSH_keys_ is selected in the panel on the right, then click on the _Add SSH key_ button as shown in the following:

{% img center /images/chapa01/gce_add_ssh_key.png [Add SSH key to a GCE Project] %}

Copy the public key `id_rsa.pub` file contents to the clipboard and paste it into the _Enter entire key data_ field.  Click on the _Done_ button to save.

{% img center /images/chapa01/gce_ssh_keys.png [Register SSH Public Key] %}

Run the following `kitchen init` command to add Test Kitchen support to your project using the `kitchen-gce` driver:

    $ kitchen init --driver=kitchen-gce --create-gemfile
          create  .kitchen.yml
          create  test/integration/default
          create  Gemfile
          append  Gemfile
          append  Gemfile
    You must run `bundle install' to fetch any new gems.

Run `bundle install` to fetch any new gems.

### kitchen-gce .kitchen.yml Example

Since the project, client e-mail and key location are sensitive information and differ between users, it is recommended that you store them in environment variables instead of directly in your `.kitchen.yml` file.  This way, you can share your `.kitchen.yml` file with others and store it in source control.  You can use an embedded Ruby template in a `.kitchen.yml` file to load values from the environment.  Here is an example `kitchen.yml` which spins up a CentOS 6.5 sandbox environment, loading the project and client e-mail from corresponding environment variables:

{% codeblock gce/.kitchen.yml lang:ruby %}
 ---
driver:
  name: gce
  google_project: "<%= ENV['GOOGLE_PROJECT']%>"
  google_client_email: "<%= ENV['GOOGLE_CLIENT_EMAIL']%>"
  google_key_location: "<%= ENV['GOOGLE_KEY_LOCATION']%>"
  
provisioner:
  name: chef_solo

platforms:
  - name: centos65
    driver: gce
      area: us
      image_name: centos-6-v20140619

suites:
  - name: default
    run_list:
    attributes:
{% endcodeblock %}

Before running any Test Kitchen commands, make sure you set the appropriate environment variables as shown below (with your own values):

Linux and Mac OS X:

    export GOOGLE_PROJECT="alpha-bravo-123"
    export GOOGLE_CLIENT_EMAIL="123456789012@developer.gserviceaccount.com"
    export GOOGLE_KEY_LOCATION="$HOME/gce/1234567890abcdef1234567890abcdef12345678-privatekey.p12"

Windows Command Prompt:

    set GOOGLE_PROJECT=alpha-bravo-123
    set GOOGLE_CLIENT_EMAIL=123456789012@developer.gserviceaccount.com
    set GOOGLE_KEY_LOCATION=%USERPROFILE%/gce/1234567890abcdef1234567890abcdef12345678-privatekey.p12

Windows Powershell:

    $env:GOOGLE_PROJECT="alpha-bravo-123"
    $env:GOOGLE_CLIENT_EMAIL="123456789012@developer.gserviceaccount.com"
    $env:GOOGLE_KEY_LOCATION="$env:userprofile/gce/1234567890abcdef1234567890abcdef12345678-privatekey.p12"

The output of `kitchen list` should resemble the following:

    $ kitchen list
    Instance          Driver  Provisioner  Last Action
    default-centos65  Gce     ChefSolo     <Not Created>

Spin up the node with `kitchen create`:

    $ kitchen create default-centos65
    -----> Starting Kitchen (v1.2.2.dev)
    -----> Creating <default-centos65>...
           GCE instance <default-centos65-31681aab-e6a2-494b-99cb-9b920a1f6284> created.
    ..       (server ready)
           (ssh ready)
           Finished creating <default-centos65> (1m26.70s).
    -----> Kitchen is finished. (1m28.18s)

Install Chef Client with `kitchen setup`.  `kitchen destroy` will delete your Google Compute Engine instance.

Refer to the `kitchen-gce` driver documentation on https://github.com/anl/kitchen-gce for more information on additional `.kitchen.yml` settings.

## Rackspace Cloud Provider (kitchen-rackspace)

### kitchen-rackspace Setup

Login to the Cloud Sites Control Panel at https://manage.rackspacecloud.com/pages/Login.jsp  Navigate to _Your Account_ > _API Access_ to display your username and API key as shown in below:

{% img center /images/chapa01/rackspace_api_key_image.png [Rackspace API Key] %}

Run the following `kitchen init` command to add Test Kitchen support to your project using the `kitchen-rackspace` driver:

    $ kitchen init --driver=kitchen-rackspace --create-gemfile
          create  .kitchen.yml
          create  test/integration/default
          create  Gemfile
          append  Gemfile
          append  Gemfile
    You must run `bundle install' to fetch any new gems.

Run `bundle install` to fetch any new gems.

### kitchen-rackspace .kitchen.yml Example

Since the username and API Key are sensitive information and differ between users, it is recommended that you store them in environment variables instead of directly in your `.kitchen.yml` file.  This way, you can share your `.kitchen.yml` file with others and store it in source control.  You can use an embedded Ruby template in a `.kitchen.yml` file to load values from the environment.  Here is an example `kitchen.yml` which spins up a CentOS 6.5 sandbox environment, loading the project and client e-mail from corresponding environment variables:

{% codeblock rackspace/.kitchen.yml lang:ruby %}
 ---
driver:
  require_chef_omnibus: true
  name: rackspace
  rackspace_username: "<%= ENV['RACKSPACE_USERNAME']%>"
  rackspace_api_key: "<%= ENV['RACKSPACE_API_KEY']%>"
  public_key_path: "<%= ENV['RACKSPACE_PUBLIC_KEY_PATH']%>"

provisioner:
  name: chef_solo

platforms:
  - name: centos65
    driver: rackspace
      image_id: "592c879e-f37d-43e6-8b54-8c2d97cf04d4"
      flavor_id: "performance1-1"

suites:
  - name: default
    run_list:
    attributes:
{% endcodeblock %}

Before running any Test Kitchen commands, make sure you set the appropriate environment variables as shown below (with your own values):

Linux and Mac OS X:

    export RACKSPACE_USERNAME="alice"
    export RACKSPACE_API_KEY="abcdef0123456789abcdef0123456789"
    export RACKSPACE_PUBLIC_KEY_PATH="$HOME/.ssh/id_rsa.pub"

Windows Command Prompt:

    set RACKSPACE_USERNAME=alice
    set RACKSPACE_API_KEY=abcdef0123456789abcdef0123456789
    set RACKSPACE_PUBLIC_KEY_PATH=%USERPROFILE%/.ssh/id_rsa.pub

Windows Powershell:

    $env:RACKSPACE_USERNAME="alice"
    $env:RACKSPACE_API_KEY="abcdef0123456789abcdef0123456789"
    $env:RACKSPACE_PUBLIC_KEY_PATH="$env:userprofile/.ssh/id_rsa.pub"

The output of `kitchen list` should resemble the following:

    $ kitchen list
    Instance          Driver     Provisioner  Last Action
    default-centos65  Rackspace  ChefSolo     <Not Created>

Spin up the node with `kitchen create`:

    $ kitchen create default-centos65
    -----> Starting Kitchen (v1.2.2.dev)
    -----> Creating <default-centos65>...
           Rackspace instance <9456b985-3a41-4cb0-a3cf-7536cc15baf7> created.
           (server ready)
           (ssh ready)
           Finished creating <default-centos65> (0m37.77s).
    -----> Kitchen is finished. (0m38.21s)

Then install Chef Client with `kitchen setup`.  `kitchen destroy` will delete your instance on Rackspace.

Refer to the `kitchen-gce` driver documentation on https://github.com/test-kitchen/kitchen-rackspace for more information on additional `.kitchen.yml` settings.

# Linux Container Drivers

You can regard Linux Containers to be a resource-efficient variant of virtual machines.  As shown in the following diagram, Linux containers trade off the flexibility (and overhead) of being able to run different operating systems in each guest to minimize resource consumption by having all guests share the same OS kernel.  In container environments, guests are isolated like virtual machines using more lightweight mechanisms around Linux processes instead.

{% img center /images/chapa01/tkdriver.006.png [Virtual Machines versus Containers] %}

This idea has its origins in attempts to provide better process isolation to _chroot jails_.  _chroot_ is a Unix command that facilitates creating a separate virtualized copy of the operating system by changing the apparent _root directory_ (/) to processes running within this copy of the operating system.  Other variants of Unix have added extensions to this _chroot_ mechanism to provide better isolation of the guest process, such as FreeBSD jails and Solaris Containers.  Linux Containers bring this process-based isolation mechanism to the standard Linux kernel via a recently added kernel feature called _control groups_.

As of this writing, there are no container-like Test Kitchen drivers for Windows.  Microsoft is working on adding similar lightweight virtualization technology to Windows via its Drawbridge virtalization technology[http://research.microsoft.com/en-us/projects/drawbridge/].  The only equivalent to Linux Containers in Windows at this moment is Microsoft Applications Virtualization (App-V), which has been around for quite some time, but it has a major drawback in requiring modification of target applications in order to work with the system, so it is not widely used.

The following diagram shows the steps in the sandbox environment creation process for containers.  It is identical to the host-based model presented previously, just using lightweight, isolated container processes instead of full-blown virtual machines.

1. Test Kitchen invokes the container driver (`kitchen-docker` or `kitchen-lxc`) to create a container instance.
2. The Test Kitchen driver uses the operating system APIs for Linux Containers to create a new instance for our sandbox environment.
3. Once the sandbox environment is running, Test Kitchen links the instance for communication.

{% img center /images/chapa01/tkdriver.007.png [Sandbox environment creation with kitchen-docker] %}

As of this writing, Test Kitchen drivers for Linux Containers do not support functionality equivalent to synchronized folders.  All Test Kitchen commands use `scp` to transfer files from your host to the container instance.  For any other file sharing beyond what is supported by Test Kitchen, you'll need to make direct use of the file sharing mechanisms provided by the container driver being used.  This is where Docker shines, as it supports data volume containers which bypass container image layering.  Data volume containers are an ideal way to share data between containers.  It is also possible to mount host directories in a container, but that has more limited use cases.  Refer to the documentation on your container provider for more information.

You can combine together virtual machines with Linux containers to use containers on platforms that do not have native container support, like Mac OS X and Windows.  The following diagram presents an overview of the setup.  With virtual machines, it is usually not possible to nest virtualization software instances.  Running virtualization software _inside_ guest OS instances is either prohibited or painfully slow.  However, it's perfectly fine to run Linux Containers within a virtual machine.  To the outer virtualization software, the container instances are merely Linux processes.

{% img center /images/chapa01/tkdriver.003.png [Docker running in a virtual machine] %}

In the next section on Docker, we'll show you how to use this technique for readers running Mac OS X or Windows.  Neither platform supports Linux containers natively on the host.  Chef Software uses a Docker-based VM in training classes, so that students with laptops running Mac OS X or Windows can use the same setup as the students using Linux.  This approach also saves money, as Chef Software uses cloud providers for training, and these providers charge based the number of instances and resources used.  The lightweight Docker instances consume fewer resources and only require one running instance on the cloud provider - all the other instances are just lightweight container instances, which cloud providers (currently) do not charge extra.  You may want to consider using Linux Containers in a similar fashion to save money if you make heavy use of third-party virtualization or cloud providers, like we do.

## Docker Driver (kitchen-docker)

If you are using Linux, refer to the Docker installation guide for instructions on how to install and configure Docker in your environment: http://www.docker.com/.

### Chef Training Environment Setup

Skip ahead to the next secion if you are using Linux and already have Docker installed.  Otherwise, you'll need to spin up a virtual machine with Docker installed in order to play around with a container environment.

We've created a Chef training environment that has Docker and the Chef Development Kit used in this book preinstalled on a Linux virtual machine.  We use this same instance in official Chef training.  It's also a handy environment for playing around with containers using Test Kitchen.

First, make sure you install Vagrant and VirtualBox or Vagrant and VMware.

Create a directory for the Chef training environment project called `chef` and make it the current directory.

    $ mkdir chef
    $ cd chef

Add Test Kitchen support to the project using the default `kitchen-vagrant` driver by running `kitchen init`.  Then run `bundle install` to install the necessary gems for the Test Kitchen driver.

    $ kitchen init --create-gemfile
          create  .kitchen.yml
          create  test/integration/default
          create  Gemfile
          append  Gemfile
          append  Gemfile
    You must run `bundle install' to fetch any new gems.

    $ bundle install
    Fetching gem metadata from https://rubygems.org/..........
    Fetching additional metadata from https://rubygems.org/..
    Resolving dependencies...
    Using mixlib-shellout (1.4.0)
    Using net-ssh (2.9.1)
    Using net-scp (1.2.1)
    Using safe_yaml (1.0.3)
    Using thor (0.19.1)
    Using test-kitchen (1.2.1)
    Using kitchen-vagrant (0.15.0)
    Using bundler (1.5.2)
    Your bundle is complete!
    Use `bundle show [gemname]` to see where a bundled gem is installed.

Modify the `.kitchen.yml` file to use the Chef training image as shown in the following `.kitchen.yml`:

{% codeblock chef/.kitchen.yml lang:ruby %}
 ---
driver:
  name: vagrant

provisioner:
  name: chef_solo

platforms:
  - name: learningchef
    driver:
      box: learningchef/cheftraining
      box_url: learningchef/cheftraining

suites:
  - name: default
    run_list:
    attributes:
{% endcodeblock %}

Run `kitchen create` to spin up the image:

    $ kitchen create
    -----> Starting Kitchen (v1.2.2.dev)
    -----> Creating <default-learningchef>...
           Bringing machine 'default' up with 'virtualbox' provider...
           ==> default: Importing base box 'learningchef/chefdk-box'...
           ==> default: Matching MAC address for NAT networking...
           ==> default: Checking if box 'learningchef/chefdk-box' is up to date...
           ==> default: Setting the name of the VM: default-learningchef_default_1404728110875_23069
           ==> default: Fixed port collision for 22 => 2222. Now on port 2200.
           ==> default: Clearing any previously set network interfaces...
           ==> default: Preparing network interfaces based on configuration...
               default: Adapter 1: nat
           ==> default: Forwarding ports...
               default: 22 => 2200 (adapter 1)
           ==> default: Booting VM...
           ==> default: Waiting for machine to boot. This may take a few minutes...
               default: SSH address: 127.0.0.1:2200
               default: SSH username: vagrant
               default: SSH auth method: private key
               default: Warning: Remote connection disconnect. Retrying...
           ==> default: Machine booted and ready!
           ==> default: Checking for guest additions in VM...
           ==> default: Setting hostname...
           ==> default: Machine not provisioning because `--no-provision` is specified.
           Vagrant instance <default-learningchef> created.
           Finished creating <default-learningchef> (0m36.99s).
    -----> Kitchen is finished. (0m37.44s)

Then run `kitchen login` to use Docker!  Note that the image also has the latest Chef Development Kit installed (as of this writing).  You will be running the Test Kitchen Docker driver _inside_ this virtual machine.  It has been pre-populated with all the necessary files to spin up the CentOS 6.5 images used in the exercises for this book:

    $ kitchen login
    Welcome to Ubuntu 14.04 LTS (GNU/Linux 3.13.0-24-generic x86_64)

     * Documentation:  https://help.ubuntu.com/
    Welcome to the Learning Chef training environment
    Last login: Fri May 23 13:49:31 2014 from 10.0.2.2
    vagrant@default-learningchef:~$ docker --version
    Docker version 0.11.1, build fb99f99
    vagrant@default-learningchef:~$ kitchen --version
    Test Kitchen version 1.2.2.dev
    vagrant@default-learningchef:~$

NOTE:

Sharp-eyed readers might notice that this is an Ubuntu image.  It is perfectly OK to spin up CentOS images on Ubuntu, as long as you use a version that shares the same kernel!

TIP:

At first, the multiple layers of instances might be a little confusing.  Refer back to the Docker diagram shown previously so that you can keep the big picture of this setup in mind.  Also, modifying the command prompts so they clearly indicate which environment is the VM and which environment is a container instance is strongly recommended.

### kitchen-docker Setup

Run the following `kitchen init` command to add Test Kitchen support to your project using the `kitchen-docker` driver:

    $ kitchen init --driver=kitchen-docker --create-gemfile
          create  .kitchen.yml
          create  test/integration/default
          create  Gemfile
          append  Gemfile
          append  Gemfile
    You must run `bundle install' to fetch any new gems.

Run `bundle install` to download and install any required gems.

### kitchen-docker .kitchen.yml Example

The following `.kitchen.yml` presents an example which spins up a CentOS 6.5 sandbox environment:

{% codeblock docker/.kitchen.yml lang:ruby %}
 ---
driver:
  name: docker

provisioner:
  name: chef_solo

platforms:
  - name: centos65
    driver:
      image_id: 3448641
      region_id: 4

suites:
  - name: default
    run_list:
    attributes:
{% endcodeblock %}

The output of `kitchen list` should resemble the following:

    $ kitchen list
    Instance          Driver  Provisioner  Last Action
    default-centos65  Docker  ChefSolo     <Not Created>

Spin up the node with `kitchen create`:

    $ kitchen create
    -----> Starting Kitchen (v1.2.2.dev)
    -----> Creating <default-centos65>...
           Step 0 : FROM centos:latest
           Pulling repository centos
            ---> 0c752394b855
    ...
           Waiting for localhost:49153...
           Waiting for localhost:49153...
           Finished creating <default-centos65> (1m19.28s).
    -----> Kitchen is finished. (1m19.34s)

At the time of this writing, due to some issues with `kitchen-docker`, you may be prompted for `kitchen@localhost's password`.  The password is `kitchen`
   
    $ kitchen login
    kitchen@localhost's password: kitchen
    Last login: Mon Jul  7 11:37:14 2014 from 172.17.42.1
    [kitchen@55f29336b435 ~]$ cat /etc/redhat-release
    CentOS release 6.5 (Final)
    [kitchen@55f29336b435 ~]$ exit
    logout
    Connection to localhost closed.

Install Chef Client with `kitchen setup`.  `kitchen destroy` will delete container instance.

Refer to the `kitchen-docker` driver documentation on https://github.com/portertech/kitchen-docker for more information on additional `.kitchen.yml` settings.

# Physical Machine Drivers

As of this writing, Test Kitchen does not currently support `chef-metal`.  It is currently planned to provide robust support for managing sandbox environments running on physical machines using `chef-metal` (though plans sometimes change).

Until Test Kitchen supports `chef-metal`, the only way to use Test Kitchen with physical machines currently (other than your local host) is to use the `kitchen-ssh` driver.  This is actually a generic way to integrate any kind of machine with Test Kitchen, not just physical machines.  As long as the machine accepts `ssh` connections, it will work.

The following diagram shows an overview of the Test Kitchen instance creation process using `kitchen-ssh`.  It is similar to the creation process used for cloud instances with the Test Kitchen environment being run on a remote machine, but there is only one step because an isolated sandbox instance is not created.  The `kitchen-ssh` driver merely links up an SSH communication channel with Test Kitchen in the remote machine's host environment.

{% img center /images/chapa01/tkdriver.008.png [Sandbox environment creation with kitchen-ssh] %}

It is assumed that you are using some other method outside of Test Kitchen to be able to easily reset the environment.  Also, since it does not spin up a new instance, you will need to make sure the machine that you are linking to has CentOS 6 installed to match the exercises in this book.

## Driver for any server with an SSH address (kitchen-ssh)

Run the following `kitchen init` command to add Test Kitchen support to your project using the `kitchen-ssh` driver:

    $ kitchen init --driver=kitchen-ssh --create-gemfile
          create  .kitchen.yml
          create  test/integration/default
          create  Gemfile
          append  Gemfile
          append  Gemfile
    You must run `bundle install' to fetch any new gems.

Run `bundle install` to fetch any required gems.

### kitchen-ssh .kitchen.yml Example

The following `.kitchen.yml` assumes that you are connecting to an existing CentOS 6.5 environment with an SSH server running.  Change the `hostname:`, `username:` and `password:` fields accordingly to match your remote machine's settings:

{% codeblock ssh/.kitchen.yml lang:ruby %}
 ---
driver:
  name: ssh

provisioner:
  name: chef_solo

platforms:
  - name: centos65
    driver:
      hostname: 192.168.33.33
      username: alice
      password: averysecretpassword

suites:
  - name: default
    run_list:
    attributes:
{% endcodeblock %}

The output of `kitchen list` should resemble the following:

    Instance          Driver  Provisioner  Last Action
    default-centos65  Ssh     ChefSolo     Created

Initiate a connection to the node with `kitchen create`.  You could also run `kitchen login` without needing to run `kitchen create` in this case, as `kitchen create` does nothing:

    $ kitchen create
    -----> Starting Kitchen (v1.2.2.dev)
    -----> Creating <default-centos65>...
           Kitchen-ssh does not start your server '192.168.33.33' but will look for an ssh connection with user 'alice'
    ---
           Kitchen-ssh found ssh ready on host '192.168.33.33' with user 'alice'

           Finished creating <default-centos65> (0m0.01s).
    -----> Kitchen is finished. (0m0.02s)

Install Chef Client with `kitchen setup`.  For this driver, `kitchen destroy` does nothing, just like `kitchen create`, besides updating the status in Test Kitchen.

Refer to the `kitchen-ssh` driver documentation on https://github.com/neillturner/kitchen-ssh/blob/master/lib/kitchen/driver/ssh.rb for more information on additional `.kitchen.yml` settings.