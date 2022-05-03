# OutSystems 技术问题汇总（持续更新）
OutSystems 带来了全新的开发思想和方式，对传统技术路线的前后端同学来说是一个不小的挑战。其实它并没有颠覆软件开发的底层逻辑，而是对软件开发过程中的每一个环节进行优化和探索。在学习上手的过程中，我们不可避免会遇到许多技术问题，就像当初学习传统开发方式一样，各种技术问题需要经过不断的实践和研究才能得到最佳的解决方案。本文将记录我在实践中遇到的各类有意义的问题并分享初步解决思路，持续更新，欢迎讨论和指正。

## 最佳实践の思路
1. 理解并遵循 OutSystems 提供的默认范式，这是最事半功倍的做法。
2. 如果做一个简单功能需要改动很多地方，很可能存在更好更快的做法。
3. 尽量不要尝试对 OutSystems 的默认范式进行大改，这样做将事倍功半，失去使用低代码的优势和意义。

## 技术问题の列表
### Q1: 一个同时拥有前台（to C）与后台（to B）的应用该使用一个还是两个 Module 进行开发？
这类应用的常见需求是：用户（顾客）在前台（小程序或 H5）浏览和提交数据，客户（公司）在后台（PC 端）对用户数据进行查看和管理。两端共用一个 Database 但有完全不同的数据权限和界面。

在同一个 Module 里操作 Database 最方便，但一个 Module 默认使用一个主题、对应一个 Layout、使用一个菜单列表，这就意味着前后台页面的展示不好区分。虽然我们可以手动创建两个主题、两个 Layout、两个菜单列表来解决这个问题，但这样有违 Module 的默认范式，不是好的解决方法。

个人推荐的做法是：新建两个 Module 分别对应前台和后台，Database 位于其中一个 Module。Database 下的每一个 Entity 有一个属性 Public，代表是否可以跨 Module \/ Application 访问；另一个属性 Expose Read Only 代表跨 Module \/ Application 调用时是否可写。在需要跨 Module 调用 Database 的 Module 里，点击顶部菜单的 Manage Dependencies... 按钮（插座图案），选择需要调用的 Public Entity，完成后即可进行调用。 

### Q2: 如何修改新建页面的默认 Layout 以及如何修改已存在页面的 Layout？
UI Flows / Layout 下内置了5 个组件：
| 名称 | 层级 | 导航栏 | 用途 |
| :--- | :---: | :---: | :--- |
| LayoutBase | 页面级  | 带导航栏（顶部） | 适用于简单背景图 + 文字内容 |
| LayoutBaseSection | 组件级 | 不带导航栏 | LayoutBase 用到的组件 |
| LayoutBlank | 页面级 | 不带导航栏 | 适用于完全自定义内容 |
| LayoutSideMenu | 页面级 | 带导航栏（左侧） | 适用于列表页、详情页 |
| LayoutTopMenu | 页面级 | 带导航栏（顶部） | 适用于列表页、详情页 |

新增页面的默认 Layout 可以在当前使用主题的 Layout 属性找到并修改，仅影响新增页面，已存在页面不受影响。已存在页面由于已经过页面搭建，删掉重来代价太高，幸好在 ServiceStudio 中也有考虑到并提供办法：切换到已存在页面的 Widget Tree，选中 Layout 组件，其 Source Block 属性就是当前使用的 Layout，直接选择新的 Layout 即可。

### Q3: Entity 内部数据关系的建立需要注意什么？
在传统开发模式下，数据库表之间需要建立一对一、一对多、多对多的关联。OutSystems Entity 同样支持这几种关联关系，相关教程 [Modeling Data Relationships](https://www.outsystems.com/training/courses/128/modeling-data-relationships/) 做了很好的介绍。在具体操作时，外键字段的数据类型需要设置为目标 Entity 的 Identifier。

还有一种情况，一个 Entity 内部的数据之间存在某种关联，比如上下级菜单、上下级员工等。在传统开发模式下也比较简单，用一个字段记录关联数据 ID 即可，但 OutSystems 对这种情况支持得不太好，需要开发者自己注意某些细节：
1. 记录关联数据 ID 的字段类型不能用当前 Entity 的 Identifier，否则创建 Entity 时会报错提示必须选择其它 Entity Identifier。
2. 每个 Entity Identifier 看起来虽然是自增整数，但其实都有特有的数据类型。OutSystems 有严格的数据类型机制，Integer 和 Entitty Identifier 是不能混用的，所以在所有必要的地方，都需要使用内置的类型转换方法 IntegerToIdentifier 或 IdentifierToInteger 进行转换。
