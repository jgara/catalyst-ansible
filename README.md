# catalyst-ansible

Ansible project to build and deploy [Catalyst](https://github.com/jhu-sheridan-libraries/blacklight-rails), Johns Hopkins University Libraries blacklight-based library catalogue.

## WIP

## Setup SSH keys

Here using setting up stage as an example. For other environment, revise accordingly.

Update ansible.cfg and comment out remote_user at the top, and uncomment the
section at the bottom that starts with `# first-run vars for setup.yml:`

ansible-playbook setup.yml -i inventory/stage -v

Change it back afterwards

Update ~/.ssh/config, rename/or add catalyst-stage entry

Then find the ssh passphrase in vault

```
ansible-vault view inventory/group_vars/stage/vault.yml | grep 'vault_login_user_passphrase'
```

Copy the passphrase

ssh to `catalyst-stage.library.jhu.edu` and `catsolrslave-test`. You may need the passphrase that was copied above.

Then you can run the deployment commands below.

## deployment

### Server Component Upgrades

#### Upgrade Ruby
We assume that the application has been tested on dev, and either  test/stage.

Follow the following checklist 
* Update the production inventory, choose ruby version, and app_branch
* Check you have ssh deploy@catalyst.library.jhu.edu works
* Check free space on server (df -h ) | require at least 2G
* Run the catalyst.yml playbook on production
```
time OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES ansible-playbook -i inventory/prod playbooks/catalyst.yml  --limit=catalyst -v
```
* This takes 13m to run, 4m of downtime
* Keep an eye on the logs for errors
* Run the cucumber test locally

Note: Fallback plan you can pass in the ruby, and app version to revert.
```
#Role Back
time OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES ansible-playbook -i inventory/prod playbooks/catalyst.yml   --extra-vars="app_branch=v1.1.4"  --extra-vars="chruby_ruby_version=ruby-2.2.2" --limit=catalyst -v  # 2m23s
```

### Deploy the web service 

* To catalyst-prod.library.jhu.edu

```
ansible-playbook playbooks/horizonws_install.yml -i inventory/prod -v
```
