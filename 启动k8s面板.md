**1、安装 Kubernetes Dashboard：**

Kubernetes Dashboard 是一个 Kubernetes 官方提供的 Web UI 界面，用于管理和监视集群中的资源。首先，您需要将 Dashboard 安装到您的集群中。可以使用以下命令来安装它：
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml
```
请注意，上述命令将安装 Dashboard 版本 v2.3.1，您可以根据需要更改版本号。

**2、创建用户和角色：**

默认情况下，Kubernetes Dashboard 不会直接开放公开访问，因此您需要创建一个用户，并为该用户授予必要的权限。下面是创建用户和角色的示例：
```
# 创建一个 ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system

# 创建一个 ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```
保存为 admin-user.yaml 并使用以下命令应用配置：
```
kubectl apply -f admin-user.yaml
```

**3、获取登录令牌：**

获取刚刚创建的用户的登录令牌，您可以使用以下命令：
```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```
将输出中的 token 字段的值复制下来，您稍后会用到它。

**4、启动代理：**

要访问 Dashboard，您需要在本地启动一个代理。运行以下命令：
```
kubectl proxy
```
默认情况下，代理会在 http://localhost:8001 上监听。

**5、访问 Dashboard：**

打开您的浏览器并访问以下 URL 以登录到 Kubernetes Dashboard：
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```
选择 "Token" 选项，将刚刚复制的登录令牌粘贴到输入框中，然后点击 "Sign In"。