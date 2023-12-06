# 赛事脚本

赛事管理员可提交赛事脚本，实现自动化生成对局、淘汰选手等功能。

编程语言为 JavaScript，运行时为 [Goja](https://github.com/dop251/goja)。支持 ES5.1 与 核心的 ES6 特性（尚不支持的 ES6 特性在赛事脚本中应该都是用不到的，如 `BigInt`, `async-iteration`）。

??? note "Goja 对 JavaScript 特性的详细支持情况"

    - [README](https://github.com/dop251/goja)
    - [a collection of ES6+ features currently implemented or in progress](https://github.com/dop251/goja/milestone/1?closed=1)
    - [tc39_test.go](https://github.com/dop251/goja/blob/master/tc39_test.go#L28) 中的 `skipList` 与 `featuresBlackList` 列出了 Goja 不支持的 ES6 特性。

若实在想用 Goja 不支持的特性，可考虑用 [Babel](https://babeljs.io/) 转译；类似的，TypeScript 也是能用的，只要转译后再提交即可。

引入了 [goja_nodejs](https://github.com/dop251/goja_nodejs) 中的 `eventloop`，提供 `setTimeout`、`setInterval` 等函数。

## 简单示例

```js
初赛、复赛、决赛
```

## 可调用的函数、变量等

可调用的变量与函数，为了后续新增功能时不破坏兼容性，都封装在 `hiper` 对象中。

```js
/**
 * 选手。选手即在本赛事中指定了出战 AI 的用户。
 * @typedef {Object} Contestant
 * @property {string} username - 选手的用户名。
 * @property {number} assignedAiId - 选手的出战 AI 的 ID。
 * @property {number} points - 选手积分，用于排名。
 * @property {string} performance - 选手的表现描述，具体格式由赛事脚本自定义。可在排行榜中展示。
 * @property {boolean} publicMatchEnabled - 能否参与公开对局，即是否尚未被淘汰。
 */
```

### 基本函数

```js
/**
 * 创建一场对局。
 *
 * @param {Contestant[]} contestants - 参与对局的选手的数组，每个元素是一个选手对象。参与对局的 AI 将是他们此时的出战 AI，在 创建对局 与 对局开始 之间修改出战 AI 不会影响对局。
 * @param {MatchOptions} [options={}] - 可选的额外配置。
 
 */
function createMatch(contestants, options = {})

/**
 * @typedef {Object} MatchOptions
 * @property {string} [tag=""] - 对局要打上的标签
 * @property {Object} [extraInfo={}] - 传递给游戏逻辑的额外信息，将以 JSON 格式表示。
 */
```

```js
/**
 * 按排名顺序获取选手列表。
 *
 * @param {string} [filter="survived"] - 筛选选手的条件。可选值为 "all"（所有选手）、"eliminated"（已淘汰选手）、"survived"（未淘汰选手）。 
 * @returns {Contestant[]} - 每个元素是一个选手对象，包含选手的一些信息。
 */
function getContestantsByRanking(filter = "survived")
```

```js
/**
 * 修改选手信息
 * 
 * @param {Contestant} contestant - 要修改的选手。
 * @param {Object} body - 要修改的内容。允许填写的字段有：score: number, performance: string, publicMatchEnabled: boolean, assignAiEnabled: boolean。
 */
function updateContestant(contestant, body)
```

### 辅助函数

## hook函数

hook 函数没有封装在 `hiper` 对象中，可直接在全局作用域中定义。

```js
/**
 * 选手修改了出战 AI 时触发。
 * 
 * @param {Contestant} contestant - 修改了出战 AI 的选手。
 */
function onAiAssigned(contestant)
```

```js
/**
 * 
 * @typedef {Object} Player
 * @property {Contestant} contestant - 选手。
 * @property {number} score - 选手在本场对局中的得分。
 */

/**
 * 一场对局完成时触发。
 * 
 * @param {Player[]} players
 * @param {string} tag - 对局的标签，在创建对局时指定。
 * @param {string} replay - 对局的回放，以 JSON 格式表示。
 */
function onMatchFinished(players, tag, replay)
```
