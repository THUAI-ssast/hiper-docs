# 首页

欢迎来到 Hiper 文档。

Hiper 是一个 AI 编程赛事平台。

价值追求：

- 实用
- 简明
- 通用、易于定制

业务需求：

1. 供清华大学软件学院与自动化系科协举办的 AI 编程赛事使用。
2. 作为正式的开源项目，争取获得一定影响力，在更多同类场景（中小型AI编程赛事，如大学的校内赛事）中得到采用。

## Hiper 的特色

通用、易于定制：

- 使用 Docker 运行用户程序。赛事开发方 能用任何语言编写游戏逻辑、能允许选手用任何语言编写 AI、还可以决定选手能用哪些第三方库。
- 对游戏的限制少、简单直接，即使是 分上下半场的游戏 / 即时制游戏 之类也能支持，而且基本无需针对 Hiper 做太多适配。
- 赛事脚本。可自定义赛制。
- 对局的输入输出可高度自定义。
- 对局详情模板。

TODO：尚未实现. 不过目前的后端实现已经充分做好了扩展性准备。
更多实用功能：

- 支持私下对局
- 支持测试对局，可设定游戏场景测试 AI

## 临时设计文档

### 示例

赛事脚本，给予赛事方极高的自由度与便捷度。赛事方甚至可以考虑在文档里贴一段脚本代码帮忙解释赛制。

示例赛制（部分可考虑作为辅助函数提供，即执行用户提交的赛事脚本前先运行我们写的JS脚本）：

- 天梯制，又名 Elo等级分制度。Botzone 与 Saiblo都基本只用这个赛制。
- 循环赛、淘汰赛。特别简单，软院举办赛事时常用。

可以解说下以下游戏如何部署在本平台：

- THUAI6 软件学院赛道，3 vs 3枪战游戏，1个AI对应3个玩家。
- THUAI6 自动化系赛道，让当前全部AI参与同一场MC游戏。
- THUAI6 电子系赛道，游戏分2个阵营，一名选手负责一个阵营，一个阵营有3个玩家、另一个阵营有1个玩家。一场对局内会跑上下两半场，双方交换阵营，根据上下半场得分大小决定胜负。
    - 干净的游戏逻辑可能是玩家已经都连好、只跑完半场结果就退出。在本平台里这样也是可以的，无需修改游戏逻辑代码；在Dockerfile里不要直接启动游戏逻辑，而是套一层程序即可。

### 前端界面

应显示赛事的基本信息，如名称、所对应的游戏。下设以下tab：

- 介绍(readme)
- 对局(matches)
- 排行榜(ranklist)
- 提交列表（submissions）
- 房间(room)
- 测试(test) 

以下tab右对齐：

- 设置（settings）。仅管理员可见。
- 我的（my）

排行榜 的 `ranklist` 仅作为URL中的标识符使用，英文 UI 中倾向于使用 `ranking list`.

排行榜中显示选手与相应AI；标记该选手目前能否参与公开评测，以表示是否被淘汰。

## 参考项目

设计：

- [Botany](https://botany.ssast.net/#/). 清华大学软件学院科协开发维护。
    - 开源于 [ayuusweetfish/botany](https://github.com/ayuusweetfish/botany)
- [Saiblo](https://www.saiblo.net/). 清华大学计算机系科协智能体部 & 网络部 开发维护。
    - [Saiblo Wiki](https://docs.saiblo.net/index.html)
- [Botzone](http://www.botzone.org.cn/). 北京大学信息科学技术学院人工智能实验室 开发维护。
    - [Botzone Wiki](https://wiki.botzone.org.cn/index.php)

实现：

- T大树洞: [treehollow/treehollow-backend](https://github.com/treehollow/treehollow-backend)
- Botany: [ayuusweetfish/botany](https://github.com/ayuusweetfish/botany)
