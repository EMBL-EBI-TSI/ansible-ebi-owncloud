Owncloud
========
Install and configure ownCloud on a single server.

This role adds the official gitlab repository, but leaves it disabled.

Following ownCloud features are configured automatically through the role:
- MySQL is used as the default local database backend and can be tweaked by setting the `owncloud_mysql_` variables.
- Memcache using APCu for local cache.
- LDAP user authentication. All configuration is the `owncloud_user_ldap_config` hash. See below for configuration variables.
- Background jobs with the operating system's cron, running hourly. Email for cron jobs is directed to `owncloud_admin_email`.
- Mail is sent using sendmail, with `owncloud_admin_email` as the sender.
- Logging goes to ownCloud's default file path using the owncloud backend with a rotation size of 500Mb. See `owncloud_log_` variables for tweaking the logging features.
- Antivirus is activated using clamav. Note that not provisions are made to make sure clamav has run, specially ownCloud's default settings prevent it from scanning certain files (type, size,...). Note that you still need to manually activate the antivirus app from the app store. See below for antivirus configuration.
- ownCloud's main appstore is activated by default, but experimental apps are not enabled. See `owncloud_appstore_` variables for tweaks.
- Collaborative editing of documents is enabled with a local LibreOffice setup. Note that you still need to manually activate the documents app from the app store.

Multiple owncloud servers can be installed and configured. Only the first server in the list will be responsible for configuring the database and other options that depend on the `occ` command line tool.

A custom theme can be optionally deployed to the application servers from a git repository. Use `owncloud_theme_name` to specify the name of the folder inside the themes directory and `owncloud_theme_repo` to point to the git repository with the theme code.

Requirements
------------
See `meta/main.yml`.

Role Variables
--------------
See `defaults/main.yml`.

### LDAP configuration
All LDAP configuration is stored inside the `owncloud_user_ldap_config` hash, which is empty by default, therefore disabling LDAP. Any variable understood by `occ ldap:set-config` can be included in the hash. Because the need to retrieve values from `occ` to make the role idempotent and the difference in syntax between `occ config:app:get user_ldap` and `occ ldap:set-config`, each element in the hash has the following format:
```
get_var: [set_var, value]
```
where `get_var` is the variable name used by `occ config:app:get user_ldap` and `set_var` is the variable name used by `occ ldap:set-config`. For example:
```
owncloud_user_ldap_config:
  ldap_host: [ldapHost, 'ldaps://my.ldap.host.domain']
  ldap_port: [ldapPort, '636']
```

### Antivirus configuration
Antivirus configuration goes into the `owncloud_files_antivirus_config` hash. All options understood by `occ config:app:get files_antivirus` can be used. A common setup might be:
```
owncloud_files_antivirus_config:
  av_chunk_size: '1024'
  av_infected_action: 'only_log'
  av_mode: 'socket'
  av_path: '/usr/bin/clamscan'
  av_port: '0'
  av_socket: '/var/run/clamd.scan/clamd.sock'
```

Note that you still need to manually activate the antivirus app from the app store.

Dependencies
------------
None on this role, but a webserver, php and a database should be pre-installed before applying this role.

Example Playbook
----------------
Example:
```
- hosts: servers
  roles:
    - owncloud
```

TODO
----
- Select version to install.
- Keep mailto=root and set an alias for root?
- Better logic for running freshclam for the first time, or clamd sefrvice migh bail. Make a service? Or install cron packages.
- Move php dependencies to individual tasks?
- Update basic options (admin_pass, admin_email, ...).
- Activate calendar app?
- Abstract the list of servers, currently `owncloud-owncloud-servers` is used.

Licence
-------
Licensed under [CC-BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

Author Information
------------------
Luis Gracia <luis.gracia@ebi.ac.uk>
