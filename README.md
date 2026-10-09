# 🍄 蘑菇勇者 · 俯视角打怪收集小游戏（Cocos Creator 3.8.8）

> 4399 笔试项目：一款基于 **Cocos Creator 3.8.8 + TypeScript** 的 2D 俯视角小游戏。
> 玩家操控一只蘑菇小战士，在镜像拼接的大草原世界里拖动虚拟摇杆移动，静止时自动索敌攻击怪物，
> 击杀怪物后其掉落金币，金币磁吸自动入袋并计入 HUD。附带 3 秒开屏 LOGO 广告流程。

---

## 一、项目简介

- **玩法定位**：轻量动作 + 打怪收集 + 大地图探索（俯视角 2D）。
- **画面**：1280×720 设计分辨率；世界由 **2 行 × 3 列** 草地贴图无缝拼接（实际尺寸 3840×1440），镜头平滑跟随主角且锁定不出世界边界。
- **交互**：触屏/鼠标“按下即出”虚拟摇杆（拖动移动、带死区），松手即停。
- **运行方式**：开屏 3 秒 → 自动进入主玩法（可离线打包试玩）。

---

## 二、技术栈与工程信息

| 项目 | 说明 |
| --- | --- |
| 引擎 | Cocos Creator **3.8.8**（`creator.version`） |
| 语言 | TypeScript（`assets/scripts`） |
| 场景 | `assets/scenes/Splash.scene`（开屏）、`assets/scenes/Main.scene`（玩法） |
| 设计分辨率 | 1280 × 720 |
| 渲染 | **纯 UI（Canvas 2D）**，未引入物理引擎，碰撞/绕障为代码级实现 |
| 构建产物 | `build/web-mobile-004`（最新可试玩静态包，含双端一键启动脚本） |

---

## 三、如何运行 / 体验

### 1. 在 Cocos Creator 中运行（开发调试）

1. 用 Cocos Creator **3.8.8** 打开本目录。
2. 打开 `assets/scenes/Splash.scene`，点击编辑器顶部 **▶ 预览**（浏览器/模拟器均可）。
3. 入口场景由项目配置 `profiles/v2/packages/preview.json → general.start_scene` 决定，当前指向 **Splash**：
   先看 3 秒开屏 LOGO，随后自动 `loadScene('Main')` 进入游戏。

> 直接打开 `Main.scene` 预览亦可跳过开屏进入玩法。

### 2. 试玩已构建的静态包（无需 Cocos Creator）

打包产物是 **多文件静态站点**（`index.html` + `cocos-js/` + `assets/` + `src/settings.json`）。
受浏览器安全策略限制（`file://` 下跨文件读取/CORS 会被拦截），**不能双击 `index.html`**，需要任意 HTTP 服务。

本项目已在 `build/web-mobile-004/` 内置一键启动器：

- **macOS**：双击 `Play.command`（若系统拦截，右键 → 打开）
- **Windows**：双击 `Play.bat`

启动器会自动在本地起一个临时 `http.server` 并打开浏览器（端口被占用时自动顺延；关闭终端窗口即停止服务）。
若本机没有 Python，可用 `npx http-server -p 8123` 替代。

> 把整个 `build/web-mobile-004/` 目录上传到任意静态托管（如 nginx、GitHub Pages、对象存储）后，直接访问 `index.html` 即可分发给他人试玩。

### 3. 重新构建

菜单 **项目 → 构建发布**，选择 **web-mobile**：

- **场景勾选**：必须在场景列表中同时包含 `Splash` 与 `Main`，且 **Splash 在上**（否则运行期无法按场景名切到 `Main`）。
- 构建产物输出到 `build/web-mobile-XXX`，随后按第 2 步方式试玩即可。

---

## 四、玩法与操作

| 操作 | 行为 |
| --- | --- |
| 按住屏幕 / 鼠标左键 | 按下处出现**虚拟摇杆**底盘 + 摇杆帽 |
| 拖动 | 主角朝摇杆方向移动（速度 150 px/s；拖动距离过小为死区，视为未操作） |
| 松手 | 摇杆消失，主角停下 |
| 静止 | 自动索敌：最近的怪物进入 **170 px 射程** 后，每 **0.7 s** 攻击一次（每次 1 点伤害） |

**循环**：打死怪物 → 怪物原地掉落 1 枚金币 → 金币短暂停顿后**磁吸加速**飞向主角入袋 → 顶部 HUD 金币数 +1（带弹跳脉冲）。场上怪物低于上限时持续补刷，可一直刷。

**数值速览**（可读 `assets/scripts/*.ts` 中 `@property` 核对）：

| 角色 | 血量 | 速度 | 碰撞半径 | 备注 |
| --- | --- | --- | --- | --- |
| 玩家 | 10 | 150 | 34 | 初始金币 6，摇杆半径 75，显示缩放 ≈0.21 |
| 怪物 | 2 | 55 | 36 | 靠近到 70px 停止贴脸；掉落 1 金币 |

**刷怪规则**（`GameManager`）：开局 3 只；场上存活 ≤ 4 只；低于上限每 **1.8 s** 在玩家周围 240~400px 环带补刷 1 只（尝试 12 次保证不越出世界，越界时收敛到边界内兜底）。

> 当前主线以“打击反馈 / 收集 / 碰撞 / 镜头”为演示重点，怪物近身后不做主动反伤；
> 玩家的受击扣血、闪红与死亡动画（`PlayerController.takeDamage / die`）接口已预留，可随时接入怪物攻击逻辑。

---

## 五、场景流转与关键实现

### 1. 开屏广告（Splash → Main）

`Splash.scene`：黑底 + 居中透明 LOGO，挂 `Splash.ts`，停留 **3 秒**后 `director.loadScene('Main')`。
场景切换做了双保险：优先按**场景名**加载（web 构建生效）；失败时回退 `assetManager.loadScene('db://assets/scenes/...')`（编辑器预览生效），两种环境都能跑。

### 2. 镜像拼接大世界 + 平滑镜头

- 6 块 `1280×720` 草地贴图（`Bg_C0_R0…Bg_C2_R1`）平铺成 **2 行 × 3 列**，实际世界 = 3840×1440，半宽高 = (1920, 720)。
- 主角坐标写在 **World 局部系**，镜头用一个 `World` 反向平移实现：每帧 `World.position = -镜头中心`，
  镜头中心按 `k = 1 - exp(-8·dt)` 指数平滑逼近玩家（带边界钳制：**视窗永不露出世界外**）。
- 头顶血条挂在角色自身节点下，随 World 一起滚动；HUD（金币图标/数字、摇杆盘）挂在 Canvas 上，不随镜头动。

### 3. 纯 UI 场景的“代码级碰撞”（无物理引擎）

整场无任何物理组件，玩家/怪物坐标均为手动推进。`Obstacles.ts` 把实心装饰抽象为**世界坐标碰撞圆**：

- `SOLID_NAMES = { tree1, tree2, stone, crate, bush }`（花朵、装饰簇根节点不挡路，可穿过）。
- 半径 = `0.5 × max(宽,高) × scale × 0.72`（下限 30），在 `onLoad` 收集一次。
- 移动方每帧 `resolveCircles`：逐障碍圆**只消去径向重叠、保留切向分量** → “直线顶不进、斜向能贴边滑过”，实现不穿模手感。

### 4. 怪物 AI：追踪 + 势场绕障

`MonsterController` 每帧朝玩家方向移动，同时做**两层避障**：

1. **势场式绕行**：进入障碍影响带（`障碍半径 + 自身碰撞半径 + 22px`）即被推离；当障碍几乎正对行进方向时，
   再叠加一个恒定**侧向偏置**，避免把怪“顶死在”树/箱前。
2. **硬兜底**：绕行后仍调 `resolveCircles` 再推挤一次，保证任何情况下怪物都不会与障碍重叠。

### 5. 战斗表现（姿态切换 + 反馈）

- 玩家四态 `idle / walk / attack / hit` 通过直接切换 SpriteFrame + 小位移 Tween 表现：
  idle 轻微上下漂浮（±3px、1s 往返），walk 脚步起伏更快，attack 前冲 14px 震摆并在摆回前结算一次伤害，hit 闪红 0.12s。
- 怪物受击：闪白→红→白（各 0.1s）+ 受击姿态，0.3s 后恢复行走；朝向通过 `scale.x` 翻转实现，永不旋转 Sprite。
- 左右朝向仅翻转 `scale.x` 的符号，统一用 `visualScale` 作为基数，避免翻转时尺寸漂移。

### 6. 血条（单条，无“两条/不掉血”问题）

- 结构：角色根节点下 `HpBar/HpRim`（描边底框）+ `HpFill`（**左锚点** 0,0.5 的填充条）。
- 满血比例按运行时实际显示宽度换算：`fillBase = (rim 显示宽 × 0.9) / fill 原宽`；掉血即 `scaleX = fillBase × hp/maxHp`，从右向左线性缩短。
- 美术上把 `hp_rim` 原图中间的绿色内芯**镂空成透明（只留描边框）**，保证掉血后画面只显示唯一一条填充条。

### 7. 金币磁吸收集

`GoldCoin`（`GoldCoin.prefab`）：生成时 `0→1` 弹性弹出 → 停顿 0.15s → 进入追踪；
进入 **320px 磁吸范围**后按“越近越快”（`460 × (0.35 + 0.65 × 贴近度)`）飞向玩家；贴近 < 26px 结算
`GameUI.addGold(1)` 并销毁。`GameUI` 初始金币 6，`addGold` 触发数字标签 1.35 倍弹跳脉冲。

---

## 六、场景节点结构（Main）

```
Canvas  (1280×720)
├─ World                         ← 镜头跟随容器（整树随镜头反向平移）
│  ├─ Bg_C0_R0 … Bg_C2_R1        ← 6 块草地（2 行 × 3 列）
│  ├─ DecorRoot                  ← 装饰总根
│  │   ├─ tree1 / tree2 / stone / crate / bush / flower / flower2   ← 世界中心一簇
│  │   └─ Deco_BL / Deco_BR / Deco_TL / Deco_TC / Deco_TR           ← 四周五簇（各含同款 7 件）
│  ├─ PlayerRoot                 ← 玩家（挂 PlayerController）
│  │   ├─ PlayerSprite
│  │   └─ HpBar → HpRim + HpFillWrap → HpFill
│  └─ Monsters                   ← 怪物/金币容器（空）
├─ GameManager                   ← 刷怪调度（空节点挂 GameManager.ts）
├─ UI                            ← HUD 逻辑（空节点挂 GameUI.ts）
├─ CoinIcon                      ← 金币图标
└─ CoinLabel                     ← 金币数字（Label）
```

坐标约定：`World` 下所有实体用同一套世界局部坐标（不随镜头滚动的“世界坐标”），
装饰碰撞圆的圆心与玩家/怪物可直接相减，简化了碰撞与索敌运算。

---

## 七、脚本职责一览

| 脚本 | 位置 | 职责 |
| --- | --- | --- |
| `Splash.ts` | Splash 场景 | 3s 开屏倒计时 → 切 Main（含编辑器/构建双保险） |
| `PlayerController.ts` | Canvas/PlayerRoot | 虚拟摇杆移动、镜头跟随、自动索敌攻击、受击/死亡、与障碍碰撞 |
| `MonsterController.ts` | Skeleton.prefab 实例 | 追踪玩家、势场绕障、受击扣血、死亡掉落金币 |
| `GameManager.ts` | Canvas/GameManager | 开局刷 3 只、保持场上 ≤4、周期补刷、注入运行时依赖 |
| `GameUI.ts` | Canvas/UI | 金币计数与加金币弹跳 |
| `GoldCoin.ts` | GoldCoin.prefab | 弹出登场 → 磁吸追踪 → 入袋结算 |
| `Obstacles.ts` | 纯模块 | 收集实心装饰碰撞圆 + 圆-圆推挤（`collectObstacles`/`resolveCircles`） |

**预制体**：

| 预制体 | 内容 |
| --- | --- |
| `Skeleton.prefab` | 怪物根节点：`Body`(Sprite) + `HpBar`(HpRim / HpFill) |
| `GoldCoin.prefab` | 金币根节点：挂 `GoldCoin.ts` + 金币 Sprite |

**素材**（`assets/textures`，按语义分目录）：`bg/` 草地、`decorations/` 树/石/箱/灌木/花、`monster/` 与 `player/` 各 4 姿态、`item/gold_coin`、`ui/`(logo / hp_rim / hp_fill)。

---

## 八、素材美术管线（透明底预处理）

原始素材为白底/带水印，统一经 `tools/preprocess/replace_new_assets.py` 自动处理成**透明底**并覆盖同名贴图：

- 主体提取：取 `alpha≥25` 的连通块，剔除水印带（底边 8%）与碎点后按联合包围盒裁剪；
- 缩放：**RGBA 预乘 + LANCZOS**，避免透明边缘出现黑/白描边；
- 尺寸策略：装饰/金币等单帧图按**“高度锚”**输出（新图高 = 旧贴图高，宽随主体比例），
  因此场景里节点 `scale` 不变、显示高度不变，无需重摆；
- 角色四姿态统一画布高/宽并水平居中 → 切姿态时画面不跳动；
- 血条按像素绿色判据自动拆成 `rim(描边)` / `fill(填充)` 两张，并镂空内芯避免双层。

> ⚠️ Cocos 中 Sprite 默认 `sizeMode = CUSTOM`，PNG 像素尺寸改变后 **contentSize 不会自动跟随**，
> 故替换素材后需把每个引用了该贴图的 Sprite `contentSize` 同步为新 PNG 尺寸
> （本项目 12 处 tree、各处装饰/角色/血条均已同步，相关参数记录于 `temp/new_assets_dims.json`）。
> 处理前快照在 `temp/textures_backup/`，方便回滚。

---

## 九、可扩展点 / 已知取舍

- 怪物近身不主动伤害玩家：`PlayerController.takeDamage / die`（受击闪红、扣血、死亡动画）已就绪，接入攻击判定即可形成完整“受伤—死亡—结算”。
- 当前各姿态为单帧静态 SpriteFrame，可替换为序列帧动画（`animation` 组件或代码换帧）提升流畅度。
- 刷怪、攻击、速度等全部为 `@property` 可调参数，便于在编辑器里做数值调优。
- 暂未含音效、胜负结算 UI 与数据上报（4399 渠道一般需要），可作为后续增量。

---

## 十、工程目录速览

```
.
├─ assets
│  ├─ scenes/            Splash.scene / Main.scene
│  ├─ scripts/           Splash / PlayerController / MonsterController /
│  │                     GameManager / GameUI / GoldCoin / Obstacles
│  ├─ prefabs/           Skeleton.prefab / GoldCoin.prefab
│  └─ textures/          bg / decorations / monster / player / item / ui
├─ tools/preprocess/     replace_new_assets.py（素材透明底处理管线）
├─ profiles/  settings/  编辑器/构建配置（start_scene 入口在此）
├─ temp/                 new_assets_dims.json、textures_backup/ 等过程产物
└─ build/
   ├─ web-mobile-004/    最新可试玩静态包（含 Play.command / Play.bat）
   └─ web-mobile-…       历史构建（web-desktop 等）
```
