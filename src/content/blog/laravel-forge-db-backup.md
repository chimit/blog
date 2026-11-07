---
title: "How to set up free Database Backups on Laravel Forge"
author: Chimit
pubDatetime: 2025-10-27T21:00:00Z
modDatetime: 2026-11-07T07:00:00Z
featured: false
draft: false
tags:
  - Laravel Forge
  - Backup
  - MySQL
  - S3
  - Cloudflare R2
  - Netcup
description: "Simple and free database backup solution for Laravel Forge using Cloudflare R2 with heartbeats monitoring, automated cleanup and non-critical tables exclusion."
---

Recently, I moved my websites from dearly loved Hetzner to [Netcup](https://www.netcup.com/en/?ref=319497) and realized that unlike Hetzner, Netcup doesn't offer daily backups. It brought me back in memory to 2010th when I spent hours configuring [Backupninja](https://0xacab.org/liberate/backupninja) and [Duplicity](https://duplicity.gitlab.io) to backup my servers. They are still great tools, but I wanted something simpler, easier to manage, but still completely free.

Since I'm a big fan of [Laravel Forge](https://forge.laravel.com) and use it to manage all my servers, I decided to leverage its features to create a simple backup solution for my databases.

> Laravel Forge offers a built-in backup feature, but it's available only on a Business plan, which is quite expensive for my needs.

My goal was to create a backup solution that is:

1. **Free** - no additional costs at all
2. **Reliable** - stores backups offsite on S3-compatible storage
3. **Configurable** - allows me to choose which databases and tables to backup
4. **Easy to set up and manage** completely via Laravel Forge UI
5. **Automated** - runs on a schedule, automatically cleans up old backups

First of all, I needed a free offsite storage solution and [Cloudflare R2](https://www.cloudflare.com/r2/) is exactly it. It offers a generous free tier with 10 GB of storage, which is more than enough for my needs. A great point is that it's S3-compatible, so I can switch to another S3-compatible service in the future if needed.

Another critical point for me was the ability to exclude specific tables from the backup. For example, I have some large tables with search indexes that I don't need to backup, as they can be easily regenerated. For one of my sites, excluding those tables reduced the backup size from 4.1 GB to 2.8 GB! It makes the process significantly faster and saves storage space.

I created a simple bash script, but wanted to avoid the hassle of managing it via SSH. Laravel Forge provides a great way to manage environment variables, schedule tasks, and monitor heartbeats without needing to SSH into the server. So I decided to set it up as a Laravel Forge website to use all those features.

### Installation

Fork the [chimit/forge-db-backup](https://github.com/chimit/forge-db-backup) repository to your GitHub account and create a new PHP site from it in Laravel Forge:

- Don't connect any database;
- Don't install Composer dependencies;

<img src="/assets/laravel_forge_setup_php_application.png" width="500" alt="Laravel Forge - create PHP application">

- In **Advanced settings** make sure the **Web directory** is set to `/public`;
- Disable **Zero downtime deployments**.

<img src="/assets/laravel_forge_setup_php_application2.png" width="500" alt="Laravel Forge - create PHP application">

### Configuration

Now you need to configure the script via environment variables (**Settings** -> **Environment**). Put your MySQL server credentials and specify which databases to backup, which tables to ignore and how many backups to keep.

If you use **Laravel Pulse**, I would recommend to exclude the `pulse_entries` and `pulse_aggregates` tables as they can grow significantly over time and are not needed in the backup. The same applies to some [other popular Laravel packages](https://github.com/chimit/forge-db-backup?tab=readme-ov-file#ignoring-tables).

```shell
# Database configuration
DB_USERNAME="your-database-user"
DB_PASSWORD="your-database-password"

# Comma-separated list of databases to backup
DB_DATABASES="database1,database2"

# Tables to ignore per database (optional)
# Format: database.table1,database.table2,another_db.table1
# Use comma to separate database.table pairs
DB_IGNORE_TABLES="database1.search_index,database2.pulse_entries,database2.pulse_aggregates"

# Number of backups to keep per database.
# With a bucket lock, set to lock days + 1 (see below)
KEEP_COUNT=11
```

If you chose Cloudflare R2 as your storage, you need to create a bucket and an **S3 API token** with the **Object Read & Write** permission for that bucket. Settings should look familiar if you have used S3 in Laravel before:

```shell
# S3 storage configuration
AWS_ACCESS_KEY_ID="your-access-key-id"
AWS_SECRET_ACCESS_KEY="your-secret-access-key"
AWS_ENDPOINT="https://your-s3-endpoint.com"
AWS_BUCKET="your-bucket-name"
```

### Protecting backups from deletion

Some time after I set this up, one thought started to bother me. The script cleans up old backups, which means the key sitting in `.env` can delete anything in the bucket. Now imagine someone breaks into the server. Before encrypting or dropping your databases, a smart attacker gets rid of the backups first, and here the key to do it is lying right next to them.

Luckily, Cloudflare R2 has bucket locks. A lock rule says that nobody can delete or overwrite an object for N days after upload, and it doesn't matter which key they use. I went with 10 days: open the bucket **Settings**, find **Bucket lock rules** and add a rule with an empty prefix, so it covers the whole bucket.

<img src="/assets/r2_bucket_lock.png" width="400" alt="Cloudflare R2 - bucket lock rule">

There are two small catches.

First, the lock is only as strong as the token on your server. An Admin token can simply remove the rule, so stick to **Object Read & Write** limited to the backup bucket.

Second, set `KEEP_COUNT` one higher than the lock period, so `11` in my case. The lock counts each backup's age from the moment it was uploaded, and the script removes the oldest backup right after uploading a fresh one. With `KEEP_COUNT=10` that oldest backup is almost exactly 10 days old. If today's dump happened to finish a few seconds faster, it's still locked and R2 refuses to delete it. Nothing terrible happens: the script prints a warning and cleans it up the next night. One extra backup just keeps the output clean.

### Scheduled job

Go to **Processes** -> **Scheduler** tab of your Forge site and create a new scheduled task. Give it any name you like, e.g. `DB backup` and put the full path to the `backup.sh` script as the command (`/home/forge/{your-site}/backup.sh`). Choose desired frequency (e.g., nightly) and optionally enable **Monitor with heartbeats**.

<img src="/assets/laravel_forge_scheduled_job.png" width="500" alt="Laravel Forge - create scheduled job">

### Heartbeat monitoring

It's always a good idea to monitor your backups to ensure they run successfully. If you enabled heartbeat monitoring in the scheduled job, Laravel Forge will provide you with a unique heartbeat URL. Put that URL into the `HEARTBEAT_URL` environment variable. If the backup script doesn't complete successfully, you will get notified.

<img src="/assets/laravel_forge_heartbeat.png" width="300" alt="Laravel Forge - create scheduled job">

All done! Now let's make sure everything works by running the backup script manually - click **Run** in the dropdown next to your scheduled job. After a few seconds or minutes (depending on the size of your databases), your backups should appear in the `backups` folder of your website and in your S3 bucket.

In the command output, you should see something like this:

```
Starting database backup...

Processing database1...
  Ignoring tables: search_index
  ✓ Backup created successfully: database1_2025-10-27_00-00-01.sql.gz
  ✓ Deleted old local backup: database1_2025-10-16_00-00-01.sql.gz
  ✓ Uploaded to S3 (700.27 MB)
  ✓ Deleted old S3 backup: database1_2025-10-16_00-00-01.sql.gz

Processing database2...
  Ignoring tables: pulse_entries, pulse_aggregates
  ✓ Backup created successfully: database2_2025-10-27_00-00-01.sql.gz
  ✓ Uploaded to S3 (19.02 MB)

✓ All backups completed successfully on Mon, Oct 27, 2025 at 00:05:13 (5m 12s)
```
