---
slug: day2-helm-rolling-upgrade
id: 9x84mbenxfwl
type: challenge
title: Rolling upgrade
teaser: This section shows how to perform a rolling upgrade, and lists a few things
  to consider before kicking off this upgrade process.
notes:
- type: text
  contents: |-
    The next challenge shows how to quickly upgrade Redpanda with the helm chart.

    ![skate.png](../assets/skate.png)
tabs:
- id: k5ejpbuyxukf
  title: Shell
  type: terminal
  hostname: server
difficulty: ""
timelimit: 600
enhanced_loading: null
---
The versions of both the helm chart and Repdanda should be tracked, and potentially upgraded, when doing any Redpanda upgrade. It is possible to upgrade only one or the other, but please note that not all helm chart versions are compatible with all versions of Redpanda.

We provide a version matrix in order to make this process easier. More details can be found [here](https://docs.redpanda.com/current/upgrade/k-compatibility/#rp).

## Discover new versions

It is recommended to check the release notes for each version (found on the Github releases page linked below) to see if there are any significant changes that could impact your deployment. Also make sure to review the compatibility matrix to see which components have been tested with each other (linked above).

First let's determine which versions to use during the upgrade for each component. Run the following commands to get the latest versions:

Redpanda (from [Github releases page](https://github.com/redpanda-data/helm-charts/releases)):

```bash,run
curl -s https://api.github.com/repos/redpanda-data/redpanda/releases/latest | jq -cr .tag_name
```

Redpanda chart (from [Github releases page](https://github.com/redpanda-data/helm-charts/releases)):

```bash,run
curl -s https://api.github.com/repos/redpanda-data/helm-charts/releases | jq -r '.[].name' | grep redpanda | head -1 | tr "-" "\n" | tail -1
```

This upgrade will focused on the following target versions:

| Component | Version |
|-|-|
| Redpanda | 24.2.7 |
| Redpanda chart | 5.9.9 |

> Note: The version used above are likely different than the latest, but we are pinning to specific version to ensure future users don't run into any unforseen issues. In your environment you will likely want to target upgrading to the latest available version.

Since we are currently on Redpanda 23.3.4, we will proceed with the upgrade order above, upgrading each component and finishing with upgrading Redpanda itself. But we will upgrade Redpanda two times: first to the latest 24.1 (24.1.17 at this time) and last to the latest 24.2 (24.2.7 at this time).

## Update Redpanda chart and first update of Redpanda

Modify the Redpanda resource to the first Redpanda version as well as the updated chart version. First create a patch file:

```bash,run
cat <<EOF > patch.yaml
image:
  tag: "v24.1.7"
EOF
```

Then merge the patch with the latest chart values:

```bash,run
yq '. *= load("patch.yaml")' updated-values-2.yaml > updated-values-3.yaml
```

Now upgrade the deployment with patched values:

```bash,run
helm upgrade --install redpanda redpanda --repo https://charts.redpanda.com -n redpanda --wait --timeout 2h --create-namespace --version 5.9.9 -f updated-values-3.yaml
```

A rolling restart has begun. The first broker is put into maintenance mode, moving any partition leadership to other brokers. Then the pod is redeployed with the updated Redpanda version. Then the broker is taken out of maintenance mode and the process continues to the next broker until all brokers have been upgraded.

Verify the new version has eventually been applied to each broker:

```bash,run
rpk redpanda admin brokers list
```

Eventually the output will be similar the following:

```bash,nocopy
NODE-ID  NUM-CORES  MEMBERSHIP-STATUS  IS-ALIVE  BROKER-VERSION
4        1          active             true      v24.1.7 - 2c0d4bcfb15f814433713fc7be2586b37c4fda06
5        1          active             true      v24.1.7 - 2c0d4bcfb15f814433713fc7be2586b37c4fda06
6        1          active             true      v24.1.7 - 2c0d4bcfb15f814433713fc7be2586b37c4fda06
```

## Last Redpanda update

Now we can update from Redpanda 24.1.17 to 24.2.7. Follow the same steps as above (only this time we don't need to include the Redpanda chart version since it is already updated):

```bash,run
cat <<EOF > patch.yaml
image:
  tag: "v24.2.7"
EOF
```

Then merge the patch with the latest chart values:

```bash,run
yq '. *= load("patch.yaml")' updated-values-3.yaml > updated-values-4.yaml
```

Now upgrade the deployment with patched values:

```bash,run
helm upgrade --install redpanda redpanda --repo https://charts.redpanda.com -n redpanda --wait --timeout 2h --create-namespace --version 5.9.9 -f updated-values-4.yaml
```

You can eventually verify the version again, and this time we'll use rpk:

```bash,run
rpk version
```

Expected output:

```bash,nocopy
Version:     v23.3.4
Git ref:     e562b3441c
Build date:  2024-01-26T00:44:04Z
OS/Arch:     linux/amd64
Go version:  go1.21.3

Redpanda Cluster
  node-4  v24.2.7 - 512ef3edd2d906ddb4f6f68fc3436bbc103ba800
  node-5  v24.2.7 - 512ef3edd2d906ddb4f6f68fc3436bbc103ba800
  node-6  v24.2.7 - 512ef3edd2d906ddb4f6f68fc3436bbc103ba800
```

This challenge is now complete. Click 'Next' in the bottom right to continue to the next assignment.

