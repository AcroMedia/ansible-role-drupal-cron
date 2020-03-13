# ansible-role-drupal-cron
Set up a linux cron job to trigger Drupal Cron on a locally installed site.

This role isn't usually added after a site is up and running, since it requires the cron URL, which won't exist until after the site has been configured.

## Dependencies / Requirements
If specifying `drupal_cron_url`, cURL already needs to be installed on the server.

If specifying `drupal_cron_doc_root` + `drupal_cron_drush_path`, Drush needs to already be installed on the server.

## Role Variables

### **drupal_cron_job_name** - required

Technically, this is optional for backwards compatibility, but this really needs to be **carefully defined, and then never changed again**. It simply needs to be unique string per environment, since ansible uses it as an anchor to find and edit the correct job on the server.

The original default for this was based on the value of `server_name` to avoid creating duplicate jobs on servers with multiple vhosts. The problem with that approach is that `server_name` may change across the lifespan of a website, which ends up creeating duplicate jobs. (see issue [#1](https://github.com/AcroMedia/ansible-role-drupal-cron/issues/1)).

### **linux_owner** - required
The name of the linux account to create the cron job under.

Be aware of permissions implications:

- For curl mode (see below), the linux owner only needs to be able to execute curl commands.

- For drush or custom modes (see below), **linux_owner** must be able to modify the files that PHP created. E.g. if you run PHP FPM, and your PHP process runs as its own user, that's who you need create the job under. If your PHP files are created by the www-data or apache user, that's who you need to create the cron job under.

### **server_name** - required
The fully qualified domain name part of your drupal cron URL (e.g. 'www.example.com', without any protocol or slashes). This should match the "server_name" variable of your Apache or NGINX virtual host.

## Cron modes:

The role suppports 3 different modes for setting up a cron job. The variables you set automatically determine the mode.

- **cURL mode** (best for Drupal <= 7)

  ```yaml
  drupal_cron_job_name: Drupal cron for bigcorp production
  drupal_cron_url: 'https://www.example.com/cron/abcdefgh12345678'
  ```
  - When cron url is specified, the role creates a cURL command cron job.
  - The command includes cURL's `--resolve` switch, which maps `{{ server_name }}` to `127.0.0.1`. This is so cURL doens't try and hit a server's public interface from behind a firewall, which would normally time out.


- **Drush CLI mode** (best for Drupal >= 8)

  ```yaml
  drupal_cron_job_name: Drupal cron for bigcorp production
  drupal_cron_doc_root: '/var/bigcorp/www/ecomsite/web'
  linux_owner: ecom-srv
  drupal_cron_drush_path: '../vendor/drush/drush/drush'
  ```

  - When doc root + drush path are specified, the role constructs a drush CLI command cron job. The value for `drupal_cron_doc_root` must be the absolute path to directory you'd normally execute Drush commands from the command line. The value for `drupal_cron_drush_path` can either be relative to the doc root, or an absolute path.


- **Custom mode**

  If neither of the above two constructors provide what you need, you can specify your own arbitrary shell command to execute in the cron job:

  ```yaml
  drupal_cron_job_name: Drupal cron for bigcorp production
  linux_owner: someuser
  drupal_cron_command: >
    cd /path/to/someplace && /usr/local/bin/custom-stuff.sh --foo-bar
  ```

## Changing the value of `server_name`

If your playbook already uses `drupal_cron_job_name`, you're free to change `server_name` without any impact.

If you haven't yet defined `drupal_cron_job_name`, then simply look up the current value of the job's name (`sudo crontab -l -u <linux_user>`), which will look something like this:
```
#Ansible: Drupal cron trigger for www.example.com
          \^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^/ this is the job name
```
and add `drupal_cron_job_name: <current job name>` to your playbook.

Even though the job name will be 'wrong' after your `server_name` changes, the job will not be duplicated.

Obviously, you can also just delete the current job and let the playbook create a new one, but if your app is spread across a lot of servers, that could be tedious.

## Role defaults


### `every_x_minutes` (integer) = 15
How often you want your cron job to run.

For more granularity, you can specify `drupal_cron_hour` and `drupal_cron_minute` instead.

**See also**: defaults/main.yml for undocumented defaults.



## Example

* **Pro tip no. 1:** Avoid running the role against more than one node of a multi-server application.

* **Pro tip no. 2:** Don't ever re-use any ansible role twice in the same play. Split up role instances into individual plays to avoid variable bleed.

```yaml
---
- name: Gather facts as its own play. Subsequent plays against
        the same server don't need to gather facts.
  hosts: my-primary-app-node   
  become: true
  gather_facts: true
  roles: []
  tasks: []

- hosts: my-primary-app-node   
  become: true
  gather_facts: false
  roles:
   - name: Set up Drupal cron for site no. 1 - cURL mode
     role: acromedia.drupal-cron
     vars:
       linux_owner: blog1
       server_name: blog.bigcorp.com
       drupal_cron_job_name: Drupal cron for bigcorp blog production
       drupal_cron_url: 'https://{{ server_name }}/cron.php?cron_key=abcdefgh12345678'
       every_x_minutes: 5
     tags:
       - cron
  tasks: []

- hosts: my-primary-app-node   
  become: true
  gather_facts: false
  roles:
   - name: Set up Drupal cron for site no. 2 - Drush mode
     role: acromedia.drupal-cron
     vars:
       linux_owner: www-data
       drupal_cron_job_name: Drupal cron for bigcorp shop production
       server_name: shop.bigcorp.com
       drupal_cron_doc_root: /var/www/vhosts/shop1/web
       drupal_cron_drush_path: ../vendor/drush/drush/drush
     tags:
       - cron
  tasks: []
```

## License
GPL3

## Author Information
Acro Media Inc., https://www.acromedia.com
