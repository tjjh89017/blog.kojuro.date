# Managing VyOS Routers from Kubernetes with vRouter-Operator


If you run VyOS routers on KubeVirt, Proxmox VE, or bare-metal servers, you probably want a simple way to push configuration to them. That is what **vRouter-Operator** does. It is a Kubernetes operator that manages VyOS router configuration in a declarative way. No SSH, no manual login, just Kubernetes resources.

In this post, I will explain what vRouter-Operator is, how it works, and how **vRouter-Daemon** extends it to support bare-metal routers.

<!--more-->

## Why vRouter-Operator?

Managing network devices by hand is slow and easy to get wrong. When you have many routers, keeping their configuration consistent becomes a real challenge.

vRouter-Operator solves this by letting you define router configuration as Kubernetes custom resources. You write YAML, apply it with `kubectl`, and the operator takes care of the rest. This fits well with GitOps workflows. You can store your router config in Git and let your CI/CD pipeline apply it.

**Key benefits:**

- **No network access needed**: Config is delivered via QEMU Guest Agent (QGA) or gRPC, not SSH
- **Multi-platform**: Supports KubeVirt, Proxmox VE, and bare-metal (via vRouter-Daemon)
- **Declarative**: Define your desired state; the operator handles the execution
- **Template reuse**: Write a template once, apply it to many routers with different parameters

## Custom Resources

vRouter-Operator defines five custom resources. Here is what each one does.

### VRouterTemplate

A **VRouterTemplate** holds a piece of VyOS configuration or commands as a Go template. You can use variables and [Sprig](https://masterminds.github.io/sprig/) functions inside.

```yaml
apiVersion: vrouter.kojuro.date/v1
kind: VRouterTemplate
metadata:
  name: hostname
spec:
  commands: |
    set system host-name '{{ .hostname }}'
```

### VRouterTarget

A **VRouterTarget** represents one VyOS router. It specifies which provider to use (KubeVirt, Proxmox, or vRouter-Daemon) and holds default parameters for template rendering.

```yaml
apiVersion: vrouter.kojuro.date/v1
kind: VRouterTarget
metadata:
  name: vyos-router-01
spec:
  provider:
    type: kubevirt
    kubevirt:
      name: vyos-vm
      namespace: default
  params:
    hostname: "router-01"
    dns: "8.8.8.8"
```

### VRouterBinding

A **VRouterBinding** connects templates to targets. When you create a binding, the operator renders the templates with the target's parameters and generates a **VRouterConfig** for each target automatically.

```yaml
apiVersion: vrouter.kojuro.date/v1
kind: VRouterBinding
metadata:
  name: base-config
spec:
  templateRefs:
    - name: hostname
    - name: dns-settings
  targetRefs:
    - name: vyos-router-01
    - name: vyos-router-02
  save: true
  params:
    dns: "1.1.1.1"
```

Parameters are merged: binding-level params serve as defaults, and target-level params override them.

### VRouterConfig

A **VRouterConfig** is the final, rendered configuration that gets applied to a router. It is usually created automatically by a VRouterBinding, but you can also create one directly.

The status tracks the apply progress:

- **Pending**: Waiting for the VM to be ready
- **Applying**: Configuration is being pushed
- **Applied**: Success
- **Failed**: Something went wrong

### ProxmoxCluster

A **ProxmoxCluster** stores the connection details for a Proxmox VE cluster, including API endpoints, credentials, and sync settings. It supports multiple endpoints for high availability.

```yaml
apiVersion: vrouter.kojuro.date/v1
kind: ProxmoxCluster
metadata:
  name: pve-cluster
spec:
  endpoints:
    - https://pve1.example.com:8006
    - https://pve2.example.com:8006
  credentialsRef:
    name: pve-api-token
  syncInterval: 60
  checkGuestUptime: true
```

## How It Works

The operator runs several controllers that work together:

1. **VRouterTargetController** polls each target every 60 seconds to check if the VM is running. For KubeVirt, it also watches VirtualMachineInstance events for instant detection.

2. **VRouterBindingController** watches templates and targets. When anything changes, it re-renders the templates and creates or updates VRouterConfig resources.

3. **VRouterConfigController** is the main worker. When a VRouterConfig is created or updated, it:
   - Checks if the VM is running
   - Verifies that the VyOS router service is active
   - Sends the configuration via the provider (QGA or gRPC)
   - Polls the execution status until it finishes
   - Updates the VRouterConfig status

4. **ProxmoxClusterController** syncs with the Proxmox API to track which node each VM runs on and detect reboots (both hard reboots and guest-initiated soft reboots).

## Providers

Each provider handles how the configuration reaches the router.

### KubeVirt

The operator connects to the virt-launcher pod using SPDY exec and runs a vbash script inside the VM. No guest network access is needed because it goes through the KubeVirt API.

### Proxmox VE

The operator uses the Proxmox REST API to execute commands via QEMU Guest Agent. The script is base64-encoded and runs asynchronously. The operator polls for the result.

### vRouter-Daemon (Bare-Metal)

For routers running outside the cluster, the operator talks to **vRouter-Daemon** over gRPC. This is where things get interesting.

## vRouter-Daemon

vRouter-Daemon is a separate project that bridges Kubernetes and bare-metal VyOS routers. It has two components:

- **vrouter-server**: Runs inside Kubernetes, provides two gRPC services
- **vrouter-agent**: Runs on each VyOS router, connects back to the server

### Architecture

```text
┌──────────────────────────────────────────────────────┐
│  Kubernetes                                          │
│                                                      │
│  vRouter-Operator ──gRPC──→ vrouter-server           │
│                             ├─ ControlService :50052 │
│                             ├─ AgentService   :30051 │
│                             └─ Redis (HA)            │
└──────────────────────────────┬───────────────────────┘
                               │ NodePort :30051
                               │
┌──────────────────────────────┴───────────────────────┐
│  Bare-Metal VyOS Routers                             │
│                                                      │
│  vrouter-agent ──gRPC──→ AgentService                │
│  (connects back to server)                           │
└──────────────────────────────────────────────────────┘
```

The key idea is that the **agent initiates the connection** to the server. This means the server does not need to reach the router's network. The agent registers itself, and then waits for configuration commands.

### How It Works

1. The operator calls `ApplyConfig()` on the ControlService
2. The server puts the request into a Redis queue
3. The agent picks up the request from the queue
4. The agent renders a vbash script and executes it on VyOS
5. The result flows back through Redis to the server
6. The operator polls `GetExecStatus()` and updates the VRouterConfig status

Redis acts as the broker between server pods. This means you can run multiple server replicas. Any pod can accept a request, and any pod that owns the agent connection will deliver it.

### Init Config and Disconnect Policies

vRouter-Daemon supports an **init config**, a baseline configuration that is always applied before any pushed configuration. This is useful for making sure the router keeps basic connectivity even if a bad config is pushed.

The init config has two phases:

- **Before phase**: Applied before the pushed config (baseline settings)
- **After phase**: Applied after the pushed config commits (protection layer)

This two-phase approach means you can protect critical settings (like management interfaces and firewall rules) from being overwritten.

There are also two **disconnect policies** for when the agent loses connection to the server:

- `keep` (default): Keep the current configuration as-is
- `rollback`: Roll back to the init config to restore a known-good state

### Deploying vRouter-Daemon

On the Kubernetes side, deploy the server and Redis:

```bash
kubectl apply -f deploy/kubernetes/namespace.yaml
kubectl apply -f deploy/kubernetes/redis-ha.yaml
kubectl apply -f deploy/kubernetes/vrouter-daemon.yaml
```

On each VyOS router, install the agent:

```bash
dpkg -i vrouter-agent_<version>_amd64.deb
```

Then edit `/etc/default/vrouter-agent`:

```bash
AGENT_ARGS="--server 172.30.0.40:30051"
```

Start the service:

```bash
systemctl enable --now vrouter-agent
```

The agent uses `/etc/machine-id` as its default agent ID, so you can reference it in the VRouterTarget:

```yaml
apiVersion: vrouter.kojuro.date/v1
kind: VRouterTarget
metadata:
  name: bare-metal-router
spec:
  provider:
    type: vrouter-daemon
    daemon:
      address: "vrouter-server.vrouter-system.svc:50052"
      agentID: "a1b2c3d4e5f6..."
      timeoutSeconds: 60
```

## Putting It All Together

Here is a full example. Say you have two routers, one on KubeVirt and one on bare-metal, and you want them both to have the same hostname pattern and DNS config.

First, create the templates:

```yaml
apiVersion: vrouter.kojuro.date/v1
kind: VRouterTemplate
metadata:
  name: hostname
spec:
  commands: |
    set system host-name '{{ .hostname }}'
---
apiVersion: vrouter.kojuro.date/v1
kind: VRouterTemplate
metadata:
  name: dns
spec:
  commands: |
    set system name-server '{{ .dns }}'
```

Then define the targets:

```yaml
apiVersion: vrouter.kojuro.date/v1
kind: VRouterTarget
metadata:
  name: kv-router
spec:
  provider:
    type: kubevirt
    kubevirt:
      name: vyos-vm
      namespace: default
  params:
    hostname: "kv-router"
---
apiVersion: vrouter.kojuro.date/v1
kind: VRouterTarget
metadata:
  name: bm-router
spec:
  provider:
    type: vrouter-daemon
    daemon:
      address: "vrouter-server.vrouter-system.svc:50052"
      agentID: "a1b2c3d4e5f6..."
  params:
    hostname: "bm-router"
```

Finally, bind them together:

```yaml
apiVersion: vrouter.kojuro.date/v1
kind: VRouterBinding
metadata:
  name: base-config
spec:
  templateRefs:
    - name: hostname
    - name: dns
  targetRefs:
    - name: kv-router
    - name: bm-router
  save: true
  params:
    dns: "8.8.8.8"
```

The operator will create two VRouterConfig objects, one for each target, render the templates with the merged parameters, and push the config through the right provider.

## Summary

| Component | Role |
|-----------|------|
| **vRouter-Operator** | Kubernetes operator that manages VyOS config as custom resources |
| **vRouter-Daemon** (server) | gRPC server in Kubernetes that brokers config to bare-metal agents |
| **vRouter-Daemon** (agent) | Lightweight agent on VyOS that executes pushed config |

vRouter-Operator brings infrastructure-as-code to VyOS routers. Combined with vRouter-Daemon, it covers KubeVirt, Proxmox VE, and bare-metal environments with a single, unified approach.

If you want to try it out:

- [vRouter-Operator on GitHub](https://github.com/tjjh89017/vrouter-operator)
- [vRouter-Daemon on GitHub](https://github.com/tjjh89017/vrouter-daemon)

