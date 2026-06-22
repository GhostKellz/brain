---
type: reference
title: "Cloud Backup Storage"
created: 2026-06-21
updated: 2026-06-21
tags:
  - backup
  - s3
  - backblaze
  - wasabi
status: developing
related:
  - "[[Linux Administration]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Object-storage backup targets (Backblaze B2, Wasabi). These are S3-compatible buckets used as backup destinations.

> [!warning] Access keys, secret keys, key IDs, and application keys are SECRETS. Never store them in notes or a public wiki — keep them in a password manager / secrets vault and reference them by name only.

## Credential Shape (placeholders only)

| Provider | Fields | Where to store |
| --- | --- | --- |
| Backblaze B2 | `keyID`, `keyName`, `applicationKey` | secrets vault |
| Wasabi | `access-key`, `secret-key` | secrets vault |

Both are S3-compatible, so most backup tools (restic, rclone, Veeam, Duplicati, etc.) connect using an S3 endpoint URL + access key + secret key.

```
# Example shape only — supply real values from your vault at runtime
endpoint    = <provider-s3-endpoint>
bucket      = <bucket-name>
access-key  = <access-key>
secret-key  = <secret-key>
```

## Notes

- Rotate keys periodically and scope them to the minimum bucket/permissions needed.
