subscription-manager
====================

The Subscription Manager package provides programs and libraries
to allow users to manage subscriptions and yum repositories
from the  Candlepin.

 - http://candlepinproject.org/
 - https://fedorahosted.org/subscription-manager/
 - https://github.com/candlepin/subscription-Manager

Local Installation
------------------
Consider using Vagrant instead (see below) for development as it can be a much
more consistent and easy experience.

In order to build, develop, and test locally, please follow
[instructions at candlepinproject.org](http://www.candlepinproject.org/docs/subscription-manager/installation.html#installation-of-upstream-from-source-code).

Due to unintuitive behavior with `sys.path`
(see https://github.com/asottile/scratch/wiki/PythonPathSadness),
`python src/subscription_manager/scripts/subscription_manager.py` does not work
as expected. One can run the script like this instead:

```bash
PYTHONPATH=./src:./syspurpose/src python -m subscription_manager.scripts.subscription_manager
```

Similar for other bin scripts:

```bash
PYTHONPATH=./src python -m subscription_manager.scripts.rct
PYTHONPATH=./src python -m subscription_manager.scripts.rhn_migrate_classic_to_rhsm
# ... etc.
```

(You can also just export `PYTHONPATH` instead of setting it in each command).

Vagrant
-------

The setup that most developers are using is `vagrant-libvirt` with
`vagrant-hostmanager` installed on Fedora via:

```bash
dnf install vagrant-libvirt vagrant-hostmanager
```

We are avoiding coupling to libvirt, but use of VirtualBox is less tested.
If you'd like to ensure vagrant uses libvirt, you can set
`VAGRANT_DEFAULT_PROVIDER=libvirt` in your environment.

`vagrant up` can be used to spin up various VMs set up for development work.
These VMs are all configured using the included ansible role "subman-devel".
The python paths and `PATH` inside these environments are modified so that
running `subscription-manager` or `subscription-manager-gui` will use
scripts and modules from the source code.

Cockpit can be accessed at port 9090 on the VM, for example:
https://centos7.subman.example.com:9090/

**NOTE**: Use of hostnames per above requires the vagrant hostmanager plugin.
Cockpit credentials are the same as user credientials;
i.e. User name: `vagrant` password: `vagrant`

There are two links added in the cockpit interface: one for the
subscription-manager cockpit plugin itself, and then one which runs integration
tests for the D-Bus interface.

Currently, the source is set up as a vagrant shared folder using rsync. This
means that it is necessary to use `vagrant rsync` to sync changes with the
host if desired. `vagrant rsync-auto` can be used to monitor the directory for
changes and then sync when appropriate.

The ansible role that provisions the VMs tries to find the IP address of
candlepin.example.com, so if the candlepin vagrant image is started first,
then the box can resolve it. (If it is started later, `vagrant provision` can
be used to update the VM's `hosts` file).

Additionally, the `Vagrantfile` is set up for sharing base VMs with
[katello/forklift](https://github.com/theforeman/forklift). Specifically,
forklift plugins can be added to a subscription-manager checkout beneath
`vagrant/plugins`in order to provide additional base images.

If RHEL-based images are added, then the `Vagrantfile` uses the values of
`SUBMAN_RHSM_USERNAME`, `SUBMAN_RHSM_PASSWORD`, `SUBMAN_RHSM_HOSTNAME`,
`SUBMAN_RHSM_PORT`, and `SUBMAN_RHSM_INSECURE` to register and auto-attach
during provisioning (best done in `.bashrc` or similar). If unspecified,
hostname and port are left alone (i.e. whatever is in the VM's `rhsm.conf` will
be untouched).

To register against subscription.rhsm.redhat.com, `.bashrc` might contain:
```bash
export SUBMAN_RHSM_USERNAME=foobar
export SUBMAN_RHSM_PASSWORD=password
```
(Replace username and password with actual values).

To register against a local candlepin instance, `.bashrc` might contain:
```bash
export SUBMAN_RHSM_HOSTNAME=candlepin.example.com
export SUBMAN_RHSM_PORT=443
export SUBMAN_RHSM_INSECURE=1
export SUBMAN_RHSM_USERNAME=foobar
export SUBMAN_RHSM_PASSWORD=password
```

(Replace username and password with actual values).

Note, however, since the registration is necessary to download RPMs to set up
the VM for development, registering against a local candlepin might not be
particularly useful (at least not for initial provisioning).


> If you want use proxy server to play with subscription you can run it using vagrant:
>       vagrant up proxy-server

The current info about proxy

url | proxy-server.subman.example.com:3128
username | proxyuser
password | password

D-Bus Development
-----------------
In a vagrant VM, the `com.redhat.RHSM1` service along with related files 
(scripts, policy files, etc.) are linked to those from the source. However, it
is necessary to restart the D-Bus service if edits are made while it is running
with, for example, `sudo systemctl restart rhsm`.

Cockpit
-------

The easiest way to get started with cockpit plugin development is Vagrant.
Inside the VM, from the directory `/vagrant/cockpit`, the following commands
can be used:

 - `yarn install` - fetch dependencies, and update the lockfile if necessary.
 - `npm run build` - do a build of the JavaScript source.
 - `npm run watch` - monitor the source for changes and rebuild the cockpit
  plugin when necessary.

See `cockpit/README.md` for more detailed information on cockpit development.


syspurpose
---------
The syspurpose utility manages certain user-definable values tracked in
the /etc/rhsm/syspurpose/syspurpose.json file (in json format).

See ./packages/syspurpose/README.md for details on getting started


Testing
-------
We run tests using nose (see candlepinproject.org for details).  Some tests
are not run by default. For example, since we are not maintaining the GTK GUI
for all platforms, they are not run by default. They can be included via
`-a gui` option for the nose command. It is recommended if you run the GUI
tests to also use `--with-xvfb` in order to use Xvfb instead of spawning
GTK windows in your desktop session (ex. `nosetests -a gui --with-xvfb`).

[More details about testing](./TESTING.md)

Troubleshooting
---------------

If you are working inside one of the vagrant boxes and you find subscription-manager and/or
subscription-manager-gui will not work with output that looks like the following:
"Unable to find Subscription Manager module.
Error: libssl.so.10: cannot open shared object file: No such file or directory"

You should be able to run `vagrant provision [vm-name]` from the host machine to fix the issue.

This issue can happen if the python-rhsm/build or python-rhsm/build_ext directories are copied to
the virtual machine and the virtual machine provides different libraries than those available on
the build host.

TEST
