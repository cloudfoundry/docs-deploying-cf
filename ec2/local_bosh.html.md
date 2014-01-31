---
title: Preparing Your Local Machine for Micro Bosh Deployment
---

We use Micro BOSH to deploy BOSH. A micro BOSH is a single VM that
includes all of the BOSH components. In order to deploy BOSH, install
a Micro BOSH and then run the BOSH deployer. The libraries we need to
get started are distributed via a Ruby gem.

Preparing your local computer for deploying Micro BOSH is the same
regardless of the infrastructure you will deploy to. You may have some
of the steps below already installed if you are a Ruby developer (and
if you are, you know which steps you can skip):

1. Install Ruby

2. Install the Git command line tool

3. Install RubyGems

4. Install Gems for BOSH CLI

### Install Ruby

You need either Ruby 1.9.3 or 2.0.0 installed locally on your computer.
We recommend
using either [rvm](https://rvm.io/rvm/install) or
[rbenv](https://github.com/sstephenson/rbenv) to manage
your Ruby environment.

For instructions on installing Ruby, see these pages:

* [Ruby with rbenv](../common/install_ruby_rbenv.html)
* [Ruby with rvm](../common/install_ruby_rvm.html)

### Install the Git command line tool

We leverage git for several downloads, so the git cli needs to be installed. Visit [http://git-scm.com/](http://git-scm.com/) for
instructions to download and install git on your local system. Then
validate that git was installed successfully and is available in your terminal:

<pre class="terminal">
  git --version
  git version 1.8.4.2
</pre>

### Update RubyGems

This installs Ruby’s package management framework. Additional details
are located [here](http://rubygems.org/pages/download).
<pre class="terminal">
  gem update --system
</pre>

### If using rvm, you can create a new gemset and update package management

<pre class="terminal">
  rvm use 2.0.0@bosh --create
</pre>

For Ubuntu:

<pre class="terminal">
  sudo apt-get update
</pre>

For Fedora:

<pre class="terminal">
  sudo yum update
</pre>

### Install Gems for BOSH CLI

Run the following command to install the gems required to run BOSH
from the command line. This step can take a few minutes.

<pre class="terminal">
  gem install bosh_cli_plugin_micro --no-rdoc --no-ri
</pre>

**Note**: If the step above errors out referencing sqlite on aptitude based systems, execute "apt-get install sqlite-devel", then try the previous step again or otherwise ensure that sqlite development packages have been installed.

Now run bosh status to confirm:

<pre class="terminal">
  bosh status
</pre>


###Go on to [Configuring AWS for Micro BOSH](./configure_aws_micro_bosh.html) or [Return to Index](./index.html)



