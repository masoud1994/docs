# Install Gitlab  on Ubuntu

You can use this document for the following purposes.
* Install gitlab-ce and gitLab-ee 
* Reconfiguring to https
* Backup Gitlab
* Restore GitLab 
* Upgrade Gitlab

The following links were used to create this document
* https://about.gitlab.com/install/#ubuntu
* https://docs.gitlab.com/ee/administration/backup_restore


# Install Gitlab


1. ####  Resolve hostname on /ete/hosts.
```bash
echo "127.0.0.1 gitlab-host" >> /etc/hosts
```

2. #### Install and configure the necessary dependencies.
```bash
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates tzdata perl
```

 3. #### Add the GitLab package repository and install the package
you can add GitLab package repository for gitlab-ce and gitlab-ee
```bash
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
```
After executing the above command , itlab_gitlab-ce.list file will be placed in the following path
```bash
ls -l /etc/apt/sources.list.d/
```
```bash
sudo EXTERNAL_URL="https://gitlab.example.com" apt-get install gitlab-ce
```
List available versions: 
```bash
apt-cache madison gitlab-ce
```
Specifiy version:
```bash
sudo EXTERNAL_URL="https://gitlab.example.com" apt-get install gitlab-ce=16.2.3-ee.0
```
Pin the version to limit auto-updates:
```bash
sudo apt-mark hold gitlab-ee
```
Show what packages are held back: 
 
```bash
sudo apt-mark showhold
```
4. #### Browse to the hostname and login
Unless you provided a custom password during installation, a password will be randomly generated and stored for 24 hours in /etc/gitlab/initial_root_password. Use this password with username root to login.
``` bash
cat /etc/gitlab/initial_root_password
```

# Reconfiguring to https
Now we want To configure GitLab to use HTTPS, you’ll need to set up an SSL certificate and configure GitLab to use it. Here’s a step-by-step guide to set up HTTPS for your GitLab instance:

1. #### Obtain an SSL Certificate

You have two main options:

Use Let’s Encrypt (free, automated SSL): GitLab has built-in support for Let’s Encrypt, which makes obtaining and renewing certificates easy.

Use a custom SSL certificate: If you have a certificate from another provider or your own CA, you can configure GitLab to use it.


2. #### Configure GitLab for HTTPS

Edit the GitLab configuration file:
```bash
sudo nano /etc/gitlab/gitlab.rb
```

2.1 #### Option A: Set Up Let’s Encrypt (Free, Automated SSL)

Enable Let’s Encrypt in the configuration file:



Set your GitLab URL with HTTPS
```bash
external_url "https://gitlab.example.com"
```


Enable Let's Encrypt
```
letsencrypt['enable'] = true
```


Optional: Set a contact email for certificate renewal issues
```
letsencrypt['contact_emails'] = ['your-email@example.com']
```


Optional: Use HTTP to HTTPS redirection
```
nginx['redirect_http_to_https'] = true
```
Save and close the file.

Reconfigure GitLab to apply changes and request the Let’s Encrypt certificate:
```
sudo gitlab-ctl reconfigure
```

GitLab will automatically handle certificate renewal with Let’s Encrypt.

2.2 #### Option B: Use a Custom SSL Certificate

Place your SSL certificate and private key files on the server (e.g., in /etc/gitlab/ssl):
```
sudo mkdir -p /etc/gitlab/ssl
sudo cp your_certificate.crt /etc/gitlab/ssl/gitlab.example.com.crt
sudo cp your_private_key.key /etc/gitlab/ssl/gitlab.example.com.key
sudo chmod 600 /etc/gitlab/ssl/gitlab.example.com.*
```

Edit the GitLab configuration file to set the URL and point to your certificate files:

Set your GitLab URL with HTTPS
```
external_url "https://gitlab.example.com"
```

Specify SSL certificate and key paths
```
nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.example.com.crt"

nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.example.com.key"
```

Optional: Use HTTP to HTTPS redirection
```
nginx['redirect_http_to_https'] = true
```
Save and close the file.
Reconfigure GitLab to apply the SSL configuration:
```
sudo gitlab-ctl reconfigure
```


4. #### Test the HTTPS Configuration

	•	Open a web browser and go to https://gitlab.example.com.

	•	Ensure the certificate is valid, and the connection is secure.

5. #### Optional: Enforce HTTPS by Redirecting HTTP Traffic

To force all HTTP requests to redirect to HTTPS, ensure this line is set in gitlab.rb:

nginx['redirect_http_to_https'] = true
Then reconfigure GitLab once again:

```
sudo gitlab-ctl reconfigure
```

6. #### Verify the NGINX Configuration (Optional)

GitLab uses its own NGINX configuration by default. If you have other custom configurations, you may need to review the NGINX settings.

This setup should allow you to access GitLab securely over HTTPS.


# Backup gitlab 
Backing up GitLab involves creating a backup of repositories, configurations, and the entire database. GitLab provides a built-in backup feature to simplify this process. Here’s a step-by-step guide to perform a GitLab backup.

1. #### Set Up a Backup Directory
Choose a directory to store your GitLab backups. You can configure this in GitLab’s configuration file:
```bash
sudo nano /etc/gitlab/gitlab.rb
```
Locate and set the backup path:

gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"  # Change this to your desired path

Save and close the file, then reconfigure GitLab:
```bash
sudo gitlab-ctl reconfigure
```
2. ####  Create a Backup

GitLab provides a rake task to create a backup. Run this command:
```bash
sudo gitlab-backup create STRATEGY=copy
``` 
This will create a backup file in the specified backup_path. The backup will contain:

• Repositories
• Database
• Attachments
• CI/CD artifacts
• LFS objects (if enabled)

3. #### Automate Backups with Cron (Optional)

You can automate backups by setting up a cron job. Open the cron file:
```bash
sudo crontab -e
```
Add the following line to schedule a daily backup at 2:00 AM:
```bash
0 2 * * * /opt/gitlab/bin/gitlab-backup create CRON=1
```
4. Backup Configuration Files Separately

The built-in backup does not include the GitLab configuration files. To back up these files, copy them manually:
```bash
sudo cp /etc/gitlab/gitlab.rb /your-backup-location/
sudo cp /etc/gitlab/gitlab-secrets.json /your-backup-location/
```


Notes

• Database backup: If you use an external PostgreSQL database, back it up separately.
• Offsite storage: Consider storing backups offsite for added security.
• Encryption: Encrypt your backups if they contain sensitive data.

This should set you up with regular GitLab backups that you can easily restore if needed.

## Restore a GitLab
To restore a GitLab instance from a backup, you need a valid backup file that includes repositories, the database, attachments, and other necessary files. Here’s a step-by-step guide on how to restore GitLab from a backup:


1. #### Prepare for the Restore

	*	Ensure Compatibility: The backup should match the GitLab version you’re restoring to. Restore might fail if versions differ significantly.

	*	Stop GitLab Services: Stopping certain GitLab services is necessary to avoid conflicts during restoration.


```bash
sudo gitlab-ctl stop puma

sudo gitlab-ctl stop sidekiq

sudo gitlab-ctl stop nginx  # Optional, only if you face port conflicts
```

2. #### Move the Backup File

Move the backup file you want to restore to the backup directory defined in your gitlab.rb configuration. By default, it’s /var/opt/gitlab/backups.

For example, if your backup file is 1633444556_2023_11_01_gitlab_backup.tar, move it as follows:

```bash
sudo mv /path/to/your/backup/1633444556_2023_11_01_gitlab_backup.tar /var/opt/gitlab/backups/
```


3. #### Check Backup Permissions

Ensure that the backup file has the correct permissions, so GitLab can read it:

```bash
sudo chown git:git /var/opt/gitlab/backups/1633444556_2023_11_01_gitlab_backup.tar
```


4. #### Restore the Backup

To restore the backup, use the following command, replacing <timestamp> with the actual timestamp from the backup filename:

```bash
sudo gitlab-backup restore BACKUP=<timestamp>
```


For example:
```bash
sudo gitlab-backup restore BACKUP=1633444556_2023_11_01
```


GitLab will extract the backup and restore repositories, the database, and related assets.



5. #### Restore Configuration Files

The built-in GitLab restore command does not restore configuration files like gitlab.rb and gitlab-secrets.json. You need to restore these manually if needed:

```bash
sudo cp /path/to/backup/gitlab.rb /etc/gitlab/gitlab.rb
sudo cp /path/to/backup/gitlab-secrets.json /etc/gitlab/gitlab-secrets.json
```

Then, reconfigure GitLab to apply these configurations:
```bash
sudo gitlab-ctl reconfigure
```


6. #### Restart GitLab Services

Once restoration is complete, restart GitLab services:

```bash
sudo gitlab-ctl start
```
7. #### Verify the Restoration

	*	Log in to your GitLab instance and verify that all repositories, user accounts, and settings have been restored correctly.

	*	Check GitLab’s logs if there were any issues during the restore.

Notes

External Database: If GitLab uses an external PostgreSQL database, you may need to restore the database separately using PostgreSQL commands.

Test Restore: It’s a good idea to test your restore process periodically to ensure backups work correctly in a real scenario.

# Upgrade Gitlab
Upgrading GitLab involves a few steps to ensure data integrity and minimize downtime. Here’s a high-level guide for upgrading a GitLab instance, whether you’re using GitLab CE (Community Edition) or GitLab EE (Enterprise Edition). Always refer to the official GitLab upgrade documentation for the most up-to-date and detailed instructions.


#### Steps to Upgrade GitLab

1. #### Prepare a Backup


Before upgrading, it’s crucial to back up your GitLab instance to avoid any data loss. You can create a backup using this document:

2. #### Check GitLab’s Upgrade Path


GitLab has specific recommended upgrade paths for certain versions. Major versions often require incremental upgrades (e.g., from 13.x to 14.x to 15.x) instead of a direct jump. Check the GitLab version-specific upgrade paths if you’re skipping multiple versions.

Upgrade Path tool. To quickly calculate which upgrade stops are required based on your current and desired target GitLab version
```
https://gitlab-com.gitlab.io/support/toolbox/upgrade-path
```


3. #### Review the Release Notes

Review the release notes for the GitLab version you are upgrading to. This will inform you of any breaking changes, deprecated features, and important updates that may impact your environment.


4. #### Update GitLab via Package Manager

If you installed GitLab using a package manager like apt or yum, you can typically upgrade by running:

For Debian/Ubuntu:

#### Update the repository
```
sudo apt-get update
```


#### Install the new version
```
sudo apt-get install gitlab-ce
```


For CentOS/RHEL:

#### Clear the yum cache
```
sudo yum clean all
```


#### Install the new version
```
sudo yum install gitlab-ce
```


Alternatively, if you’re upgrading to a specific version, specify it directly:

```
sudo apt-get install gitlab-ce=<version>
```


5. #### Reconfigure GitLab

After installation, run the following command to apply the new configurations and changes:

```
sudo gitlab-ctl reconfigure
```


6. #### Run Migrations

GitLab might need to apply migrations to the database. Run:
```
sudo gitlab-ctl upgrade
```

This will ensure any necessary database and configuration migrations complete successfully.


7. #### Restart GitLab Services

Restart GitLab to make sure everything is running with the updated version:
```
sudo gitlab-ctl restart
```


8. #### Verify the Upgrade

Check the GitLab version to verify it has been upgraded successfully:
```
gitlab-rake gitlab:env:info
```


9. #### Test the Instance

Finally, log in to your GitLab instance and test key features to ensure everything is functioning correctly.

Additional Tips
* Downtime Planning: Some upgrades might require a short downtime, especially for larger instances.
* Upgrade Dependencies: Make sure system dependencies, such as PostgreSQL, Redis, or Git, are compatible with the new GitLab version.
* Rollback Plan: In case of issues, have a rollback plan ready, including your backup and any necessary documentation.



This guide covers a typical upgrade; however, more advanced scenarios may involve Docker, Kubernetes, or Omnibus GitLab packages. Let me know if you have a specific setup, and I can provide more tailored instructions.