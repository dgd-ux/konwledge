一、整体实现
实现基于 RBAC 的权限控制模型，通过 自定义注解 + AOP + REDIS实现权限校验。
库表设计：
niop_nexus_role  niop_nexus_permission  niop_nexus_org  niop_nexus_user
niop_nexus_role_permission：角色权限关联表
???（待定）：niop_nexus_user_role：用户角色关联表
???（待定）： niop_nexus_org_member用户组织关联表新增字段：关联用户在该组织是什么角色（组织管理员、执行者）
问题
PUBLISHER、ORG_LEADER、EXECUTOR、REVIEWER哪些是相对于整个平台的，哪些是相对于组织的

二、核心流程
2.1 注解
@NRCheckPermission
public @interface NRCheckPermission {

    /**
     * 权限编码（支持多个）
     */
    PermissonEnum[] value();

    /**
     * 验证模式：AND | OR，默认 AND
     */
    AuthMode mode() default AuthMode.AND;
}
参数说明：
● value：需要的权限，支持多个
● mode：校验模式
○ AND：需要拥有所有权限
○ OR：拥有任一权限即可
使用：
● 用在方法（接口上）
● 例：
/**
* 表示只需要org:view和org:create其中一种权限就可以了
  **/
  @NRCheckPermission(value=
  {
  PermissionEnum.ORG_VIEW,PermissionEnum.ORG_CRETE
  },
  mode = AuthMode.OR
  )
  2.2 AOP 切面
  工作流程
1. 拦截带有 @NRCheckPermission 注解的方法
2. 获取当前登录用户 ID（从 SysUserThreadLocal）
3. 提取请求的 URI（org、user....），识别模块名（组织、用户、需求等）

4. 从该请求中获取参数（获取到对应模块的id，如orgId、requirementId等）
5. 根据模块名选择对应的 权限获取器 实现
6. 权限获取器 通过userId和对应参数(xxxId)获取权限列表
7. 进行权限校验
   2.3 权限获取器
   以组织模块为例，返回权限的流程：
1. 是否是平台管理员？
   ├─ 是 → 返回平台管理员的所有权限(org:*、sys:*、review:*、demand:xxx)
   └─ 否 → 继续

2. 参数中获取的orgId是否为空？
   ├─ 没有（create\page）
   │   ├─ 查询登录用户是否有组织管理员身份？
   │   │   ├─ 是 → 返回组织管理员的所有权限(org:*、demand:assign、review:view)
   │   │   └─ 否 → 返回 org:view
   └─ 有（delete\update\member-crud）
   ├─ 是该组织的成员？
   │   ├─ 是 → 根据组织内角色返回权限（如果
   是该组织的管理员，返回组织管理员的所有权限(org:*、demand:assign、review:view)，
   如果是普通成员，返回org:view权限）
   │   └─ 否 → 返回 org:view
   2.4 校验权限
   校验方式： 根据 权限获取器 返回的用户拥有的权限列表和注解指定的校验模式（AND/OR）检查权限
   AND模式，权限列表全contains所需权限，校验通过
   OR模式，权限列表只需要包含任意一个所需权限，校验通过
   三、完整工作流程
   权限校验流程图
   用户请求 → Controller 方法
   ↓
   @NRCheckPermission 注解被 AOP 拦截
   ↓
   ┌──────────────────────────────────────────┐
   │ 1. 获取注解参数                            │
   │    - value: [PermissonEnum.ORG_UPDATE]   │
   │    - mode: AuthMode.AND                  │
   └──────────────────────────────────────────┘
   ↓
   ┌──────────────────────────────────────────┐
   │ 2. 获取当前用户 ID                         │
   │    - userId = SysUserThreadLocal.getUserId() │
   └──────────────────────────────────────────┘
   ↓
   ┌──────────────────────────────────────────┐
   │ 3. 提取请求模块名                          │
   │    - URI: /api/nexus/org/update          │
   │    - moduleName: "org"                   │
   └──────────────────────────────────────────┘
   ↓
   ┌──────────────────────────────────────────┐
   │ 4. 从方法参数提取：                       │
   │      • orgId = "org_001"/null ?          │
   │      • userId = "user_123"               │
   └──────────────────────────────────────────┘
   ↓
   ┌──────────────────────────────────────────┐
   │ 5. 获取 权限获取器的 Bean                 │
   │    - Bean name: "org"                    │
   └──────────────────────────────────────────┘
   ↓
   ┌──────────────────────────────────────────┐
   │ 6. 根据校验逻辑获取权限                 │
   │    - 输入: orgId, userId           │
   │    - 输出: ["org:view", "org:update"]   │
   └──────────────────────────────────────────┘
   ↓
   ┌──────────────────────────────────────────┐
   │ 7. 权限校验                              │
   │    - 用户权限: ["org:view", "org:update"]│
   │    - 需要权限: ["org:update"]            │
   │    - 模式: AND                           │
   │    - 结果: true ✓                        │
   └──────────────────────────────────────────┘
   ↓
   通过 → 执行业务逻辑
   拒绝 → 抛出 BizException(PERMISSION_DENIED,"没有权限")