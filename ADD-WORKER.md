# 添加 Worker 节点

## 使用方法

### 1. 更新 Inventory 文件

编辑 `inventory/hosts.ini`，添加新 worker 节点到 `[new_workers]` 组：

```ini
[workers]
k8s-worker-1 ansible_host=192.168.200.20 ansible_user=root

[new_workers]
k8s-worker-2 ansible_host=192.168.200.21 ansible_user=root
k8s-worker-3 ansible_host=192.168.200.22 ansible_user=root
```

### 2. 执行添加 Worker 节点剧本

```bash
# 添加所有新 worker 节点
ansible-playbook add-worker.yml

# 添加特定节点
ansible-playbook add-worker.yml -l k8s-worker-2

# 分步执行
ansible-playbook add-worker.yml --tags "system-init"
ansible-playbook add-worker.yml --tags "containerd"
ansible-playbook add-worker.yml --tags "kubernetes-install"
ansible-playbook add-worker.yml --tags "worker-join"
ansible-playbook add-worker.yml --tags "nginx-proxy"
ansible-playbook add-worker.yml --tags "verify"
```

### 3. 验证节点

```bash
# 查看所有节点
kubectl get nodes

# 查看节点详细信息
kubectl get nodes -o wide

# 查看节点状态
kubectl describe node k8s-worker-2
```

## 剧本说明

`add-worker.yml` 包含以下步骤：

1. **system-init** - 系统初始化（时区、SELinux、swap、内核参数等）
2. **containerd** - 安装容器运行时
3. **kubernetes-install** - 安装 Kubernetes 组件
4. **worker-join** - 加入集群
5. **nginx-proxy** - 配置 nginx 反向代理
6. **verify** - 验证节点状态

## 注意事项

- 新节点必须与集群网络互通
- SSH 密钥需要正确配置
- 建议逐个添加节点，观察状态
- 添加后可为节点添加标签：`kubectl label nodes k8s-worker-2 <key>=<value>`
