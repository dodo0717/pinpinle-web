# 萌宠拼拼乐 · Web 版（pinpinle-web）

> 《萌宠拼拼乐》网页版，与 Cocos 版**分开存储**的两个仓库之一。
> Cocos 版（主线工程）在 **https://github.com/dodo0717/pinpinle**。

## 跑起来

```bash
node preview/serve.mjs
```

浏览器打开 <http://localhost:5181/web/index.html>（改了代码记得 **Ctrl/Cmd+Shift+R** 硬刷新；
服务已对静态资源加 `Cache-Control: no-store`，但清不掉浏览器已缓存的旧资源）。

- 零依赖：静态服务器会现场做「类型擦除 + 补全 import 扩展名」，不需要 npm install / 构建。
- `PORT` 环境变量可换端口（默认 5181）。

## 目录结构

| 目录 | 内容 |
|---|---|
| `web/` | 页面与 UI 组件（`main.ts` 主控制器 + `web/ui/*` 各屏），`web/art/` 贴纸与地图底图 |
| `assets/scripts/核心/` | **算法层**：关卡配置（`config.ts`）、引擎、排行榜（`honor.ts`）、存档校验 —— 与 Cocos 版同源 |
| `assets/resources/stickers/` | 108 张贴纸 PNG（`web/ui/dom.ts` 的 `STICKER_DIR` 指到这里） |
| `preview/serve.mjs` | 零依赖静态服务器 |

**当前规模**：15 个世界 × 12 关 = 180 关、108 张贴纸、简单 / 困难两种模式。

## ⚠️ 与 Cocos 版的同步关系

算法层 `assets/scripts/核心/` 在两个仓库里**各存一份**：

- 改**数值 / 规则**（关卡配置、星线、榜单口径、存档格式）→ 两边都要改，提交信息里注明另一仓的对应提交；
- 改**Web 界面** → 只改本仓库（`web/`）；
- 改**Cocos 界面** → 只改 pinpinle 仓库（`assets/scripts/场景|界面|玩法/`）。

## 近期更新（倒序，最新在上）

### 2026-11-03

- **地图关卡珠：状态与星级按「简单 + 困难两套模式取最好」显示**（与 Cocos 版同口径）：
  `web/main.ts` 的 `mapView()` —— 任一模式有星就算 `cleared`（星取较高者）、历史最高分取较高者、
  解锁范围取两套 `unlocked` 较大值、「当前关」高亮同口径。此前只按当前模式渲染，
  困难档 3 星打过的关在简单页签下显示「未通关 / 锁定」，而排行榜里那关明明有分有星。
- **困难门槛放宽**：`nightmareUnlocked(level)` —— 困难档**自己已经通关过**这一关时不再被
  「简单 3 星」新门槛锁回（门槛只管首次进入，不把旧规则时代的成绩锁回去）。
- **困难门槛死锁修复**：`web/ui/map.ts` 的 `selectNode` 命中 `lockNote === MAP.nightmareLocked`
  时提示照给，但回调新增的 `handlers.onGateFallback(level)`；
  `web/main.ts` 里该回调 `applyMode('normal')` 切回简单模式并照常 `levelPreview.open`。
  —— 此前困难模式下点「简单还没 3 星」的关，只弹一句提示，而门票（这关的简单 3 星）
  恰恰要在简单版里拿，地图又没有模式切换按钮 → 玩家被卡死。
- **AI 角色分数口径**（算法层 `honor.ts`，两仓同步）：每关先定总分（地板 3★、天花板 1.6×3★），
  **简单分 = 总分 × 60%~80%**（确定性随机，种子 = 关号 + 昵称哈希），困难分 = 总分 − 简单分；
  AI 分数**只配置到第 66 关**（`AI_SCORE_MAX_LEVEL`），67 关起榜单只有本机（空榜「暂无记录」）；
  总分榜 = 第 1~66 关逐关累加。本机玩家两列永远是进度表真实值，**绝不拆分**。
- **验证**：造档实测（简单 1~3 关 3★ / 困难空 → 三颗金珠 `node cleared n-s3`；
  简单/困难各通第 1 关 3★、困难模式点第 2 关 → toast + 牌面打开（模式=简单）+ 节点变 `unlocked`）；
  `tsc` 0 错、单测 249/249（在 Cocos 仓库跑，两仓算法层同源）。

### 更早的更新

见 Cocos 仓库 README「近期更新」与 `web/UI_README.md` §8 更新日志（本仓库 `web/UI_README.md` 同）。
