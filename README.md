![Python version](https://img.shields.io/badge/python-3-blue.svg)

# azure-blob-container-backup

This is a script to backup Microsoft Azure blob containers.

## What this does

This script takes in any number of Azure blob storage containers and
creates backups of them under a destination Azure storage account. The
backup containers are named using a timestamp (YYYMMDDHHMM) so you can
identify and parse your backups easily.

## What this doesn't do

+ This doesn't schedule backups for your containers (use cron for that)
+ This doesn't clean up old backups for you
+ This doesn't do any fancy versioning (e.g., by comparing file
  modification dates and only pulling in what has changed)—this creates
  a complete copy of each container passed in every time

## Usage

This only works on Unix-like systems. You're going to need AzCopy for
this to work. As of this writing, the easiest way to install this is to
follow [Microsoft's installation instructions
here](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-linux).

Once that's done, copy [`config.yaml.example`](config.yaml.example) to
`config.yaml` and fill in the latter with appropriate values, keeping it
in the same directory as the main script
([`container_backup.py`](container_backup.py)).

Then run the script with

```
./container_backup.py
```

or [schedule it with
cron](https://fossbytes.com/how-to-schedule-jobs-in-linux-cron-crontab/).

## Details

### Potentially very important

If you're backing up **big** containers I'd recommend running this
script on an Azure VM in the same region that one or more of the storage
accounts are in. This is faster and cheaper than if they're in separate
regions.

### Somewhat important

AzCopy doesn't like being parallelized. In my experience, it has
complained (fatally) every time I've tried to run more than one instance
of it. The consequences of this are that if you're running **long**
jobs, you're going to need them to finish before starting any other
job. This means you're going to need to schedule intelligently (cf.
blindly) if you're using this to schedule regular backups.

### Probably not very important at all

Container names need to be unique within a given storage account.
Ideally, a naming scheme that uniquely labels each backup container
would look something like

```
{timestamp}{some indicator this container is a backup}{source storage account name}{source storage container name}
```

The problem with this approach is that container names can be, as of
this writing, at most 63 characters.

My solution to this problem was to take the first 63 characters of the
above scheme and use that as the backup container name. This is fine for
almost all cases; however, at worst this can result in non-unique
container names, which is no good. To remedy this, when a non-uniqueness
condition occurs, the script inserts an extra element `{count}` into the
above scheme after the `{backup indicator}`, and keeps incrementing
count until it gets a unique name, which is nearly guaranteed to occur.
