# ansible-role-drupal-cron
Set up a linux cron job to trigger Drupal Cron on a locally installed site.

This role does two things:
- Creates an entry in your /etc/hosts file to maps your site's host name to 127.0.0.1
- Creates a linux cron job that hits your drupal installation's cron url with cURL

The /etc/hosts entry is just to avoid the cURL request from trying to hit your site with its public IP address, which typically won't work from behind firewalls.

This role is usually added after a site is up and running, since it requires the cron URL, which won't exist until after the site configuration has completed.

## Dependencies
The cron job created by this role uses the cURL command line utility. Curl must already be installed on the server.

## Role Variables

### linux_owner
The name of the linux account to create the cron job under.

### server_name (string)
The domain name part of your drupal cron URL. This should match the "server_name" variable of your Apache or NGINX virtual host. It's used to place an entry into your /etc/hosts file.

### drupal_cron_url (string)
Copy this from your drupal installation. It will look like https://myservername.com/cron/Jawefijawifaw3oij3oiuh2oui3hoiu2giuyg

## Role defaults

### every_x_minutes (integer) = 15
How often you want cURL to hit your drupal cron page.


## Example playbook
```yaml
---
- hosts: my-primary-app-node   # Pro-tip: Don't run drupal cron against more than one node of a multi-server application.
  become: true
  gather_facts: true
  roles:
   - role: acromedia.drupal-cron
     linux_owner: bigcorp
     server_name: www.bigcorp.com
     drupal_cron_url: 'https://{{ server_name }}/cron.php?cron_key=abcdefgh12345678'
     every_x_minutes: 5
     tags:
       - cron
```

## License
GPL3

## Author Information
Acro Media Inc., https://www.acromedia.com
