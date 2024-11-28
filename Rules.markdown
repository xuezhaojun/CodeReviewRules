# Code Review 规则：标准库优先原则

## 规则描述
在进行代码审查时，应优先使用标准库或项目依赖的成熟库中的工具函数，而不是创建自定义的辅助函数。

## 检查清单
在添加自定义辅助函数之前，需检查以下来源是否已提供所需功能：
- [ ] 语言标准库
- [ ] 项目现有依赖库
- [ ] 项目现有代码库

## 示例

### ❌ 不推荐
```go
// 自定义指针转换函数
func ptr(s string) *string {
    return &s
}

value := ptr("example")
```

### ✅ 推荐
```go
import "k8s.io/utils/ptr"

value := ptr.To("example")
```

## 优势
1. **代码一致性** - 保持项目风格统一
2. **减少重复** - 避免重复实现已有功能
3. **可靠性** - 使用经过充分测试的库函数
4. **可维护性** - 降低维护成本
5. **安全性** - 减少引入 bug 的风险

## 适用场景
- Kubernetes 相关项目中的指针转换
- 常见类型转换操作
- 标准数据结构处理
- 通用工具函数

## 注意事项
- 在引入新的依赖库前，需评估其必要性
- 确保使用的库函数版本与项目兼容
- 在特殊场景下，如果标准库无法满足特定需求，可以考虑自定义实现


# Code Review 规则：命名规范原则

## 规则描述
在进行代码审查时，应确保命名遵循以下规范：
1. 当表示优先级概念时，应使用明确的命名而非隐含的数字顺序
2. 对于 KubeConfig 相关命名，应使用正确的大小写形式 "KubeConfig" 而非 "Kubeconfig"

## 检查清单
- [ ] 优先级相关命名是否清晰表达其用途而非暗示顺序
- [ ] KubeConfig 相关命名是否使用正确的大小写形式
- [ ] 命名是否符合项目既定的命名约定

## 示例

### ❌ 不推荐
```go
// 使用数字暗示优先级
type PriorityBootstrapKubeconfigs struct {
    // ...
}

// 错误的大小写形式
func GetKubeconfig() string {
    // ...
}
```

### ✅ 推荐
```go
// 使用描述性名称
type KlusterletRegistrationBootstrapKubeConfig struct {
    // ...
}

// 正确的大小写形式
func GetKubeConfig() string {
    // ...
}
```

## 优势
1. **清晰性** - 命名直接表达其用途和含义
2. **一致性** - 保持项目命名规范统一
3. **可维护性** - 减少因命名混淆导致的错误
4. **可读性** - 提高代码的可读性和理解性

## 适用场景
- 涉及优先级或顺序的结构体命名
- Kubernetes 配置相关的命名
- API 对象的命名
- 接口和方法的命名

## 注意事项
- 避免使用数字来暗示优先级或重要性
- 保持项目中命名规范的一致性
- 在重构旧代码时注意命名规范的更新
- 确保命名变更不会破坏现有的 API 兼容性


# Code Review 规则：类型定义和结构体组织规范

## 规则描述
在进行代码审查时，应确保：
1. 使用 `kubebuilder:validation:Enum` 为字符串类型添加枚举验证
2. 当结构体中包含多个相关字段时，应将这些字段组织到一个独立的类型中
3. 使用清晰的类型定义来表示不同的配置选项

## 检查清单
- [ ] 字符串类型的枚举值是否添加了 `kubebuilder:validation:Enum` 注解
- [ ] 相关的配置字段是否已经组织到独立的类型中
- [ ] 是否使用了适当的类型定义来表示配置选项
- [ ] 字段注释是否完整且清晰

## 示例

### ❌ 不推荐
```go
type BootstrapKubeConfigs struct {
    // Type specifies the type of priority bootstrap kubeconfigs.
    // +required
    // +kubebuilder:default:=None
    Type TypeBootstrapKubeConfigs `json:"type,omitempty"`

    // 相关配置直接散落在结构体中
    LocalSecrets []string `json:"localSecretsConfig,omitempty"`
    SkipFailedBootstrapKubeConfigSeconds int32 `json:"skipFailedBootstrapKubeConfigSeconds,omitempty"`
    HubConnectionTimeoutSeconds int32 `json:"hubConnectionTimeoutSeconds,omitempty"`
}
```

### ✅ 推荐
```go
type TypeBootstrapKubeConfigs string

const (
    LocalSecrets TypeBootstrapKubeConfigs = "LocalSecrets"
    None         TypeBootstrapKubeConfigs = "None"
)

type BootstrapKubeConfigs struct {
    // Type specifies the type of priority bootstrap kubeconfigs.
    // +required
    // +kubebuilder:default:=None
    // +kubebuilder:validation:Enum=None;LocalSecrets
    Type TypeBootstrapKubeConfigs `json:"type,omitempty"`

    // 相关配置组织到独立的类型中
    LocalSecrets LocalSecretsConfig `json:"localSecretsConfig,omitempty"`
}

type LocalSecretsConfig struct {
    // +required
    // +kubebuilder:validation:minItems=2
    SecretNames []string `json:"secretNames"`

    // +optional
    // +kubebuilder:default:=180
    // +kubebuilder:validation:Minimum=60
    SkipFailedBootstrapKubeConfigSeconds int32 `json:"skipFailedBootstrapKubeConfigSeconds,omitempty"`

    // +optional
    // +kubebuilder:default:=600
    // +kubebuilder:validation:Minimum=180
    HubConnectionTimeoutSeconds int32 `json:"hubConnectionTimeoutSeconds,omitempty"`
}
```

## 优势
1. **类型安全** - 通过枚举验证确保类型安全
2. **结构清晰** - 相关字段组织到独立类型中提高代码可读性
3. **维护性好** - 结构化的代码更易于维护和扩展
4. **验证完善** - 通过 kubebuilder 注解提供完整的字段验证

## 适用场景
- Kubernetes CRD 定义
- API 类型定义
- 配置结构体设计
- 枚举类型定义

## 注意事项
- 确保为字符串类型的枚举添加 `kubebuilder:validation:Enum` 注解
- 相关的配置字段应该组织到独立的类型中
- 为每个字段添加完整的注释和验证注解
- 考虑字段的默认值和验证规则

# Code Review 规则：对象修改和日志记录规范

## 规则描述
在进行代码审查时，应确保：
1. 不直接修改从 lister 获取的对象
2. 在工具函数中修改输入对象前进行深拷贝
3. 使用上下文相关的日志和事件记录

## 检查清单
- [ ] 从 lister 获取的对象是否在修改前进行了深拷贝
- [ ] 工具函数是否对输入对象进行了深拷贝
- [ ] 日志和事件是否包含足够的上下文信息
- [ ] 是否使用了适当的日志级别

## 示例

### ❌ 不推荐
```go
func (r *ReconcileCSR) syncCSR(csr *certificates.CertificateSigningRequest) error {
    // 直接修改从 lister 获取的对象
    csr.Status.Conditions = append(csr.Status.Conditions, certificates.CertificateSigningRequestCondition{
        Type:    certificates.CertificateApproved,
        Status:  corev1.ConditionTrue,
    })

    // 简单的日志记录
    klog.Info("CSR approved")
    return nil
}

// 工具函数直接修改输入对象
func UpdateLabels(obj *metav1.ObjectMeta, labels map[string]string) {
    for k, v := range labels {
        obj.Labels[k] = v
    }
}
```

### ✅ 推荐
```go
func (r *ReconcileCSR) syncCSR(csr *certificates.CertificateSigningRequest) error {
    // 在修改前进行深拷贝
    csrCopy := csr.DeepCopy()
    csrCopy.Status.Conditions = append(csrCopy.Status.Conditions, certificates.CertificateSigningRequestCondition{
        Type:    certificates.CertificateApproved,
        Status:  corev1.ConditionTrue,
    })

    // 使用上下文相关的日志
    log := klog.LoggerWithValues(r.log,
        "csrName", csr.Name,
        "clusterName", csr.Spec.Username)
    log.Info("Approving CSR")

    // 记录有意义的事件
    r.eventRecorder.Eventf(
        csrCopy,
        corev1.EventTypeNormal,
        "CSRApproved",
        "CSR %q for cluster %q is approved",
        csr.Name,
        csr.Spec.Username,
    )
    return nil
}

// 工具函数在修改前进行深拷贝
func UpdateLabels(obj *metav1.ObjectMeta, labels map[string]string) *metav1.ObjectMeta {
    objCopy := obj.DeepCopy()
    for k, v := range labels {
        objCopy.Labels[k] = v
    }
    return objCopy
}
```

## 优势
1. **数据一致性** - 避免意外修改缓存中的对象
2. **并发安全** - 防止并发操作导致的数据竞争
3. **可追踪性** - 详细的日志和事件便于问题排查
4. **代码可靠性** - 减少因对象修改导致的错误

## 适用场景
- 控制器中的对象处理
- 工具函数的实现
- 日志和事件记录
- 缓存对象的操作

## 注意事项
- 始终对从 lister 获取的对象进行深拷贝后再修改
- 工具函数中需要对输入对象进行深拷贝
- 日志应包含足够的上下文信息以便追踪
- 合理使用事件记录器记录重要操作
```


# Code Review 规则：Deployment 滚动更新策略规范

## 规则描述
在进行代码审查时，应确保：
1. 不使用 `Recreate` 作为 Deployment 的滚动更新策略
2. 在 `spec.strategy.rollingUpdate` 字段中显式设置为 `null`

## 检查清单
- [ ] Deployment 的滚动更新策略是否未设置为 `Recreate`
- [ ] `spec.strategy.rollingUpdate` 字段是否显式设置为 `null`

## 示例

### ❌ 不推荐
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  strategy:
    type: Recreate
```

### ✅ 推荐
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate: null
```

## 优势
1. **高可用性** - 避免在更新期间服务中断
2. **灵活性** - 允许更细粒度的更新控制
3. **一致性** - 确保所有 Deployment 使用一致的更新策略

## 适用场景
- Kubernetes Deployment 配置
- 应用程序的持续交付和部署

## 注意事项
- 确保不使用 `Recreate` 策略以避免服务中断
- 在 `spec.strategy.rollingUpdate` 中显式设置 `null` 以确保配置的明确性
- 定期审查 Deployment 配置以确保符合最佳实践


# Code Review 规则：Eventually 更新操作规范

## 规则描述
在进行代码审查时，应确保：
1. 对资源的更新操作应放在 `Eventually` 块中以处理并发修改
2. 在测试中使用 `Eventually` 来处理资源更新的冲突情况
3. 合理设置重试超时和间隔时间

## 检查清单
- [ ] 资源更新操作是否放在 `Eventually` 块中
- [ ] 是否正确处理了资源版本冲突
- [ ] 是否设置了合适的超时和间隔参数
- [ ] 是否包含了必要的错误处理逻辑

## 示例

### ❌ 不推荐
```go
func TestClusterApproval(t *testing.T) {
    // 直接更新可能导致版本冲突
    managedCluster.Spec.HubAcceptsClient = false
    _, err := clusterClient.ClusterV1().ManagedClusters().Update(
        context.TODO(), managedCluster, metav1.UpdateOptions{})
    if err != nil {
        t.Errorf("failed to update cluster: %v", err)
    }
}
```

### ✅ 推荐
```go
func TestClusterApproval(t *testing.T) {
    gomega.Eventually(func() error {
        // 获取最新版本的资源
        managedCluster, err := clusterClient.ClusterV1().ManagedClusters().Get(
            context.TODO(), clusterName, metav1.GetOptions{})
        if err != nil {
            return err
        }

        // 进行更新
        managedCluster.Spec.HubAcceptsClient = false
        _, err = clusterClient.ClusterV1().ManagedClusters().Update(
            context.TODO(), managedCluster, metav1.UpdateOptions{})
        return err
    }, eventuallyTimeout, eventuallyInterval).Should(gomega.Succeed())
}
```

## 优势
1. **可靠性** - 自动处理资源版本冲突
2. **稳定性** - 减少测试的偶发性失败
3. **可维护性** - 提供清晰的错误处理模式
4. **一致性** - 统一的资源更新模式

## 适用场景
- 集成测试中的资源更新
- 并发环境下的资源操作
- 需要重试逻辑的操作
- 异步操作的验证

## 注意事项
- 设置合适的超时时间和重试间隔
- 在 `Eventually` 块中获取最新的资源版本
- 考虑添加适当的错误日志
- 避免在 `Eventually` 块外部进行资源更新


# Code Review 规则：Kubernetes CA 证书访问规范

## 规则描述
在进行代码审查时，应确保：
1. 优先使用 Pod 中自动挂载的 ServiceAccount CA 证书
2. 避免不必要的 ConfigMap API 调用
3. 遵循 Kubernetes 标准的证书访问路径

## 检查清单
- [ ] 是否直接使用了 ServiceAccount 挂载路径下的 CA 证书
- [ ] 是否避免了不必要的 API 调用
- [ ] 是否使用了正确的证书文件路径
- [ ] 是否正确处理了文件读取错误

## 示例

### ❌ 不推荐
```go
func (f *FlightCtl) getAgentRegistrationCA(ctx context.Context) (string, error) {
    if f.cachedCA != "" {
        return f.cachedCA, nil
    }

    // 通过 API 调用获取 ConfigMap
    configMap := &corev1.ConfigMap{}
    err := f.clientHolder.RuntimeClient.Get(ctx, types.NamespacedName{
        Name:      "kube-root-ca.crt",
        Namespace: "kube-system",
    }, configMap)
    if err != nil {
        return "", err
    }

    ca, ok := configMap.Data["ca.crt"]
    if !ok {
        return "", fmt.Errorf("ca.crt not found in configmap kube-root-ca.crt")
    }

    f.cachedCA = base64.StdEncoding.EncodeToString([]byte(ca))
    return f.cachedCA, nil
}
```

### ✅ 推荐
```go
func (f *FlightCtl) getAgentRegistrationCA(ctx context.Context) (string, error) {
    if f.cachedCA != "" {
        return f.cachedCA, nil
    }

    // 直接读取挂载的 CA 证书文件
    caData, err := os.ReadFile("/var/run/secrets/kubernetes.io/serviceaccount/ca.crt")
    if err != nil {
        return "", fmt.Errorf("failed to read service account CA: %v", err)
    }

    f.cachedCA = base64.StdEncoding.EncodeToString(caData)
    return f.cachedCA, nil
}
```

## 优势
1. **性能优化** - 避免不必要的 API 调用
2. **简化代码** - 减少代码复杂度
3. **最佳实践** - 符合 Kubernetes 标准做法
4. **可靠性** - 使用系统提供的标准机制

## 适用场景
- 需要访问集群 CA 证书的场景
- Pod 中运行的服务组件
- 与 API Server 通信的客户端代码
- 证书管理相关功能

## 注意事项
- 确保使用正确的证书文件路径
- 正确处理文件读取错误
- 考虑证书缓存机制
- 确保 Pod 有适当的权限访问证书文件
