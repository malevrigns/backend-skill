# backend-skill

面向 Go 后端分析、设计和实现的 AI Skill，围绕应用层、业务层、数据层的解耦，以及各层通过契约协作和扩展。

## 能力

- 深入分析 adaptor、repository、接口、构造函数和依赖装配。
- 分别追踪对象构造链、请求调用链和 DTO / DO / Model 数据转换。
- 明确功能扩展的语义契约、修改范围、兼容性与验证要求。
- 交付前后端对接说明、OpenAPI 接口文档、Mock 和联调步骤。
- 规划模块化单体到微服务的演进，包括数据归属、远程契约、幂等、一致性与回滚。

## 使用

将本仓库放入支持 SKILL.md 的助手的技能目录，目录命名为 backend-skill。具体安装方式以所用助手为准。

在 Codex 中，可将仓库克隆到个人技能目录下的 backend-skill 文件夹；若已安装同名技能，先比较内容，避免覆盖已有修改。

使用示例：

```text
使用 $backend-skill 分析这个 Go 项目，说明三层解耦、接口契约和完整请求链路。
使用 $backend-skill 实现管理员分页查询，同时交付接口文档和前端联调说明。
使用 $backend-skill 规划订单模块拆分为微服务，说明契约、数据迁移和失败处理。
```

## 文件

| 文件 | 内容 |
| --- | --- |
| [SKILL.md](SKILL.md) | 主流程和执行要求 |
| [分层设计](references/layered-design.md) | 参考项目与架构分析问题 |
| [扩展契约](references/extension-contracts.md) | 功能扩展方法及分页示例 |
| [前端与 API](references/frontend-api-contracts.md) | 接口文档、类型、鉴权、Mock 与联调 |
| [微服务演进](references/microservice-evolution.md) | 服务拆分、通信、数据一致性及部署迁移 |

## 参考与边界

架构讨论参考 [yuanqinguo/flyfei 的 mall 项目](https://github.com/yuanqinguo/flyfei/tree/master/src/golang/project/backend/mall)。本仓库提供独立编写的技能指引，没有打包该项目源码。参考说明属于分析快照；应用到项目时应重新核对实际源码。示例和演进方案不表示参考项目已经实现这些功能。

本仓库是技能指引，不包含可运行的商城、已生成的业务 OpenAPI 或微服务部署系统；使用技能执行具体开发任务时，应按项目实现交付相应产物。
