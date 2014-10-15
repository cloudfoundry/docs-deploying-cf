---
title: Preparing Your Local Machine for MicroBOSH Deployment
---

We use a MicroBOSH to deploy BOSH.
A MicroBOSH is a single VM that includes all of the BOSH components.
In order to deploy BOSH, first install a MicroBOSH, then run the BOSH deployer.
We distribute the libraries needed to get started via a Ruby gem.

Preparing your local computer for deploying MicroBOSH is the same
regardless of the infrastructure you will deploy to.
If you are a Ruby developer, you may have some of the steps below already
completed.

1. Install Ruby

2. Install the Git command line tool

3. Install RubyGems

4. Install Gems for BOSH CLI

## <a id="install-ruby"></a>Install Ruby ##

You need either Ruby 1.9.3 or 2.0.0 installed locally on your computer.
We recommend using either [rvm](https://rvm.io/rvm/install) or [rbenv](https://github.com/sstephenson/rbenv) to manage your Ruby environment.

For instructions on installing Ruby, see these pages:

* [Ruby with rbenv](../common/install_ruby_rbenv.html)
* [Ruby with rvm](../common/install_ruby_rvm.html)

## <a id="install-git"></a>Install the Git Command Line Tool ##

See [http://git-scm.com/](http://git-scm.com/) for instructions to download and
install the Git command line tool on your local system.

Use the `git --version` command to confirm that you have successfully installed the Git command line tool.

<pre class="terminal">
$ git --version
git version 1.9.3
</pre>

## <a id="update-rubygems"></a>Update RubyGems ##

Use the `gem update --system` command to installs the Ruby package management
framework.
For more information about RebyGems, see [http://rubygems.org/pages/download](http://rubygems.org/pages/download).

<pre class="terminal">
$ gem update --system
Updating rubygems-update
Fetching: rubygems-update-2.4.2.gem (100%)
Successfully installed rubygems-update-2.4.2
Installing RubyGems 2.4.2
RubyGems 2.4.2 installed
</pre>

### If using rvm, you can create a new gemset and update package management

<pre class="terminal">
$ rvm use 2.0.0@bosh --create
</pre>

For Ubuntu:

<pre class="terminal">
$ sudo apt-get update
</pre>

For Fedora:

<pre class="terminal">
$ sudo yum update
</pre>

## <a id="bosh-cli-gems"></a>Install Gems for BOSH CLI ##

Run the following command to install the gems you need to run BOSH from the
command line.

<pre class="terminal">
$ gem install bosh_cli_plugin_micro --no-rdoc --no-ri
</pre>

<p class="note"><strong>Note</strong>: If the step above errors out referencing sqlite on aptitude based systems, execute `apt-get install sqlite-devel` or otherwise ensure that sqlite development packages have been installed, then try the previous step again.</p>

## <a id="bosh-status"></a>Confirm BOSH ##
Run `bosh status` to confirm you have installed BOSH:

<pre class="terminal">
$ bosh status
</pre>

## Go on to [Configuring AWS for MicroBOSH](./configure_aws_micro_bosh.html) or [Return to Index](./index.html)