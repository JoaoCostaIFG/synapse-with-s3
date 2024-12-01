# My Synapse server container

Why this container:

- It includes [synapse-s3-storage-provider](https://github.com/matrix-org/synapse-s3-storage-provider)
- Includes a wrapper script for `s3_media_upload`:
  - The main reason for this script is so I can automate tasks by reading the homeserver's config file (e.g. get the bucket key, uri, etc.).
  - A cronjob runs daily deleting files from the media store that haven't been loaded for a while, and saving them to S3 (in tmy case R2).
  - If someone scrolls back to where these files are, they are loaded back.

## Config/Run

You just gotta make sure that your homeserver config (e.g. `/data/homeserver.yaml`) has the [synapse-s3-storage-provider](https://github.com/matrix-org/synapse-s3-storage-provider) configured with at least the following keys:

- `bucket`
- `endpoint_url`
- `access_key_id`
- `secret_access_key`

You can find an example script to run as a daily cronjob [here](./matrix_cleanup.cron.sh). This scripts offloads files that haven't been loaded in at least 7 days.

## Example config

Place something like this in your `home-server.yaml`:

```yaml
media_store_path: /data/media_store
media_retention:
  remote_media_lifetime: 2d
# S3 like storage
media_storage_providers:
  - module: s3_storage_provider.S3StorageProviderBackend
    store_local: True
    store_remote: True
    store_synchronous: True
    config:
      bucket: bucket_name
      endpoint_url: url
      access_key_id: key_id
      secret_access_key: key_secret
```

This will keep data locally for 2 days, and then offload it to S3 when no one has touched it for at least 2 days. If someone tries to access this data afterwards, the server downloads the asset from S3 and keeps it for 2 days again.

## Compose

Included an [example compose file](./docker-compose.yml) that sets up synapse, element-web, coturn (for calls), and synapse admin.
