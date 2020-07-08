# OpenStack Nova

## 1. Nova 体系结构

* 目前 Nova 主要由 API、 Compute、 Conductor、 Scheduler 4个核心组件构成， 它们之间通过RPC进行通信

* API 是进入Nova的HTTP接口, 可以通过部署多个来实现横向扩展。 API依据请求是长时任务或者是短时任务， 将请求发给 Conductor 或者 Compute。 长时任务请求被发送到Conductor， Conductor负责对其全程跟踪和调度。 对于新建虚拟机或者迁移类需要调度的请求， Conductor会向Scheduler请求一台符合要求的计算节点， 随后Conductor会把请求最终发送到合适的计算节点上。 Conductor 除了长时任务还负责代理其他节点的DB访问。 这主要是为了安全问题和实现在线升级功能。 最终对虚拟机操作的请求都会发送到 Compute 组件，Compute 负责与 Hypervisor 进行通信， 实现虚拟机的生命周期管理。 对各个Hypervisor的支持通过 VirtDriver 框架来支持。

## 2. Nova API

## 3. Rolling Upgrade

## 4. Scheduler

## 5. 典型工作流程