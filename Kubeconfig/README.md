# Day 31: Organizing Cluster Access Using Kubeconfig Files

## 📌 Overview

A **kubeconfig file** is a configuration file that contains all the information needed to connect to and authenticate with Kubernetes clusters.

> **Note:** The term "kubeconfig" is a *generic name* for these configuration files. It doesn't mean there's literally a file named `kubeconfig`. The default file is actually named `config`.

The `kubectl` command-line tool uses this file to:
- Find which cluster to connect to
- Authenticate the user
- Communicate with the cluster's API server

---

## 📂 Default Location

By default, kubectl looks for the kubeconfig file at:

| OS | Default Path |
|---|---|
| **Linux/Mac** | `$HOME/.kube/config` |
| **Windows** | `C:\Users\<YourUsername>\.kube\config` |

You can override this using:
- The `KUBECONFIG` environment variable
- The `--kubeconfig` flag with kubectl commands

---

## 🔑 Three Main Components of a Kubeconfig File

A kubeconfig file organizes information into **three main sections**:

```
┌─────────────────────────────────────────────────────────────┐
│                     KUBECONFIG FILE                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │  Cluster 1  │  │  Cluster 2  │  │  Cluster 3  │         │
│  │ (dev)       │  │ (staging)   │  │ (prod)      │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│         │                │                │                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   User 1    │  │   User 2    │  │   User 3    │         │
│  │ (dev-user)  │  │ (admin)     │  │ (prod-user) │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│         │                │                │                 │
│  ┌─────────────────────────────────────────────────┐       │
│  │                   CONTEXTS                       │       │
│  │  ┌───────────┐ ┌───────────┐ ┌───────────┐      │       │
│  │  │ dev-ctx   │ │ stage-ctx │ │ prod-ctx  │      │       │
│  │  │ cluster1  │ │ cluster2  │ │ cluster3  │      │       │
│  │  │ + user1   │ │ + user2   │ │ + user3   │      │       │
│  │  └───────────┘ └───────────┘ └───────────┘      │       │
│  └─────────────────────────────────────────────────┘       │
│                                                             │
│  current-context: dev-ctx                                   │
└─────────────────────────────────────────────────────────────┘
```

### 1. Clusters
Information about Kubernetes clusters you can connect to.

### 2. Users
Authentication credentials (certificates, tokens, etc.)

### 3. Contexts
A combination that links a **cluster** + **user** + **namespace** together under a friendly name.

---

## 🔄 Understanding Contexts

A **context** is the most important concept! It groups three parameters:

| Parameter | Description |
|-----------|-------------|
| `cluster` | Which cluster to connect to |
| `user` | Which user credentials to use |
| `namespace` | Default namespace for commands (optional) |

**Why are contexts useful?**
- You can switch between different clusters/environments quickly
- Example: Switch between `development`, `staging`, and `production` clusters with a single command

**Command to switch context:**
```bash
kubectl config use-context <context-name>
```

---

## 📄 Complete Kubeconfig File Example

```yaml
apiVersion: v1
kind: Config

# Define available clusters
clusters:
- cluster:
    server: https://development.k8s.example.com:6443
    certificate-authority: /path/to/dev-ca.crt
    # Optional: proxy-url for connecting through a proxy
    # proxy-url: http://proxy.example.org:3128
  name: development

- cluster:
    server: https://production.k8s.example.com:6443
    certificate-authority: /path/to/prod-ca.crt
  name: production

# Define users with their authentication credentials
users:
- name: dev-user
  user:
    client-certificate: /path/to/dev-user.crt
    client-key: /path/to/dev-user.key

- name: prod-admin
  user:
    token: eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...

# Define contexts (cluster + user + namespace combinations)
contexts:
- context:
    cluster: development
    user: dev-user
    namespace: default
  name: dev-context

- context:
    cluster: production
    user: prod-admin
    namespace: kube-system
  name: prod-context

# Set the current/active context
current-context: dev-context
```

---

## 📖 Explanation of Each Section

### Clusters Section

```yaml
clusters:
- cluster:
    server: https://k8s.example.com:6443     # API server URL
    certificate-authority: /path/to/ca.crt   # CA certificate to verify server
  name: my-cluster                           # Friendly name for this cluster
```

**Key fields:**

| Field | Description |
|-------|-------------|
| `server` | URL of the Kubernetes API server |
| `certificate-authority` | Path to CA certificate (validates the server) |
| `certificate-authority-data` | Base64-encoded CA certificate (inline alternative) |
| `insecure-skip-tls-verify` | Skip TLS verification (NOT recommended for production) |
| `proxy-url` | HTTP/HTTPS proxy for connecting to the cluster |

---

### Users Section

```yaml
users:
- name: my-user
  user:
    # Option 1: Certificate-based authentication
    client-certificate: /path/to/user.crt
    client-key: /path/to/user.key
    
    # Option 2: Token-based authentication
    # token: <bearer-token>
    
    # Option 3: Username/Password (deprecated)
    # username: admin
    # password: secret
```

**Authentication Methods:**

| Method | Fields Used |
|--------|-------------|
| **Certificates** | `client-certificate` + `client-key` |
| **Token** | `token` |
| **Username/Password** | `username` + `password` (deprecated) |
| **External Auth** | `exec` (for external authentication plugins) |

---

### Contexts Section

```yaml
contexts:
- context:
    cluster: my-cluster      # Reference to a cluster defined above
    user: my-user           # Reference to a user defined above
    namespace: my-namespace # Default namespace (optional)
  name: my-context          # Friendly name for this context
```

---

### Current Context

```yaml
current-context: my-context
```

This specifies which context kubectl uses by default.

---

## 🌐 The KUBECONFIG Environment Variable

You can use multiple kubeconfig files by setting the `KUBECONFIG` environment variable:

**Linux/Mac:**
```bash
export KUBECONFIG=/path/to/config1:/path/to/config2:/path/to/config3
```

**Windows PowerShell:**
```powershell
$env:KUBECONFIG = "C:\path\to\config1;C:\path\to\config2;C:\path\to\config3"
```

> **Note:** Linux/Mac uses `:` (colon) as delimiter, Windows uses `;` (semicolon) as delimiter.

---

## 🔀 Merging Rules for Multiple Kubeconfig Files

When you have multiple kubeconfig files, kubectl merges them following these rules:

1. **`--kubeconfig` flag** - If set, use ONLY that file (no merging)
2. **`KUBECONFIG` environment variable** - Merge all files listed
3. **Default file** - Use `$HOME/.kube/config` with no merging

**Merging Priority:**
- ✅ **First file wins** for any duplicate values
- Empty filenames are ignored
- Files with invalid content produce errors
- The first file to set `current-context` is preserved

---

## 🛠️ How to Prepare a Kubeconfig File

### Method 1: Manual Creation

1. **Create the directory** (if it doesn't exist):
   ```bash
   mkdir -p ~/.kube
   ```

2. **Create/edit the config file**:
   ```bash
   nano ~/.kube/config
   ```

3. **Add your configuration** (use the template above)

4. **Set proper permissions** (Linux/Mac):
   ```bash
   chmod 600 ~/.kube/config
   ```

---

### Method 2: Using kubectl Commands

```bash
# Set a cluster
kubectl config set-cluster my-cluster \
  --server=https://k8s.example.com:6443 \
  --certificate-authority=/path/to/ca.crt

# Set user credentials
kubectl config set-credentials my-user \
  --client-certificate=/path/to/user.crt \
  --client-key=/path/to/user.key

# Create a context
kubectl config set-context my-context \
  --cluster=my-cluster \
  --user=my-user \
  --namespace=default

# Use the context
kubectl config use-context my-context
```

---

### Method 3: Copy from Cloud Provider

Most cloud providers generate kubeconfig for you:

**AWS EKS:**
```bash
aws eks update-kubeconfig --name my-cluster --region us-east-1
```

**Google GKE:**
```bash
gcloud container clusters get-credentials my-cluster --zone us-central1-a
```

**Azure AKS:**
```bash
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

---

## 📋 Useful kubectl config Commands

| Command | Description |
|---------|-------------|
| `kubectl config view` | Display the merged kubeconfig |
| `kubectl config current-context` | Show the current context |
| `kubectl config get-contexts` | List all available contexts |
| `kubectl config use-context <name>` | Switch to a different context |
| `kubectl config set-context --current --namespace=<ns>` | Change default namespace |
| `kubectl config delete-context <name>` | Delete a context |
| `kubectl config delete-cluster <name>` | Delete a cluster |
| `kubectl config delete-user <name>` | Delete a user |

---

## 📁 File References in Kubeconfig

Important rules about paths:
- Paths in kubeconfig files are **relative to the kubeconfig file location**
- Paths on command line are **relative to current working directory**
- In `$HOME/.kube/config`, relative paths are stored relatively, absolute paths are stored absolutely

---

## ✅ Best Practices

1. **Never commit kubeconfig to version control** - Contains sensitive credentials
2. **Use separate kubeconfig files** for different environments
3. **Set proper file permissions** (`chmod 600`)
4. **Use contexts** to quickly switch between clusters
5. **Use `certificate-authority-data`** with base64-encoded certs for portability
6. **Regularly rotate credentials** for security

---

## 🔗 References

- [Official Kubernetes Documentation - Organizing Cluster Access](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
- [Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
- [kubectl config Command Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config)
