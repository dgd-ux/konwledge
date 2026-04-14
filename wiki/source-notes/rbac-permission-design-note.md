# RBAC 权限设计笔记

这份笔记对应权限系统草案，核心是“注解声明权限 + AOP 拦截 + 按模块获取权限 + AND/OR 校验”。

## 目标

- 建立基于 RBAC 的权限控制模型。
- 通过自定义注解把权限要求声明在接口方法上。
- 运行时根据请求模块和业务 ID 动态计算当前用户拥有的权限集合。

## 关键设计

## 1. 注解

- 使用 `@NRCheckPermission` 声明权限要求。
- `value` 支持多个 `PermissonEnum`。
- `mode` 支持 `AND` 或 `OR`，默认 `AND`。

## 2. AOP 切面流程

1. 拦截带注解的方法。
2. 从 `SysUserThreadLocal` 取当前用户 ID。
3. 从 URI 判断模块名，例如 `org`、`user`。
4. 从方法参数提取业务 ID，例如 `orgId`、`requirementId`。
5. 选择对应模块的权限获取器。
6. 获取用户在该业务上下文中的权限列表。
7. 按 `AND/OR` 执行校验。

## 3. 组织模块的权限获取思路

- 平台管理员直接拥有平台级全量权限。
- 无 `orgId` 时，用于 `create/page` 这类入口，组织管理员可拿到组织管理权限，普通用户可能只有 `org:view`。
- 有 `orgId` 时，根据是否是该组织成员、以及组织内角色返回不同权限。

## 4. 待明确问题

- `PUBLISHER`、`ORG_LEADER`、`EXECUTOR`、`REVIEWER` 哪些是平台级角色，哪些是组织级角色。
- 用户角色表与组织成员角色字段的最终模型还未完全定稿。

## 来源

- [Source 001: RBAC 权限控制设计](../../raw/sources/source-001-rbac-permission-design.md)
