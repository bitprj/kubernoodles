# Minikube

Minikube quickly sets up a local Kubernetes cluster on macOS, Linux, and Windows. It creates a single node cluster contained in a virtual machine (VM). This cluster lets you demo Kubernetes operations without requiring the time and resource-consuming installation of full-blown K8s.

This flexibility enables you to try out preview deployments, perform development tasks, or test configurations easily. Minikube is especially useful for those new to k8s since it enables you to gain familiarity with basic concepts.

### Installing Minikube

{% tabs %}
{% tab title="MacOS" %}
To install the latest minikube **stable** release on **x86-64** **macOS** using **binary download**:



```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikubeTo install the latest minikube stable release on x86-64 macOS using binary download:
```
{% endtab %}

{% tab title="Linux" %}
To install the latest minikube **stable** release on **x86-64** **Linux** using **binary download**:



```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
{% endtab %}

{% tab title="Windows" %}
1.  Download the [latest release](https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe).\
    \
    Or if using `PowerShell`, use this command:

    ```powershell
    New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force
    Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing
    ```

    Copy
2.  Add the binary in to your `PATH`.\
    \
    _Make sure to run PowerShell as Administrator._

    ```powershell
    $oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
    if ($oldPath.Split(';') -inotcontains 'C:\minikube'){ `
      [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine) `
    }
    ```

    Copy

    _If you used a CLI to perform the installation, you will need to close that CLI and open a new one before proceeding._\

{% endtab %}
{% endtabs %}
