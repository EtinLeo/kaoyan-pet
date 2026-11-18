# Clawd · 考研桌宠

一只蹲在 macOS 桌面上的 Clawd，盯着你考研。

它用 [clawd-on-desk](https://github.com/rullerzhou-afk/clawd-on-desk) 的**原版 SVG 动画**，
但盯的对象不是 AI coding agent，而是**你自己的考研打卡网站**（自行配置的学习进度接口）。

## 它会干什么

**读你的真实进度** —— 每分钟拉一次 `GET /api/widget`，拿到：

| 数据 | 用来干嘛 |
|---|---|
| `done / total` | 今日任务完成数，每勾掉一项它就冒个泡夸你 |
| `debt` | 欠账。涨了会生气，还了会高兴，堆到 5 个以上定期唠叨 |
| `streak / best` | 连续达标天数，破纪录会蹦起来 |
| `week[].min` | 最近 7 天每天的学习分钟数，面板里画成柱状图 |
| `subjects[]` | 各科总进度条 |
| `remain[]` | 今天还剩哪几科没动，催你的时候会点名 |
| `vocab` | 单词的已熟 / 欠熟练 / 未学 |
| `days / examDate` | 倒计时，台词里会带上 |
| `rest` | 休整日 —— 这天它不催你 |

**抓实时摸鱼** —— 这部分不依赖网站：读前台应用和浏览器当前标签页，
刷抖音/淘宝/B站娱乐区超过 90 秒就开始三级提醒，从「👀」到举手生气。

**番茄钟 · 打瞌睡 · 聊天** —— 检测到学习可自动开钟；键鼠五分钟没动它就趴下睡；
右键能找它说话（配了 API Key 的话由 Claude 现编回复）。

## 跑起来

```bash
cd kaoyan-pet && npm install && npm start
```

启动后 Clawd 出现在右下角，菜单栏也会多一个图标。

### 第一次要给两个系统权限

抓摸鱼那部分靠 AppleScript 读前台窗口，macOS 会弹窗要授权（开发模式下 App 显示为 **Electron**）：

- **辅助功能** —— 读前台应用和窗口标题
- **自动化 → Google Chrome / Safari** —— 读当前标签页的网址

误点了拒绝就去「系统设置 → 隐私与安全性」里手动勾上，面板里也有按钮直接跳过去。
不给也能用，只是少了实时摸鱼提醒，网站数据照常。

### 访问令牌

首次运行后可在设置面板填写，保存位置：`~/Library/Application Support/kaoyan-pet/config.json`。
**不在代码库里**，换令牌去面板「打卡网站」那栏改。

## 怎么玩

| 操作 | 效果 |
|---|---|
| 单击 | 摸头，随口说一句 |
| 拖动 | 换地方（位置记住） |
| 双击 | 打开面板 |
| 右键 | 番茄钟 / 聊天 / 战报 / 隐藏 / 退出 |
| **⌘ + 滚轮** | **直接改大小**，不用开面板 |
| `⌘⇧K` / `⌘⇧P` | 显隐 / 番茄钟 |

大小是连续可调的（16–200px），SVG 矢量，多大都清晰。面板里有滑块带实时预览。

## 表情对应

| 状态 | 动画 |
|---|---|
| 发呆 | `idle-living` / `idle-look` / `idle-yawn` 轮换 |
| 在学习 | `idle-reading`（捧着书） |
| 完成一项 / 达标 | `happy` |
| 今日全清 / 破连击纪录 | `react-double-jump` |
| 抓到摸鱼 / 欠账涨了 | `react-annoyed` |
| 你离开了 | `sleeping` |
| 被拎起来 | `react-drag` |

`assets/clawd/` 下有 47 个动画，想换映射改 `src/renderer/pet.js` 顶部的 `FULL` / `MINI` 两张表。
角色小于 34px 时自动切官方的 `mini-*` 简化版。

## 代码结构

```
src/main/
  main.js       窗口、托盘、点击穿透、拖拽、IPC
  study.js      拉打卡网站的 /api/widget，比对前后两次算出「刚发生了什么」
  monitor.js    读前台应用 / 浏览器标签页 / 键鼠空闲（osascript + ioreg）
  coach.js      每秒 tick：合并两路数据、跑番茄钟、决定什么时候开口
  lines.js      台词库，{done} {debt} {streak} {remain1} 这些占位符会填真实数据
  ai.js         可选的 Claude 台词生成
  store.js      配置持久化
src/renderer/
  pet.*         桌宠窗口（SVG 动画 + 气泡 + 聊天）
  panel.*       数据与设置面板
assets/clawd/   Clawd 的 47 个 SVG 动画（来自 clawd-on-desk）
```

想调"什么时候说什么话"，改 `coach.js` 的 `tick()` 和 `onStudyEvents()`。

## 许可证

**AGPL-3.0-only** —— 因为用了 clawd-on-desk 的动画素材，详见 `NOTICE.md`。
自己用没任何限制；将来要分发的话得同样开源。

## 已知的坑

- 只支持 macOS（摸鱼检测靠 AppleScript 和 `ioreg`）。
- Firefox 没有 AppleScript 接口，读不到网址，只能靠窗口标题。
- 网站连不上时它会退回本地判断，面板顶部会飘一条提示。

## 发布版配置

首次使用不需要学习网站或 AI 密钥。学习网站连接默认关闭，番茄钟、桌宠交互和本地学习提醒可以独立使用；在面板设置考试日期和学习目标。

接入自己的学习站时，填写网站地址及可选访问令牌，再开启连接。接口为 `GET /api/widget`，返回结构见 `examples/widget.json`。AI 聊天默认关闭，启用时填写自己的模型名称与 API 配置。

程序会在本机保存配置和学习统计；启用窗口检测会读取前台应用及浏览器标题。启用学习站连接或 AI 聊天后，相应请求会发送至你配置的服务。

## 商业使用与上游来源

本项目按 **AGPL-3.0-only** 提供，允许商业使用，使用与分发时须遵守许可证。角色动画来自 rullerzhou-afk/clawd-on-desk，保留其 AGPL 许可与来源说明；见 `NOTICE.md` 和 `LICENSE`。项目不代表上游作者或 Anthropic 官方产品。

此发布副本清除了个人学习站地址，不含个人配置、学习记录、录音、凭据或原始提交历史。
