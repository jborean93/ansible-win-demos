# EC2 Windows AMI

This is an Ansible playbook that will create a new Windows EC2 instance and
then create an AMI out of this instance. It is designed as a POC to prove
that things work and as a guideline to show people how it could potentially be done

## Requirements

* Ansible 2.4 or newer
* boto installed in the Python running Ansible
* pywinrm installed in the Python running Ansible

## Components

### main.yml

The main playbook, this is split up into 3 different plays

* The first play will create the AWS security group and Windows instance according to the parameters specified. It will finally add the new instance to the `windows` group to the next play will run
* The second play is where all the configuration is done, in this example all it is doing is installing Vim and then shutting down the host with sysprep so that a new image will start with a WinRM HTTPS listener active but this can easily be expanded by calling whatever roles/tasks you want to run
* The third play waits until the host is shutdown, creates the AMI and finally removes the test host and security group

At the end of the play it will output some information about the AMI that was
created as well as the default Windows password used for each instance created
from the AMI.

### inventory.ini

The inventory file that contains an empty group called `windows` and the base
connection vars that are required for Ansible to connect with it. You can also
define the mandatory variables that are required in this playbook as set by
the `Variables` section below.

### unatend.xml.tmpl

This is the Windows answer file supplied to the sysprep process. What it does
is to recreate the WinRM HTTPS listeners once an instance is created from an
AMI but this is where you can put in custom scripts to run for your own
purposes if required.

### userdata.txt.tmpl

The initial user data supplied to the EC2 instance that the AMI is based off.
All this does is enable the HTTPS WinRM listener with CredSSP so that Ansible
can communicate with it.

### library/win_scheduled_task_new.ps1

This is just the newer `win_scheduled_task` module that is available in Ansible
2.5. Because this is to prove it works on 2.4 I've had to add it as a custom
module but this can be removed when 2.5 is available and released. Change the
name of the task from `win_scheduled_task_new` to `win_scheduled_task` in
`main.yml` once this is removed.

## Variables

The following variables must be set either in `inventory.ini` or passed as
extra args to `ansible-playbook -e ...`.

* `man_ec2_instance_type`: The AWS instance type to create the image on, e.g. `t2.micro`.
* `man_ec2_security_group`: The name of the security group to temporarily create the EC2 instance in, this is deleted once the image is created.
* `man_ec2_region`: The region to create the instance and AMI in, e.g.` ap-southeast-2`.
* `man_ec2_ami_id`: The base AMI used to create the initial image, this is not the ID to set for the created AMI.
* `man_ec2_unique_tag`: A unique name to create on the `Name` tag of the EC2 instance, useful when running things in parallel you would use different tags.
* `man_ec2_keypair`: The name of the keypair registered in AWS used to encrypt the initial Windows password.
* `man_ec2_keyfile_path`: The local path to the private key that is used to decrypt the initial Windows password.
* `man_ec2_aws_access_key`: The AWS Access key to use when authenticating with AWS.
* `man_ec2_aws_secret_key`: The AWS Secret key to use when authentication with AWS.

## Output

This will produce an AMI of a Windows host configured in the 2nd play. In this
example all it is doing is installing vim and making sure the HTTPS WinRM
listeners are set up after a new instance is create from the AMI.

The AMI will be called `test-ami` and the password for the Administrator user
will always be the same as what is printed at the end of the playbook. Note you
can easily modify this by creating your own custom setup script that is called
in `userdata.txt.tmpl` to read from the userdata parameter but in the interest
of time and brevity I decided not to go that route for this example.

