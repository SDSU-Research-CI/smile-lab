This repo contains code and yaml for the SysteMs & InteLligEnce (SMILE) Laboratory at San Diego State University for use on TIDE and the larger NRP Nautilus.

This repo assumes that you have:
- Read the TIDE docs for [batch jobs](https://csu-tide.github.io/batch-jobs)
- [Requested access](https://csu-tide.github.io/batch-jobs/getting-access) to the sdsu-smile namespace
- Completed the [Getting Started guide](https://csu-tide.github.io/batch-jobs/getting-started)

**Namespace:** sdsu-smile

## Shared Data

The following PVCs (disks) are available in the sdsu-smile namespace:

| PVC Name | Storage Class | Options | Data Description | Capacity |
| -------- | --------------| ------- | ---------------- | -------- |
| shared-data | rook-cephfs-tide | ReadWriteMany | This PVC is for shared data between members of the lab | 500GiB |

## Example Code

- [GPU pod](./pod.yaml)
