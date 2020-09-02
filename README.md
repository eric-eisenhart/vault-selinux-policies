# vault-selinux-policies

**NOT READY FOR USE AS OF 28-JULY-2020**

A set of base SELinux policies for Vault currently under development.

# Setting up test instance

```
cd terraform
make up
```

SSH to the instance then run:

```
sudo cloud-init status --wait
```

Wait until the cloud-init stuff has finished

Then scp `vault_selinux-1.1-1.el7.noarch.rpm` to the instance from this folder

Then from the instance:

```
sudo rpm -ivh vault_selinux-1.1-1.el7.noarch.rpm
```

You can check that the policy has applied properly by checking appropriate vault_t contexts in:

```
ls -alZ /opt/vault
ls -alZ /var/log/vault
ls -alZ /usr/sbin/vault
ls -alZ /etc/vault.d
```

You should then be able to start the vault server on the instance:

```
sudo systemctl start vault.service
```

From the centos user you can:
```
export VAULT_ADDR=http://127.0.0.1:8200
vault status
```

Hopefully, if you tail the audit log you shouldn't see AVC errors for vault:

```
sudo tail -f /var/log/audit/audit.log | grep vault
```

After you've initialized and unsealed vault you can also enable the audit engine to write to /var/log/vault/vault.log

```
vault audit enable file file_path=/var/log/vault/vault.log
```

Don't forget to tear down your infra after:

```
make down
```

From the terraform folder.

# Editing the source

The source is available in `vault_selinux-1.1-src` folder, you will probably want to shift this up to the instance too.

The file context mappings are in `vault.fc` - the interfaces are in `vault.if` and the main policy is in `vault.te`.

The `vault.sh` is used to deploy and bundle the policy into an rpm file. This is the main way to update the policy on the system, and was generated with a `sepolicy generate --init -n vault /usr/sbin/vault`

The `vault_selinux.spec` is also generated by the above, and is used to create the RPM.

If you update the fc, if or te files, you can then run `sh ./vault.sh` to re-deploy, then monitor audit log for errors.

If you want to then dynamically update the te file with changes detected in the logs you can run `sh ./vault.sh --update`

If you want to remove the policy then you can run a `sudo semodule -r vault`

# Updating the rpm

You'll need to shift the `vault_selinux-1.1-src` onto a Centos instance with the sepolicy pre-requisites (if you `make up` in the `terraform` folder that's what you get).

After you `sudo sh ./vault.sh` it will install the policy, plus, it will create a folder called `noarch` - and the RPM file will be in there. Any time you make changes to the `vault.te` file (or other sepolicy files), and re-run `sudo sh ./vault.sh` it will generate a new RPM file.

# References blogs etc

I did a _lot_ of googling to get my head around this, and realize I have a bunch of tabs open, so I'll try and capture my open tabs here:

* Sysadmin's guide to selinux https://opensource.com/article/18/7/sysadmin-guide-selinux
* How to generate a new initial policy using sepolicy-generate https://mgrepl.wordpress.com/2015/05/20/how-to-create-a-new-initial-policy-using-sepolicy-generate-tool/
* Fun with selinux presentation: https://mgrepl.fedorapeople.org/PolicyCourse/writingSELinuxpolicy_MUNI.pdf
* Selinux labels: https://wiki.gentoo.org/wiki/SELinux/Labels
* Using sesearch https://www.server-world.info/en/note?os=CentOS_7&p=selinux&f=11
* Basic policy editing examples https://www.linuxtopia.org/online_books/writing_SELinux_policy_guide/policy_editing_examples_12.html
* Customizing selinux policies https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/deployment_guide/sec-sel-policy-customizing
* Adjusting selinux policies https://download.geo.drweb.com/pub/drweb/unix/doc/HTML/ControlCenter/en/dw_8_install_selinux.htm
* I cloned the following two repos to find what certain interfaces, macros and methods did. https://github.com/SELinuxProject/refpolicy & https://github.com/fedora-selinux/selinux-policy
* Troubleshooting selinux problems https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/using_selinux/troubleshooting-problems-related-to-selinux_using-selinux
* Linux restorecon command examples https://www.thegeekstuff.com/2017/05/restorecon-examples/
* Walk through of the INN policy https://www.linuxtopia.org/online_books/writing_SELinux_policy_guide/case_study_13.html 
