# ansible-role-drupal-cron
Set up a linux cron job to trigger Drupal Cron on a locally installed site.

This role isn't usually added after a site is up and running, since it requires the cron URL, which won't exist until after the site has been configured.

## Dependencies / Requirements
If specifying `drupal_cron_url`, cURL already needs to be installed on the server.

If specifying `drupal_cron_doc_root` + `drupal_cron_drush_path`, Drush needs to already be installed on the server.

## Role Variables

### `linux_owner` - required
The name of the linux account to create the cron job under.

### `server_name` - reuired
The domain name part of your drupal cron URL. This should match the "server_name" variable of your Apache or NGINX virtual host. It's used by ansible to tag the job's block in the crontab file. It's also used passed as the '--uri' variable to drush, when using

### `drupal_cron_(url|doc_root,drush_path|command)` (string):

Specify either:

 - #### drupal_cron_url (string)

    When this variable is used, the role will create a cURL command as the cron job that looks like:

    ```
    /usr/bin/curl -kSsL '{{ drupal_cron_url }}' > /dev/null
    ```

    Copy the cron URL from your drupal installation. It will look like
    ```
    https://myservername.com/cron/Jawefijawifaw3oij3oiuh2oui3hoiu2giuyg
    ```

or

- #### drupal_cron_doc_root (string) + drupal_cron_drush_path (string)

   When this pair is used, the role will create a drush command as the cron job that looks like:

   ```
   /usr/bin/env PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin COLUMNS=72 {{ drupal_cron_drush_path }} --uri={{ server_name }} --root='{{ drupal_cron_doc_root }}' --quiet cron
   ```

   drupal_cron_doc_root = the web root / base of your drupal install

   drupal_cron_drush_path = relative path from your doc root to the drush binary, or an absolute path to the global drush binary

or

 - #### drupal_cron_command

   If neither of the above two constructors provide what you need, you can specify your own shell command to execute in the cron job.

## Role defaults

### `every_x_minutes` (integer) = 15
How often you want your cron job to run.

For more granularity, you can specify `drupal_cron_hour` and `drupal_cron_minute` instead.

See also defaults/main.yml for undocumented defaults.



## Example

* **Pro tip no. 1:** Don't run the role against more than one node of a multi-server application.

* **Pro tip no. 2:** Don't re-use the same role twice in the same play. Split up role instances into individual plays to avoid variable bleed.

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
