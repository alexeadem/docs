---
title: VSCode
---
## QBO VSCode Client

{% preview "vscode.svg" %}

The **QBO VSCode extension** enables developers and ML engineers to seamlessly connect to GPU-powered QBO infrastructure directly from Visual Studio Code. With support for Remote SSH, Kubernetes clusters, and live host selection, it simplifies development workflows for AI, HPC, and cloud-native workloads running directly on metal — **without virtualization**.


{% youtube DKWPNQ_a28w %}

---

## Features

- **Remote SSH Access**  
  Open a full-featured VSCode Remote session on any QBO instance — no manual SSH setup required.

- **Kubernetes Cluster Shell Access**  
  Retrieve the KUBECONFIG for any QBO-managed Kubernetes cluster and launch a terminal with the environment ready for `kubectl`.

- **Dynamic Host Switching**  
  Seamlessly switch between connected QBO host backends in real time. The extension maintains a **single WebSocket connection** at all times to reflect the live state of the selected host.

- **Live API, Host, Instance, and Cluster Views**  
  Visualize QBO system state in the Activity Bar: connected hosts, active instances, and running clusters are always up to date with real-time changes.

- **QuickPick Selection**  
  Interactively browse available clusters or instances using built-in QuickPick menus — no need to copy-paste IDs.

- **Secure Credential Handling**  
  PEM files and KUBECONFIGs are securely stored under `~/.qbo` with appropriate file permissions.

- **Runs on Metal**  
  Connect to QBO environments that deliver **bare-metal GPU performance** without virtualization or cloud overhead.

---

## Commands

| Command                 | Description                                                  |
|-------------------------|--------------------------------------------------------------|
| `qbo: connect to instance` | Select a QBO instance and open a Remote VSCode SSH session |
| `qbo: connect to cluster`  | Select a Kubernetes cluster and open a KUBECONFIG terminal |
| `qbo: connect to host`     | Switch QBO WebSocket connection to a different metal host  |

---

## Requirements

- [VSCode Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)
- `kubectl` installed and available in your system `PATH`
- **QBO CLI credentials**  
  You must have a valid `~/.qbo/cli.json` file containing:

```json
{
  "d": "Afepgkvh0HHCWKRnl-kp_o-v2LSXIIKjIoM-wRZmmYbHb8zQNe-q8-NKi2BzGm6HAyhqQY_Zu7jPbrn572kmZBhK",
  "qbo_aux": "a6738cb7-4d04-468a-a746-bf7b12dbf051",
  "qbo_host": "o-9ef51if5.cloud.eadem.com",
  "qbo_port": 443,
  "qbo_uid": {
    "crv": "P-521",
    "kty": "EC",
    "x": "AJ3XzJ5NSy6yhNNeqyHKtkKtE-ZyNZgEktZCiKoWl8cxlMLpqbb_I8mHvt7JXjxZv_9Z2B5apJb0hsQOo5i1tDPb",
    "y": "AMQx3i5agNNM2Cl0gtnFOR7TcktB3Vwkx3GMuGVbqcQbdj1xD7IFAe9o-DhGn56WqOYOfXKyUn92_MYNeWuZ1k8K"
  },
  "qbo_user": "qbot@qbo.io"
}
