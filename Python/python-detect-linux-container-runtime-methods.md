+++
title = "Python 快速判斷是否運行於 Linux 容器環境"
description = "想知道你的 Python 程式是否在 Linux 容器中運行嗎？本文整理多種檢測技巧，從 cgroup、環境變數到第三方庫，讓你輕鬆辨識運行環境，提升部署彈性。"
date = "2025-07-04T08:22:20.074677Z"
updated = "2025-08-03T10:10:43.435Z"
draft = false

[taxonomies]
tags = [ "Container", "Docker", "Linux", "Python", "Podman" ]
providers = [ "Claude" ]

[extra]
withAI = "<https://claude.ai/share/55718b5b-22de-4d58-96a3-f4718c9bea4a>"
+++
{% chat(speaker="jim") %}
python how to know we are running in the linux container env at the runtime?
{% end %}

{% chat(speaker="claude") %}
在 Python 中，有幾種方法可以在運行時檢測是否在 Linux 容器環境中執行：
{% end %}

## 1. 檢查 cgroup 資訊

```python
import os

def is_running_in_container():
    """檢查是否在容器中運行"""
    try:
        # 檢查 /proc/1/cgroup 文件
        with open('/proc/1/cgroup', 'r') as f:
            content = f.read()
            # Docker 容器通常會包含 'docker' 或 'containerd'
            return 'docker' in content or 'containerd' in content or 'lxc' in content
    except (FileNotFoundError, PermissionError):
        return False
```

## 2. 檢查容器環境變數

```python
def is_in_docker():
    """檢查 Docker 特定的環境變數"""
    return os.path.exists('/.dockerenv') or os.environ.get('DOCKER_CONTAINER') == 'true'

def is_in_kubernetes():
    """檢查 Kubernetes 環境"""
    return (os.environ.get('KUBERNETES_SERVICE_HOST') is not None or 
            os.path.exists('/var/run/secrets/kubernetes.io'))
```

## 3. 綜合檢測函數

```python
import os
import platform

def detect_container_environment():
    """綜合檢測容器環境"""
    container_info = {
        'is_container': False,
        'container_type': None,
        'details': {}
    }
    
    # 檢查作業系統
    if platform.system() != 'Linux':
        return container_info
    
    # 檢查 Docker
    if os.path.exists('/.dockerenv'):
        container_info['is_container'] = True
        container_info['container_type'] = 'docker'
        container_info['details']['dockerenv'] = True
    
    # 檢查 cgroup
    try:
        with open('/proc/1/cgroup', 'r') as f:
            cgroup_content = f.read()
            if 'docker' in cgroup_content:
                container_info['is_container'] = True
                container_info['container_type'] = 'docker'
                container_info['details']['cgroup_docker'] = True
            elif 'containerd' in cgroup_content:
                container_info['is_container'] = True
                container_info['container_type'] = 'containerd'
                container_info['details']['cgroup_containerd'] = True
    except (FileNotFoundError, PermissionError):
        pass
    
    # 檢查 Kubernetes
    if os.environ.get('KUBERNETES_SERVICE_HOST'):
        container_info['is_container'] = True
        container_info['container_type'] = 'kubernetes'
        container_info['details']['kubernetes'] = True
    
    # 檢查其他容器運行時
    container_envs = ['DOCKER_CONTAINER', 'CONTAINER', 'container']
    for env in container_envs:
        if os.environ.get(env):
            container_info['is_container'] = True
            container_info['details'][f'env_{env}'] = os.environ.get(env)
    
    return container_info

# 使用範例
if __name__ == "__main__":
    result = detect_container_environment()
    print(f"運行在容器中: {result['is_container']}")
    print(f"容器類型: {result['container_type']}")
    print(f"詳細資訊: {result['details']}")
```

## 4. 使用第三方庫

```python
# 安裝: pip install psutil
import psutil

def is_container_via_psutil():
    """使用 psutil 檢測容器環境"""
    try:
        # 檢查虛擬化類型
        virt_type = psutil.virtual_memory()
        # 容器通常有特定的記憶體配置模式
        
        # 檢查程序樹，容器中 PID 1 通常不是 systemd
        process_1 = psutil.Process(1)
        return process_1.name() != 'systemd'
    except:
        return False
```

## 5. 簡單的一行檢測

```python
import os

# 最簡單的檢測方法
is_docker = os.path.exists('/.dockerenv')
is_container = os.path.exists('/proc/1/cgroup') and 'docker' in open('/proc/1/cgroup').read()
```

## 使用建議

1. **Docker 環境**：檢查 `/.dockerenv` 文件最為可靠
2. **Kubernetes 環境**：檢查 `KUBERNETES_SERVICE_HOST` 環境變數
3. **通用方法**：使用 cgroup 資訊進行檢測
4. **生產環境**：建議使用綜合檢測函數，提供更完整的資訊

這些方法可以幫助你在運行時準確判斷 Python 應用程式是否在容器環境中執行，並據此調整應用程式的行為。

{% alert(edit=true) %}
這段綜合檢測函數我使用在 [jim60105/standalone-civitai-shortcut](https://github.com/jim60105/standalone-civitai-shortcut/blob/0edadd4a9d976afb53dec3663d298f1820441038/scripts/civitai_manager_libs/util.py#L214-L250) 中，用來隱藏 Gradio UI 上一些在容器環境不能使用的按鈕。
{% end %}
