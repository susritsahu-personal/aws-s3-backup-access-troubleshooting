## Scenario

The S3 bucket had versioning enabled. The object `backups/system-log.txt` was accidentally deleted during the troubleshooting simulation.

The objective was to investigate the deletion and recover the object using S3 Versioning.

## Investigation

The object was deleted using the AWS CLI:

`aws s3 rm s3://aws-s3-backup-support-lab-2026/backups/system-log.txt`

A normal S3 listing confirmed that `system-log.txt` was no longer visible.

The object versions were then inspected using:

`aws s3api list-object-versions --bucket aws-s3-backup-support-lab-2026 --prefix backups/system-log.txt`

The output showed:
- The previous version of `system-log.txt` still existed.
- A delete marker had been created.
- The delete marker was the latest version.

This confirmed that the versioned object had not been permanently deleted.

## IAM Permission Issue

An attempt was made to remove the delete marker using its Version ID.

The operation returned `AccessDenied` because the IAM user did not have:

`s3:DeleteObjectVersion`

The IAM policy was updated to include this permission.

## Resolution

After updating the IAM policy, the delete marker was removed successfully using:

`aws s3api delete-object --bucket aws-s3-backup-support-lab-2026 --key backups/system-log.txt --version-id <DELETE-MARKER-VERSION-ID>`

Removing the delete marker made the previous version of `system-log.txt` current again.

## Verification

The bucket was listed again:

`aws s3 ls s3://aws-s3-backup-support-lab-2026/backups/`

The output showed all three objects:

- `app-config.txt`
- `customer-data.txt`
- `system-log.txt`

This confirmed successful recovery of the deleted object.

## Root Cause

The object was hidden by an S3 delete marker created when the object was deleted from a version-enabled bucket.

The initial recovery attempt also failed because the IAM user lacked the `s3:DeleteObjectVersion` permission.

## Key Learnings

- S3 Versioning can protect objects from accidental deletion.
- Deleting an object in a version-enabled bucket normally creates a delete marker.
- Previous object versions remain available unless they are permanently deleted.
- Removing the latest delete marker can restore visibility of the previous version.
- `s3:DeleteObject` and `s3:DeleteObjectVersion` are separate IAM permissions.
- AWS CLI error messages can help identify missing IAM permissions.
