paulrentschler.toolshed
=======================

[![MIT licensed][mit-badge]][mit-link]

Installs and configures the ToolShed scripts for data backups


Requirements
------------

None.


Role Variables
--------------

The following variables are available with defaults defined in `defaults/main.yml`:

### General settings

Where to install ToolShed.

    toolshed_install_dir: /opt/toolshed

Where the shell scripts to run the backups go.

    toolshed_scripts_dir: "{{ toolshed_install_dir }}/scripts"

Which version (branch) of ToolShed to install.

    toolshed_repo_branch: "master"


### Database backup settings

Change to `yes` in order to cause a database server to be backed up.

    toolshed_backup_db: no

Specify the type of database server to be backed up. *(currently only MySQL is supported)*

    toolshed_backup_db_type: "mysql"

Primary host (particularly in a replication cluster) to use when creating the backup user in the database. Defaults to the current host but would need to be changes if the current host is a replica and not the primary in a cluster.

    toolshed_backup_db_primary_host: "{{ ansible_hostname }}"

Username used to connect to the database.

    toolshed_backup_db_user: ""

Password used to connect to the database.

    toolshed_backup_db_password: ""

MySQL 5.6+ supports the use of the `mysql_config_editor` command ([docs](https://dev.mysql.com/doc/refman/5.6/en/mysql-config-editor.html)) to create a credentials file that does not store passwords in the clear. Use the `mysql_config_editor` command as follows to generate a .mylogin.cnf file:

    > mysql_config_editor set --login-path=<toolshed_backup_db_login_path>
                              --host=localhost
                              --user=<toolshed_backup_db_user>
                              --password
    Enter password: <enter toolshed_backup_db_password here>

Copy the .mylogin.cnf file into the "files" directory and then specify the filename with the `toolshed_backup_db_login_file` variable.

    toolshed_backup_db_login_file: "../files/mylogin-prod-datareplica.cnf"

You will also need to specify the login path used in the `mysql_config_editor` command above:

    toolshed_backup_db_login_path: ""


List of databases and/or tables to include in the backup. If not specified all databases and tables will be backed up unless excluded by `toolshed_backup_db_exclude`.

The format here is: `<database>.<table>`

To include/exclude an entire database, use `<database>.*`

    toolshed_backup_db_include: []

List of databases and/or tables to specifically exclude from being backed up regardless of whether or not it is included in `toolshed_backup_db_include`.

The format is the same as for `toolshed_backup_db_include`.

    toolshed_backup_db_exclude: []

Name of the shell script that will be created to run the backup (no extensions).

    toolshed_backup_db_script: "dbbackup"

Path of the directory that will hold the database backups.

    toolshed_backup_db_backup_dir: /backup/database

Email address of the GPG Encryption key to encrypt the backups with. If blank, the backup will **not** be encrypted.

    toolshed_backup_db_encryption_key: ""

The Owner and/or group assigned to the backup files.

    toolshed_backup_db_owner: ""
    toolshed_backup_db_group: ""

When to run the backup. These settings map to values used by Cron. Defaults to running the backup every day at 5am.

*(If using a chronological hierarchy of retained backups, it is **highly** recommended that the backup run every day and not just weekdays)*

```yaml
toolshed_backup_db_schedule_mailto: "root"
toolshed_backup_db_schedule_hour: 5
toolshed_backup_db_schedule_min: 0
toolshed_backup_db_schedule_month: "*"
toolshed_backup_db_schedule_day: "*"
toolshed_backup_db_schedule_days: "*"
toolshed_backup_db_schedule_user: "root"
```

Optionally disable the cron job (i.e., comment it out) by setting this to "no". The default is "yes".

    toolshed_backup_db_schedule_enable: yes

The structure of backup files over time and how many backups are kept are controlled by the settings below. If all values remain zero (0) then an infinite number of backups will be kept in a single directory until the file system is full.

To use a single directory and keep X number of timestamped backups, specify the number for the limit variable:

    toolshed_backup_db_limit: 0

To use a hierarchy of chronological directories with potentially different numbers of backups kept at each level (or even skipping levels), use the settings below (level values that remain zero will be skipped):

    toolshed_backup_db_daily: 0
    toolshed_backup_db_weekly: 0
    toolshed_backup_db_monthly: 0
    toolshed_backup_db_yearly: 0

An example of keeping 2 weeks (14 days) of daily backups, 6 weeks of weekly backups (Saturday is the end of the week and what is kept), 4 months worth of monthly backups (the backup on the last day of the month is kept), and 2 years worth of yearly backups (12/31 is the backup kept).

    toolshed_backup_db_daily: 14
    toolshed_backup_db_weekly: 6
    toolshed_backup_db_monthly: 4
    toolshed_backup_db_yearly: 2


#### GPG Encryption

While backups can be setup to be encrypted with a GPG key, this role **does not** install the GPG key that is specified. Setting up the GPG key must be done **manually** as follows:

Export the public key to use to encrypt the backups:

    > gpg --list-keys
    > gpg --export <key id> > public.key

Copy the `public.key` file to the server that will be doing the backups.

Install the public key on the server doing the backups:

    > gpg --import public.key
    > gpg --list-keys
    > gpg --edit-key <key id> trust quit

    Enter 5 to trust ultimately

    Enter Y to confirm

The public key is now installed and trusted to be used to encrypt the database backups.



### Plone backups

Change to `yes` in order to cause Plone installation to be backed up.

    toolshed_backup_plone: no

Name of the shell script that will be created to run the backup (no extensions).

    toolshed_backup_plone_script: "plonebackup"

Path to the zeocluster directory in the Plone instance to back up.

    toolshed_backup_plone_install_dir: /opt/current-plone/zeocluster

Path of the directory that will hold the Plone backups.

    toolshed_backup_plone_backup_dir: /backup/plone

Change to `yes` to combine the data.fs and blob storage into a single backup file. By default they are backed up as separate files.

    toolshed_backup_plone_combine_files: no

The Owner and/or group assigned to the backup files.

    toolshed_backup_plone_owner: ""
    toolshed_backup_plone_group: ""

Specify the number of days of transactions to retain when packing the Plone ZODB. Zero (0) will disable packing the ZODB.

    toolshed_backup_plone_pack_days: 0

When to run the backup. These settings map to values used by Cron. Defaults to running the backup every day at 5am.

*(If using a chronological hierarchy of retained backups, it is **highly** recommended that the backup run every day and not just weekdays)*

```yaml
toolshed_backup_plone_schedule_mailto: "root"
toolshed_backup_plone_schedule_hour: 5
toolshed_backup_plone_schedule_min: 0
toolshed_backup_plone_schedule_month: "*"
toolshed_backup_plone_schedule_day: "*"
toolshed_backup_plone_schedule_days: "*"
toolshed_backup_plone_schedule_user: "root"
```

Optionally disable the cron job (i.e., comment it out) by setting this to "no". The default is "yes".

    toolshed_backup_plone_schedule_enable: yes

The structure of backup files over time and how many backups are kept are controlled by the settings below. If all values remain zero (0) then an infinite number of backups will be kept in a single directory until the file system is full.

To use a single directory and keep X number of timestamped backups, specify the number for the limit variable:

    toolshed_backup_plone_limit: 0

To use a hierarchy of chronological directories with potentially different numbers of backups kept at each level (or even skipping levels), use the settings below (level values that remain zero will be skipped):

    toolshed_backup_plone_daily: 0
    toolshed_backup_plone_weekly: 0
    toolshed_backup_plone_monthly: 0
    toolshed_backup_plone_yearly: 0

An example of keeping 5 weeks (35 days) of daily backups and 6 months worth of monthly backups (the backup on the last day of the month is kept).

    toolshed_backup_plone_daily: 35
    toolshed_backup_plone_weekly: 0
    toolshed_backup_plone_monthly: 6
    toolshed_backup_plone_yearly: 0

**Note:** if using the chronological hierarchy of retained backups, things will be simplier if the Data.fs and blob storage backups are combined into a single backup file by using `toolshed_backup_plone_combine_files: yes`.


Dependencies
------------

None.


Example Playbook
----------------

Example of backing up a Plone 4 and Plone 5 instance on the same host and keeping 5 copies of each backup. The Plone 4 instance has its ZODB packed with 7 days worth of transactions and the Plone 5 instance does not have its ZODB packed.

```yaml
---
- hosts: webserver
  roles:
    - role: paulrentschler.toolshed
      vars:
        toolshed_backup_plone: yes
        toolshed_backup_plone_script: "plone4backup"
        toolshed_backup_plone_install_dir: /opt/plone-4.3/zeocluster
        toolshed_backup_plone_backup_dir: /backup/plone/4
        toolshed_backup_plone_pack_days: 7
        toolshed_backup_plone_schedule_hour: 2
        toolshed_backup_plone_limit: 5
    - role: paulrentschler.toolshed
      vars:
        toolshed_backup_plone: yes
        toolshed_backup_plone_script: "plone5backup"
        toolshed_backup_plone_install_dir: /opt/plone-5.1/zeocluster
        toolshed_backup_plone_backup_dir: /backup/plone/5
        toolshed_backup_plone_pack_days: 0
        toolshed_backup_plone_schedule_hour: 3
        toolshed_backup_plone_limit: 5
```

Example of backing up a MySQL database at 7:30pm with only daily and monthly backups kept and encrypts the backups with the GPG key associated with dbbackup@example.com. Also excludes the `users` and `hosts` tables in the `mysql` database.

```yaml
---
- hosts: db_replica
  roles:
    - role: paulrentschler.toolshed
      vars:
        toolshed_backup_db: yes
        toolshed_backup_db_exclude:
          - mysql.hosts
          - mysql.users
        toolshed_backup_db_encryption_key: "dbbackup@example.com"
        toolshed_backup_db_schedule_hour: 19
        toolshed_backup_db_schedule_min: 30
        toolshed_backup_db_daily: 0
        toolshed_backup_db_weekly: 0
        toolshed_backup_db_monthly: 0
        toolshed_backup_db_yearly: 0
```


License
-------

[MIT][mit-link]


Author Information
------------------

Created by Paul Rentschler in 2021.


[mit-badge]: https://img.shields.io/badge/license-MIT-blue.svg
[mit-link]: https://github.com/paulrentschler/ansible-role-toolshed/blob/master/LICENSE
