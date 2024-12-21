# Helm教程


<!--more-->
### Helm入门

Helm 包（Helm Chart）是一个用于定义、安装和管理 Kubernetes 应用程序的包，它可以简化 Kubernetes 应用的部署、配置和管理过程。Helm 是 Kubernetes 的包管理工具，它通过 Helm 包将应用的所有资源和配置打包在一起，使得 Kubernetes 应用的安装、更新、版本控制和发布更加简单和高效。

#### Helm 包的作用 

**1、简化部署流程** 

Helm 通过将应用的多个资源（如 Deployment、Service、ConfigMap、Secret、Ingress 等）打包到一个单一的 Helm 包中，简化了在 Kubernetes 上部署复杂应用的流程。通过 Helm 包，你只需使用一条命令就能自动化地创建所有资源。

**2、版本控制与回滚**

每个 Helm 包都有一个版本号，可以通过版本控制对应用进行管理。你可以轻松地安装、更新、回滚应用版本，以确保 Kubernetes 集群中始终运行着正确的应用版本。如果出现问题，可以快速回滚到先前的版本。

**3、可重用和共享**

Helm 包可以作为模板化的配置文件进行重用和共享。你可以根据需求自定义 Helm Chart，然后通过 Helm 仓库进行共享和分发。这样，不同的团队或开发者可以基于相同的 Chart 进行开发、测试和部署，而无需重复编写相同的配置文件。

**4、自定义配置**

Helm 允许你通过 values.yaml 文件或 --set 参数来传递自定义值，从而修改应用程序的配置。这使得 Helm 包具有高度的灵活性和可配置性，可以根据不同的环境（如开发、测试、生产等）进行调整。

**5、自动化管理**

Helm 包将 Kubernetes 应用程序的生命周期管理自动化，包括安装、升级、删除、回滚等操作。Helm 还能够处理依赖关系，如果应用依赖其他应用，Helm 会在安装过程中自动处理这些依赖。

**6、支持复杂应用**

对于包含多个服务和复杂依赖关系的应用，Helm 提供了强大的支持。通过 Helm 包，可以将所有资源（如数据库、后端服务、前端服务等）及其配置打包在一个 Chart 中，并方便地部署到 Kubernetes 集群。

**7、典型的 Helm 包结构**

一个标准的 Helm 包（Chart）包含以下几个文件和目录：
```
my-chart/
├── Chart.yaml          # 包含 Helm 包的元数据，如名称、版本、描述等
├── values.yaml         # 包含 Chart 的默认配置值
├── charts/             # 可选的依赖项，包含其他 Helm 包
├── templates/          # Kubernetes 资源模板文件，如 Deployment、Service 等
├── .helmignore         # 用于定义 Helm 打包时忽略的文件
└── LICENSE             # 可选，包的许可证
```
说明：

- Chart.yaml：定义 Helm 包的元数据，包括名称、版本、描述等。
- values.yaml：定义默认的配置信息，你可以在此文件中设置应用的参数，并在部署时进行覆盖。
- templates/：包含 Kubernetes 资源的模板文件，这些文件中可以使用 Helm 模板语法（如 {{ .Values.xxx }}）来引用 values.yaml 中的配置值。
- charts/：包含当前 Helm 包的依赖包。
- .helmignore：定义在打包 Helm Chart 时需要忽略的文件（类似 .gitignore）。

#### 使用 Helm 包的好处
- 一致性：Helm 包确保不同环境下应用部署的一致性，减少人为错误。
- 易于管理和维护：Helm 提供了方便的版本控制和回滚机制，可以轻松更新或回滚 Kubernetes 应用。
- 灵活配置：通过 values.yaml 文件和模板语法，你可以根据不同的需求和环境来动态配置应用。
- 跨团队协作：Helm Chart 允许团队之间共享 Kubernetes 应用配置，增强了跨团队协作和一致性。

### 常见命令
1、安装
```
helm install <release-name> <chart-name>
```
2、升级
```
helm upgrade <release-name> <chart-name>
```
3、回滚
```
helm rollback <release-name> <revision-number>
```
4、卸载
```
helm uninstall <release-name>
```
5、查看包状态
```
helm status <release-name>
```
6、查看历史
```
helm hsitory <release-name>
```





---

> 作者:   
> URL: https://yaobg.github.io/posts/notes/devops/k8s/helm/  

