# 权限校验链路

权限校验流程由注解、AOP 切面、模块权限获取器和最终的 AND/OR 计算组成。

## 入口

- 接口方法使用 `@NRCheckPermission`。
- 示例语义是“需要这些权限中的全部”或“至少其中一个”。

## 注解结构

```java
public @interface NRCheckPermission {
    PermissonEnum[] value();
    AuthMode mode() default AuthMode.AND;
}
```

## 运行时流程

1. AOP 拦截带注解的方法。
2. 读取注解中的 `value` 和 `mode`。
3. 从 `SysUserThreadLocal` 获取当前 `userId`。
4. 从请求 URI 中识别模块名，例如 `org`。
5. 从方法参数中提取业务对象 ID，例如 `orgId`。
6. 获取对应模块的权限获取器 Bean。
7. 由权限获取器根据 `userId + 业务ID` 返回权限列表。
8. 执行 `AND/OR` 校验。
9. 通过则进入业务逻辑，否则抛出无权限异常。

## 组织模块示例

- 平台管理员直接短路通过，返回高权限集合。
- 没有 `orgId` 时，多用于列表或创建场景，按“是否为组织管理员”区分。
- 有 `orgId` 时，按“是否属于该组织”以及“在组织中的角色”决定可用权限。

## 设计价值

- 权限校验不侵入 Controller/Service 业务代码。
- 模块级规则可以独立扩展。
- `AND/OR` 模式让注解表达更灵活。

## 风险点

- 模块识别依赖 URI 规则，一旦路由命名不稳定，权限获取器可能选错。
- 参数提取如果约定不统一，AOP 容易拿不到关键业务 ID。

## 来源

- [Source 001: RBAC 权限控制设计](../../raw/sources/source-001-rbac-permission-design.md)
