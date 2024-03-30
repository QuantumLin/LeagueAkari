<div align="center">
  <img
    src="https://github.com/Hanxven/LeagueAkari/raw/HEAD/pictures/logo.png"
    width="128"
    height="128"
  />
</div>

# League Akari

利用合法的 League Client Update (LCU) API 实现的工具集。

主要功能包括：

- 查看战绩，包括一切细节

  - 搜索玩家，通过任何标识符，包括召唤师名称（需附带 tag，如 `比企谷小町#33245`）、召唤师 ID 和 PUUID。

  - 多页面战绩详情，你可以同时查看多个玩家，并且任何一个带有玩家的页面，都可以重定向到此页查看其战绩

  - 你可以查看一场游戏的杂项属性，将会通过一个表格尽数展示

  - 预览各种小组件！通过 LCU 内置的数据，可以获取所有装备、符文的介绍，在鼠标悬停时，你可以看看它们

  - 额外：战绩卡片是独立的，因此你可以在任何位置查看任何一场游戏战绩。战绩卡片抄袭了 OP.GG 的大部分样式，但也有一些不同

- 查看对局中和英雄选择途中队友的战绩，给出一定的分析

  - 给出总览分析，不考虑重开局、PVE 局以及自定义、训练模式等

  - 给出英雄选择偏好和胜率信息，可以看看队友更会玩什么英雄

  - 预组队检测，通过分别拉取几十场战绩进行分析。注意分析数量过大需要拉取大量详细对局战绩，可能会触发服务器 QPS

  - 胜率队检测，根据当场次胜率超过一定阈值时，注意可能有误判，谨慎甄别

  - 在结束游戏后记录玩家，并在下次匹配到时给出提示。

  - 在游戏内或英雄选择中，发送我方或敌方的一些信息。游戏中发送使用模拟键盘输入实现

  - 等一些其他功能

- 一些客户端操作

  - 使用调整窗口大小，一般用于重置显示不全的问题，因为使用了 Win32 API 操作了其他进程，因此该功能需要管理员权限。这是一个进阶功能，需要慎用

  - 重启渲染进程，用于重置界面问题。它和重启一样，似乎能解决一切麻烦。重启渲染进程不会导致进行中的任何操作中断，比如，不会触发秒退

  - 等一些其他功能

- 一些工具集

  - 带有英雄选择台模式 (比如大乱斗) 的便捷小台，可以进行无 CD 换英雄

  - 自动化操作，包括：

    - 自动接受对局，这应该最基本的功能。你可以设置延时，并可以在延时内取消本次自动接受

    - 自动回复，去干别的事了？设置一段自动回复，在好友发来信息时回复它们

    - 自动选择，自动禁用。提供一个优先级列表，会尝试从列表中选择一个可选的或可禁用的目标英雄。自动操作会考虑到 ban 位、队友预选等信息。

  - 生涯背景替换，换成任何你想要的英雄或其皮肤。

  - 虚假的段位卡片，在好友 hover 你的好友卡片时，展示一个最强王者段位！

  - 更改在线状态，你可以更改成离线 (在好友视角就是真的不在线) 或离开等状态。注意，不在游戏中时，你无法设置当前状态在游戏中

  - 创建房间，5v5 训练房间、按照 ID 创建房间统统可以。额外地，提供了人机添加功能，你可以设置它们的难度为“一般”，在自定义或训练的房间中 vs 5 个更有挑战性的人机。

  - 观战，通过召唤师名称、ID 或 PUUID，调起观战功能。前提是目标必须存在，且在一个可观战的游戏中 (云顶、人机队列等不能观战)。

  - 等一些其他功能

- 一些调试工具

  - 在全局对象上，挂载了 lcuRequest、router 和 onLcuEvent，因此可以手动监听事件和进行对于 LCU 客户端的 HTTP 请求

  - 一个开箱即用的临时 LCU 事件订阅，你可以监听任何 URI，并且将其打印在控制台上。作者已经为你预留了一些常用的端点

  - 等一些其他功能

注 1: 对于事件 URI 的监听，你可以指定动态 URL，包括如下都是合法的：

- `/lol-gameflow/v1/session` (静态)

- `/lol-gameflow/v1/**` (通配符前缀，只能放在最后面)

- `/lol-gameflow/:version/:endpoint` (带有占位符)

- `/lol-gameflow/v1/*` (带有匿名占位符)

注 2: 对于记录对局玩家，其使用 SQLite 作为数据库，存放在应用目录的 league-akari.db 中。

# 更名

该项目的前名称为 League-Toolkit。由于和现有项目重名，更名为 LeagueAkari。

# 样例

包含了一些使用样例。

## 查询

查询任意一个召唤师。

![查询召唤师](https://github.com/Hanxven/League-Akari/raw/HEAD/pictures/5.gif "查询召唤师")

## 秒选

非常快速的选择操作。

![立即选择](https://github.com/Hanxven/League-Akari/raw/HEAD/pictures/2.gif "立即选择")

## 自动化

一切都在自动过程中。

![完全自动化](https://github.com/Hanxven/League-Akari/raw/HEAD/pictures/3.gif "完全自动化")

## 大乱斗

不止手速快。

![无内置冷却](https://github.com/Hanxven/League-Akari/raw/HEAD/pictures/4.gif "无内置冷却")

# 想要实现的功能

- 能够调出英雄联盟客户端的 DevTools，但是作者对 CEF 了解不深，因此编译不出该功能的 `.node` 模块

# 作者的希望

- 抛砖引玉，希望这个项目能为所有想要实现类似软件的人提供思路。作者使用简单的技术栈，并提供尽可能多的注释和代码可读性

- Star！如果你中意这个项目，请不要吝啬你的 Star。

# 声明

本软件作为基于 Riot 提供的 League Client Update (LCU) API 开发的辅助工具，由于其设计和实施均未采用侵入性技术手段，理论上不会直接干预或修改游戏数据。然而，需明确指出的是，虽然本软件在原理上并未直接修改游戏内部数据，但在游戏环境的持续更新和演变中 (如未来腾讯可能的反作弊系统或其他保护服务的更新)，无法完全排除由于版本更新导致的兼容性问题或其他意外后果。

特此强调，对于使用本软件可能带来的任何后果，包括但不限于游戏账户的封禁、数据损坏或其他任何形式的游戏体验负面影响，本软件的开发者将不承担任何责任。用户在决定使用本软件时，应充分考虑并自行承担由此产生的所有风险和后果。

本声明旨在全面而详尽地通知用户关于本软件使用的可能风险，以便用户在使用过程中做出充分的风险评估和明智的决策。感谢您的关注，同时敬请遵守相关游戏规则和使用指南，确保一种健康和公平的游戏环境。