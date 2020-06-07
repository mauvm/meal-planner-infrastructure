# Event Store

The app uses an [EventStoreDB](https://eventstore.com/eventstoredb/) instance for persistance through event sourcing.

## Backups

Prerequisites:

- [crond](https://linux.die.net/man/8/crond)
- [rsync](https://linux.die.net/man/1/rsync)

Add a cronjob for daily backups:

```bash
cat <<EOF | sudo tee /etc/cron.daily/backup-event-store.sh
#!/bin/sh
rsync -a /mnt/event-store/data/*.chk /backup/event-store/
rsync -a /mnt/event-store/data/index /backup/event-store/
rsync -a /mnt/event-store/data/*.0* /backup/event-store/
exit 0
EOF

sudo mkdir -p /backup/event-store/
sudo chmod 700 /backup /backup/event-store
sudo chmod +x /etc/cron.daily/backup-event-store.sh
```

See [EventStore Database backup](https://eventstore.com/docs/server/database-backup/index.html) for more information (including restoring the database).
