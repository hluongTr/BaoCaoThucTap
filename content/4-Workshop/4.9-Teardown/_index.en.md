---
title: "Cleanup and handover"
date: 2026-11-07
weight: 9
chapter: false
pre: " <b> 4.9. </b> "
---

## Before cleanup

Clean up only authorized demo resources. Check whether resources are still needed for the project through 28 November; do not remove shared infrastructure at the end of an individual workshop session. Inventory resource IDs, Regions/accounts, ownership and data to retain. This guide does not execute AWS resource deletions.

## Handover

Keep the commit/JAR and startup instructions, a secret-free configuration inventory, applied schema/migrations, test results and evidence. Store backups only in approved locations and test restoration on a copy. Signed URLs and access tokens are not handover materials.

## Demo cleanup sequence after confirmation

1. Stop application access/processes and record configuration that must be retained.
2. Back up the database if needed, then remove the designated demo RDS instance; check final snapshots and retained backups for continuing charges.
3. Handle identified S3 test files. If Versioning is enabled, review versions and delete markers separately; deleting the current object does not delete every version.
4. Terminate demo EC2 instances when no longer needed; check retained EBS, snapshots and public IPv4/Elastic IPs.
5. Delete demo Cognito pools/clients only when no longer used; retain any pool still used by the project.
6. Review CloudWatch log groups/alarms, networking such as NAT/endpoints, IAM roles/policies and other resources actually created.
7. Check Billing across Regions after updates propagate. Stopping EC2 does not eliminate charges for retained EBS, snapshots or allocated IPs.

**Expected result:** a handover record and retained/deleted resource inventory. Do not claim zero costs before reconciling the bill.
