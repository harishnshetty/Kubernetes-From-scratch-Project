| Workload      | Controller  | Storage | Access |
| ------------- | ----------- | ------- | ------ |
| MySQL         | StatefulSet | EBS     | RWO    |
| MongoDB       | StatefulSet | EBS     | RWO    |
| Jenkins       | StatefulSet | EBS     | RWO    |
| Web uploads   | Deployment  | EFS     | RWX    |
| Static files  | Deployment  | EFS     | RWX    |
| Logs (shared) | DaemonSet   | EFS     | RWX    |


| Controller  | Storage | Access | Result   |
| ----------- | ------- | ------ | -------- |
| Pod         | EBS     | RWO    | OK       |
| Deployment  | EBS     | RWO    | ❌ BROKEN |
| StatefulSet | EBS     | RWO    | ✅ BEST   |
| Deployment  | EFS     | RWX    | ✅ BEST   |
| StatefulSet | EFS     | RWX    | ⚠️ Rare  |


| Workload | Controller  | PVC Count | Storage   |
| -------- | ----------- | --------- | --------- |
| NGINX    | Deployment  | 1         | EFS (RWX) |
| Web App  | Deployment  | 1         | EFS (RWX) |
| MySQL    | StatefulSet | replicas  | EBS (RWO) |
| Kafka    | StatefulSet | replicas  | EBS (RWO) |


| Action             | PVC | PV | EBS                |
| ------------------ | --- | -- | ------------------ |
| Delete Pod         | ✔️  | ✔️ | ✔️                 |
| Delete StatefulSet | ✔️  | ✔️ | ✔️                 |
| Delete PVC         | ❌   | ❌  | ❌ (Delete policy)  |
| Delete PVC         | ❌   | ✔️ | ✔️ (Retain policy) |



| Field                  | Why it matters          |
| ---------------------- | ----------------------- |
| `ebs.csi.aws.com`      | Correct CSI driver      |
| `WaitForFirstConsumer` | AZ-safe volume creation |
| `reclaimPolicy`        | Data retention behavior |
| `allowVolumeExpansion` | Online disk resize      |
| `gp3`                  | Best price/performance  |


| Situation                  | Old EBS reused?    |
| -------------------------- | ------------------ |
| StatefulSet deleted only   | ✅ YES              |
| StatefulSet + PVC deleted  | ❌ NO               |
| PVC deleted (Retain)       | ❌ NO (manual only) |
| Different StatefulSet name | ❌ NO               |
| Same name + PVC exists     | ✅ YES              |
