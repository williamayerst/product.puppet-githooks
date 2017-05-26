
There are multiple steps to the installation of Git-Hooks into a repository to grant all levels of checking.
https://github.com/drwahl/puppet-git-hooks


# Local check
## Pull files and set executable
Git Clone the repo to your local machine, then set the pre-commit file to be executable:

`chmod +x <git-hook-repo>/pre-commit`

## Link files to target repo
Copy or Symlink the './pre-commit' file to <target repo>/.git/hooks/pre-commit i.e.

`ln -s <git-hook-repo>/pre-commit <target repo>/.git/hooks/pre-commit`

## Result
When you attempt to run a commit locally, it will refer to this file, which calls shell scripts to lint and check any changed files, for example:


```$ git add . && git commit -m "test file 2"
Error: Could not parse for environment production: Syntax error at ':' at C:/repos/product.puppet.module-keyboard/manifests/test.pp:9:37
Error: puppet syntax error in manifests/test.pp (see above)
Error: 1 syntax error(s) found in puppet manifests. Commit will be aborted.
Skipping puppet-lint check due to prior errors.
rspec not installed. Skipping rspec-puppet tests...
r10k not installed. Skipping r10k Puppetfile test...
Error: 1 subhooks failed. Please fix above errors.
```

# Remote check

## Install environment prerequisites
You need to install ruby, ruby-dev and make as prerequisites for the gems that are required by the checking process:

```apt-get install Ruby ruby-dev make
gem install rubygems-update
update_rubygems
gem install bundler
```

## Pull files and set executable
Git clone the repo to a spare space on your gitlab server (i.e. /media/data) to bring the scripts locally to the machine, then set the pre-receive and post-update files to be executable:

```chmod +x <repo>/pre-receive
chmod +x <repo>/post-update
```

## Set up hook prerequisites
Double check the Gemfile in the root of the repo to see which gems will be installed, and which versions, then run:

`bundle install`

## Set up environment
Create a directory for Git to set the default puppetlabs settings. By default git runs out of /var/opt/gitlab. After some testing I found it easier to simply create a .puppetlabs folder here than troubleshoot any further as to why it wasn't utilising the homedir I set up in /home :)

```mkdir /var/opt/gitlab/.puppetlabs
chown git /var/opt/gitlab/.puppetlabs
```

## Link files to target repo
Since hooks don't propagate between repos,  you must also copy or Symlink the './pre-receive' file to <target repo>/.git/hooks/pre-commit i.e.

`$ ln -s <git-hook-repo>/pre-commit <target repo>/.git/hooks/pre-commit`

You can set configuration options in commit_hooks/config.cfg

## Result
When you attempt to run a commit locally, it will refer to this file, which calls shell scripts to lint and check any changed files.
