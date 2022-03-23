## How to prepare and deploy VM image to create multiple VMs ##

When we need to create multiple virtual machines (VMs) on different hosts, preparation and VM image cloning can save us a lot of time. One of best practices is, we create minimal image which includes everything required for further cloning. For example we need to create group of web servers and group of DB servers.

We will use the same Linux distribution for all our VMs because we would like to unify our automation scripts and configuration management. 

1. We create base (minimal) VM image which includes all optimizations and settings we need for every cloned VM instance. There should be security and performance settings in place, everything else what we will not need, removed or disabled. Best way is to automate all these steps with appropriate tools.
2. When we have working minimal base VM, we use Bash/Ansible scripts for cloning, preparing and deploying of the new instance. This way we can have configurations and settings saved in (private) git and thus use IaaC (Infrastructure as a Code) concept.
3. With configuration Bash/Ansible scripts we then automate offline preparation of VM images. This includes individual VM network settings if not using DHCP, ssh keys, host name, firewall, .... settings which are a must before we deploy and start VM to work correctly on the destination host and on destination network. One of approaches which we use in case of QEMU based virtualization solutions  (Xen, KVM) is to cold mount the volume to get file access from host with the tool 'guestmount'.
4. After 'cold' preparation of VM image with Ansible/Bash we deploy VM image to the destination host and start VM. When VM is running we use additional Ansible/Bash scripts to auto check working instance and install additional software (web server, db server, ...) whatever this VM or VM group will need to do its job.
5. When all services for specific VM are in place we deploy content (web pages, db data, ...).
6. Repeat steps 2 to 5 for any additional VMSs or VM groups we need.

## Configuration files (create/update) automation ##
There can be cases where we need to configure and update services like Postfix mail server which for example uses 64 IPs with unique SSL certs. In this case we have blocks of configuration which do repeat with small and deterministic changes. For example only IP, FQND and SSL cert name in every configuration/settings block changes. We can do this by hand but it is time consuming, human error prone and if we later need to change/update some or all of these settings we need to repeat the whole process.

This is why we use configuration generators (this can be simple Bash script) which then take template blocks of configuration and apply dynamic data from dynamic data source. Simplest form od dynamic data source is plain text file of 64 IP addresses and associated domain names (FQDNs). An additional advantage here is when we need to update these configuration blocks for various reasons (new version of Postfix mail server, security hardening, change of IPs, other requirements...) we will just change the template configuration blocks and re-run configuration generator which will update (re-generate) complete service configuration which can consist of more than 1000 lines in the end. And this is not something we would like to edit by hand.

## Auto testing of an infrastructure setup/changes ##
Additionally we have to think about automatic testing of infrastructure changes. There are many tools for doing this, yet specific tasks can best be automated with Bash/Ansible scripts and other dedicated tools. For example when we change public IPs we need to test complete service settings, validate SSL certs and double check rDNS (PTR) records. This kind of auto testing tools is also used for continuous monitoring and immediate reporting in case of detected problems or discrepancies.
Important 

All Bash/Ansible scripts and automatic configuration generators must produce idempotent result. This means if we run the same Bash/Ansible or configuration generation tool more than once, final result must always be the same as the first time.

## Conclusion ##
Described above is just a simple road map how to create repeatable, maintainable, versionable IaaC. We save scripts, configuration templates and ansible playbooks in git repositories. Some things can be done different way, but we always have to strive to efficiency for business and IT workflows.  
With appropriate layering of Ansible/Bash automation scripts and reusable templates we can achieve infrastructure maintenance and permissions/segregation for DevSecOps. For example on test servers we can give developers authorization to only use the automation to deploy new versions of web application pages, but other server maintenance, security will be out of scope for developer group.

Thanks for reading :-)

Author: Matjaz Skoda
