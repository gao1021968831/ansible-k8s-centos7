# Kubernetes HA Cluster Deployment with Ansible

生产级别的 Kubernetes 高可用集群部署 Playbook，基于 CentOS 7.9 和 containerd。

## 架构特点

- **高可用设计**：3 个 master 节点 + N 个 worker 节点
- **API Server 代理**：每个节点上运行 nginx 静态 Pod，反向代理 master 节点的 API Server
- **容器运行时**：containerd
- **网络插件**：Flannel CNI
- **系统要求**：CentOS 7.9

## 项目结构

```
.
├── inventory/              # 清单文件
│   └── hosts.ini          # 主机配置
├── group_vars/            # 组变量
│   ├── all.yml            # 全局变量
│   ├── masters.yml        # master 组变量
│   └── workers.yml        # worker 组变量
├── roles/                 # Ansible 角色
│   ├── system-init/       # 系统初始化
│   ├── containerd/        # containerd 安装
│   ├── kubernetes-install/# Kubernetes 组件安装
│   ├── master-init/       # master 节点初始化
│   ├── cni-plugin/        # CNI 插件安装
│   ├── worker-join/       # worker 节点加入
│   └── nginx-proxy/       # nginx 代理配置
├── site.yml               # 主 playbook
├── ansible.cfg            # Ansible 配置
└── README.md              # 本文件
```

## 使用说明

### 1. 环境准备

编辑 `inventory/hosts.ini`，配置你的主机信息：

```ini
[masters]
master1 ansible_host=192.168.200.10 ansible_user=root
master2 ansible_host=192.168.200.11 ansible_user=root
master3 ansible_host=192.168.200.12 ansible_user=root

[workers]
worker1 ansible_host=192.168.200.20 ansible_user=root

```

### 2. 配置变量

根据需要编辑 `group_vars/all.yml`：

```yaml
k8s_version: "1.28.0"
containerd_version: "1.6.33"
cni_plugin: "flannel"  # 可选: flannel 或 calico
kubernetes_pod_subnet: "10.244.0.0/16"
kubernetes_service_subnet: "10.96.0.0/12"
```

### 3. 执行部署

```bash
# 检查连接
ansible all -m ping

# 执行完整部署
ansible-playbook site.yml

# 执行特定步骤
ansible-playbook site.yml --tags "system-init"
ansible-playbook site.yml --tags "containerd"
ansible-playbook site.yml --tags "master-init"
ansible-playbook site.yml --tags "cni-plugin"
ansible-playbook site.yml --tags "worker-join"
ansible-playbook site.yml --tags "nginx-proxy"

# 执行初始化步骤
ansible-playbook site.yml --tags "init"
```

### 4. 验证集群

```bash
# 在 master 节点上执行
kubectl get nodes
kubectl get pods -A
```

## 高可用架构说明

### nginx 反向代理

- 每个节点（master 和 worker）都运行一个 nginx 静态 Pod
- nginx 监听本地 6443 端口，反向代理所有 master 节点的 API Server
- 应用通过 localhost:6443 访问 API Server，实现高可用

### 静态 Pod 配置

nginx 静态 Pod 清单位于 `/etc/kubernetes/manifests/nginx-proxy.yaml`，kubelet 会自动启动和监控。

## 故障排查

### 查看 nginx 日志

```bash
docker logs $(docker ps | grep nginx-proxy | awk '{print $1}')
```

### 检查 kubelet 状态

```bash
systemctl status kubelet
journalctl -u kubelet -f
```

### 验证 API Server 连接

```bash
curl -k https://localhost:6443/api/v1
```

## 注意事项

1. 确保所有节点网络互通
2. 关闭防火墙或配置相应规则
3. 确保 SSH 密钥配置正确
4. 部署前备份重要数据
5. 生产环境建议使用外部 etcd 集群

## 参考文档

- [Kubernetes 官方文档](https://kubernetes.io/docs/)
- [kubeadm 部署指南](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)
- [containerd 文档](https://containerd.io/)
