# 操作授权类型

> 本文重点阐述操作授权类型的概念, 在授权及鉴权阶段的作用, 帮助开发者更好地理解操作授权类型

## 1. 前置阅读

- [名词及概念](../Reference/API/02-Model/00-Concepts.md) 中的 `4. Action: 权限操作，比如增删改查`
- [操作 Action API](../Reference/API/02-Model/13-Action.md)

## 2. 什么是操作授权类型

> 操作授权类型决定了操作的授权与鉴权逻辑走abac分支还是rbac分支

- `abac`: 基于属性的访问控制
- `rbac`: 基于角色的访问控制

## 3. `abac`: 基于属性的访问控制

| Subject | 权限类型   | 授权/鉴权模式 |
| ------- | ---------- | ------------- |
| User    | 自定义权限 | abac          |
| User    | 临时权限   | abac          |
| Group   | 自定义权限 | abac          |
| Group   | 模板权限   | abac          |

在`abac`的授权类型下, 操作的所有授权逻辑/鉴权逻辑都走abac的分支, 权限策略中操作关联的资源实例都以表达式的形式组合在一起, 策略表达式表达一个权限的范围 [表达式定义](Reference/Expression/01-Schema.md)

`abac`为权限中心默认的操作授权类型

## 4. `rbac`: 基于角色的访问控制

| Subject | 权限类型   | 授权/鉴权模式 |
| ------- | ---------- | ------------- |
| User    | 自定义权限 | abac          |
| User    | 临时权限   | abac          |
| Group   | 自定义权限 | rbac          |
| Group   | 模板权限   | rbac          |

在`rbac`的授权类型下, 权限被绑定在用户组上, 这里用户组相当于`rbac`中的角色, 所以用户组的权限会走到`rbac`授权/鉴权逻辑分支, 但是用户(User)的权限还是走`abac`分支, 即: 操作的`rbac`授权类型只对用户组生效

操作要配置为`rbac`的授权模式还需要满足以下条件:

- [操作 Action API](../Reference/API/02-Model/13-Action.md)

1. 操作只能关联1个资源类型, 不关联资源类型的操作与关联多个资源类型的都不能配置为`rbac`授权类型
2. 操作关联的资源类型实例选择方式, 即`selection_mode`必须配置成`instance`
3. 操作关联的资源类型的实例视图, 必须配置`ignore_iam_path`为`true`

说明: 在`rbac`中资源实例是绑定在角色上的, 用户通过成为角色来获取权限, 所以权限中心的`rbac`实现也有限制, 操作的实例选择必须为实例

## 5. 为什么要有`rbac`

以`abac`操作授权类型下按照`rbac`的接入模式接入会导致以下问题:

1. Group授权的`abac`表达式中只存在单个实例, 而`abac`表达式是为了方便范围授权, 导致`abac`的表达式碎片化, 策略数量会大幅增长, 进而恶化权限中心鉴权性能
2. 在`rbac`的模式下, 用户组通过加入用户组来获得权限, 而用户如果加入过多的用户组, 也会恶化权限中心鉴权性能, 为此权限中心还加了数量限制 [权限中心限制](./07-Limit.md)

为了解决以上2个问题, 权限中心推出了完全按照`rbac`模式的鉴权方式, 如接入系统以`rbac`的方式来设计权限模型, 需要在创建操作时需设置`auth_type`为`rbac`

## 6. `rbac`配套API

- [鉴权 API v2](../Reference/APIv2/04-Auth/02-DirectAPI.md)
- [接入系统管理类 API v2](../Reference/APIv2/10-Management/00-API.md)

如果以上`接入系统管理类 API v2`不满足需求, 可以继续使用v1接口:

- [接入系统管理类 API](../Reference/API/10-Management/00-API.md)
