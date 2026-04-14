# RBAC 权限模型

该模型的目标是在接口层用声明式注解表达权限要求，再由运行时根据用户、模块和业务上下文计算可用权限。

## 模型组成

- 角色表：`niop_nexus_role`
- 权限表：`niop_nexus_permission`
- 组织表：`niop_nexus_org`
- 用户表：`niop_nexus_user`
- 角色权限关联：`niop_nexus_role_permission`
- 待定的用户角色关联：`niop_nexus_user_role`
- 待补充的组织成员角色信息：`niop_nexus_org_member`

## 设计思想

- 权限要求定义在接口方法上，而不是散落在业务代码里。
- 运行时权限不是只看用户静态角色，还要看当前业务对象，例如某个 `orgId`。
- 权限判断依赖“模块权限获取器”，不同模块可以用不同规则组装权限。

## 角色层次

- 平台管理员属于平台级角色，拥有跨模块高权限。
- 组织管理员和普通成员属于组织上下文角色。
- `PUBLISHER`、`ORG_LEADER`、`EXECUTOR`、`REVIEWER` 的层级边界在源文档里仍未定稿。

## 常见权限形态

- 平台级：`org:*`、`sys:*`、`review:*`、`demand:*`
- 组织级：`org:view`、`org:update`、`demand:assign`、`review:view`

## 实现重点

- 权限列表在请求期动态生成。
- 一个用户在不同组织、不同资源对象上可能得到不同权限集。
- 这更接近“RBAC + 上下文约束”，不是纯静态角色判断。

## 待确认

- `user_role` 和 `org_member.role` 两层角色模型是否会重复表达同一语义。
- 是否需要引入更明确的“平台角色”和“组织角色”枚举隔离。

## 来源

- [Source 001: RBAC 权限控制设计](../../raw/sources/source-001-rbac-permission-design.md)
