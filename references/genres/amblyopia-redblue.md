# 弱视红蓝训练（同视 / 融合 / 立体视）

> 本文是生成规范，不是医学建议。**颜色和几何纪律是这类游戏的硬规矩**——不是审美问题，是训练本体。

流程照旧（读 skill → `set_plan` → TASK.md 清单驱动 → 冒烟交付）；本文只管这一类游戏**专属**的东西。
四类玩法是**四个不同结构**，不是一个游戏的换皮——先按 §0 路由定类型，再照对应章节施工。

---

## 0. 提示词路由（先看这个）

| 提示词里出现 | 做哪一类 | 去哪节 |
|---|---|---|
| 同视 / 同时视 | 双眼信息互补（闭上任一只眼任务就做不成） | §5.1 |
| 融合 + 集合 / 辐辏 / 交叉 / "往近看" | 融合·集合（左眼左半 / 右眼右半 → 交叉视差 → 显近） | §5.2 |
| 融合 + 散开 / 发散 / "往远看" | 融合·散开（对眼交换 → 非交叉视差 → 显远） | §5.3 |
| 只说"融合"，没给方向 | 默认做集合；**方向在入口选**——预览面板对融合游戏给「（集合）/（散开）」两条目，游戏内不轮换 | §5.2 / §5.3 |
| 立体 / 立体视 / 深度知觉 | 深度判断类玩法（默认 = 同深度匹配接物；提示词指定了玩法就按提示词） | §5.4 |
| 弱视 / 斜弱视训练，但没说类型 | 默认做**同视**（康复阶梯第一级），TASK.md 里写明可换 | §5.1 |
| 精细训练（穿珠 / 描红 / 找细节） | **精细类**：透明背景路线（给物理光栅留透光层）——不戴镜 | `amblyopia-fine.md` 全文 |

提示词没提训练语境的（普通"做个打怪游戏"）**整篇不用**，按 `references/genres/` 里别的类型走。

## 1. 一分钟速览

| 要素 | 规矩 |
|---|---|
| 红层色 | `0xff0000`（#FF0000）——只有戴红片的那只眼能看到 |
| 蓝层色 | `0x0066ff`（#0066FF）——只有戴蓝片的那只眼能看到 |
| 共用层 | UI / 装饰 / 文字 / 卡背等正常画——**颜色不设限**；**背景默认透明**（同视 / 融合；立体视不要求——§4.3） |
| 红片戴哪只眼 | 默认左（左红右蓝）；**融合类**暂停页必备配镜两钮：左红右蓝 / 左蓝右红（`addLensButtons` §3.1 ⑪）；提示词可指定 |
| 对象画法 | 训练对象 = AI 帧**默认通道化**（彩图×通道，细节保留，§4.1）；双眼对象 = 同一帧两份副本，按视差摆 |
| 视差符号 | **立体类**：d > 0 = 交叉 = 更近；d < 0 = 非交叉 = 更远（与镜片布局无关，见 §3.1）。**融合类**：d 是**恒正的幅值**——方向 = 半→眼分配（集合 = 左眼左半 / 散开 = 对眼交换，§5.2）；两方向屏幕摆位相同 |
| 玩法 | 同视 / 融合·集合 / 融合·散开 / 立体视，四选一 |
| 素材底线 | **未经通道化的彩图不许进画面**（tint 是 source-in 只留 alpha——直塞 = 白丢细节；原样贴 = 串眼）——AI 帧一律走 `channelTexture`（§3.1 ⑨）；**背景默认透明、不出图**（同视 / 融合；§4.3） |

## 2. 颜色与镜片纪律（硬规矩）

1. **训练对象只用两个颜色**：红层 `AN.RED`、蓝层 `AN.BLUE`（§3 常量，全项目只许从这里引用；
   别处出现 `ff0000` / `0066ff` 字面量算违规）。
2. **共用层与背景**：共用层（UI / 装饰 / 文字 / 卡背）正常画，**颜色不设限**。**背景**：**同视 / 融合默认
   透明**——**背景层不画**（整幅底 / 大面都不画；场景元素不受限）、不进 AI 出图，除非用户提示词指定了背景色（改用指定色平涂，程序化
   画、色值写死）；**立体视不要求**——背景按游戏设计走。详见 §4.3。
3. **亮暗只走配平常量**：两眼亮度失衡时用 `AN.balance`（§3.1 ⑩）压亮侧——**只许 ≤ 1**、
   口径是**压好眼**（差眼侧保持 1）；不要手调 `setAlpha`、不要换色值、不要加发光
   （bloom 会把颜色洗歪）。
4. **画面纪律**：色块大而整（滤片下细线、小字、低对比全糊）；**不许频闪**（这类训练尤其禁，
   节奏感用大小和速度做）；少用大面积高饱和红蓝紧邻的闪烁特效。
5. **校准屏**（做进暂停页，§6）：一个纯红方块 + 一个纯蓝方块 + 一行说明"闭右眼：只有红块亮
   → 红片戴左眼"。治疗师（或家长）用它验证佩戴对不对；**融合类**再按对应配镜钮（§6.2）设成一致。
6. **两套配色：红蓝（默认）与红青**——本类玩法（同视 / 融合 / 立体）**两套都允许**，按用户
   实际眼镜配套选择：红层恒为红片眼的颜色，"蓝层"是另一眼层的统称（红蓝镜 = 蓝、红青镜 = 青）；
   **配色与镜片配套、禁止混用**（红蓝镜配红青的色值 = 分视失效）。色值表与
   `colorForEye(eye, scheme)` 见 §3。

## 3. 渲染机制：`js/anaglyph.js`（分视绘制唯一入口）

**所有"该只被某只眼看到"的东西都经过这个模块画**——不要在各个场景里手写 tint、手拼副本。

### 3.1 生理眼模型（颜色与几何正交）

两个易错点先钉死：

- **几何按"生理眼"（left / right）定义**，与红片戴哪只无关——翻镜片布局时几何自动正确；
- **颜色由镜片布局换算**（`colorForEye`）——翻布局只影响颜色，不影响几何。

```js
// js/anaglyph.js —— 全部分视绘制走这里
const AN = {
  RED: 0xff0000,     // #FF0000 红层（红片眼可见）
  BLUE: 0x0066ff,    // #0066FF 蓝层（蓝片眼可见）
  redEye: 'left',    // 红片戴哪只眼：'left' | 'right'（暂停页两钮切换；默认左 = 左红右蓝）
  scheme: 'redblue', // 分视配色：'redblue'（红蓝镜，默认）| 'redcyan'（红青镜）
};

// ★ 两套配色与镜片配套，禁止混用（红蓝镜配 redcyan 色值 = 分视失效）。
const SCHEMES = {
  redblue: { red: AN.RED, other: AN.BLUE },     // #FF0000 / #0066FF（默认）
  redcyan: { red: 0xff0000, other: 0x00ffff },  // #FF0000 / #00FFFF
};

// ① 给"某只生理眼"的内容配颜色（布局翻转时自动对调）
//    scheme 省略 = AN.scheme；'redblue' 为默认配色
function colorForEye(eye, scheme) {
  const s = SCHEMES[scheme || AN.scheme] || SCHEMES.redblue; // 未知 scheme 回落默认
  return eye === AN.redEye ? s.red : s.other;                // red = 红片眼那一层
}

// ② 视差 d（px）→ 左右眼副本的横向偏移（立体类用符号表深度；融合类 d 恒正、只当幅值）
//    d > 0：左眼副本右移、右眼副本左移 → 交叉视差 → 目标更近
//    d < 0：反向            → 非交叉     → 更远
function offsetsForEye(d) { return { left: d / 2, right: -d / 2 }; }
// ④ 单眼对象：只该被某只生理眼看到的内容（同视类主角 / 障碍 / 立体 / 融合用）
//    AI 帧 → 通道贴图（⑨；可半裁 'l'/'r'/'t'/'b'）；配平（⑩）在这落地——⑥/⑦ 走它自动覆盖
function addEyeImage(scene, srcKey, x, y, eye, px = 128, half) {
  return scene.add.image(x, y, channelTexture(scene, srcKey, srcKey, eye, px, half))
    .setAlpha(balanceForEye(eye));
}

// ⑤ 双眼对象壳：两份副本必须永远同逻辑坐标（改位置只许走 move/setD）
function makePair(a, b, x, y, d) {
  const pair = {
    a, b, d, x, y,
    setD(nd) { pair.d = nd; return pair.layout(); },
    move(nx, ny) { pair.x = nx; pair.y = ny; return pair.layout(); },
    layout() {
      const o = offsetsForEye(pair.d);
      a.setPosition(pair.x + o.left, pair.y);
      b.setPosition(pair.x + o.right, pair.y);
      return pair;
    },
    destroy() { a.destroy(); b.destroy(); },
  };
  return pair.layout();
}

// ⑥ 立体对象（几何 + 深度 · 双眼同形）：d = 视差 px，只许落 ⑫ 带内；d=0 也禁——无深度的
//    "屏面感"内容走共用层颜色（§2.2）。运行中改 d（pair.setD）不经断言，约定：改也走带内值。
//    接住判定用 pair.x/y（逻辑坐标）
function addStereo(scene, key, x, y, d, px = 128) {
  if (d < AN.disp.min || d > AN.disp.max)
    console.error(`[anaglyph] addStereo 视差 ${d}px 越带（${AN.disp.min}~${AN.disp.max}）—— ` +
      '立体类所有 d（含深度层）都落带内（§5.4）');
  return makePair(addEyeImage(scene, key, 0, 0, 'left', px),
                  addEyeImage(scene, key, 0, 0, 'right', px), x, y, d);
}

// ⑦ 融合对（融合类）：两眼各看一半，融合成完整对象图；方向 = 半→眼分配（§5.2）
//    dir='in'  集合：左眼左半 / 右眼右半 → 交叉视差 → 显近
//    dir='out' 散开：对眼交换         → 非交叉   → 显远
//    两方向屏幕摆位相同（d 恒正、两半往中间合 d）；方向只决定哪只眼看到哪半。
function addFusionPiece(scene, key, x, y, d, px = 128, split = 'lr', dir = 'in') {
  const A = split === 'lr' ? 'l' : 't', B = split === 'lr' ? 'r' : 'b';
  const L = dir === 'out' ? B : A;   // 左眼看到的半
  const R = dir === 'out' ? A : B;   // 右眼看到的半
  return makePair(addEyeImage(scene, key, 0, 0, 'left', px, L),
                  addEyeImage(scene, key, 0, 0, 'right', px, R), x, y, d);
}

// ⑧ 配镜布局 setter（⑪ 的两钮组件调它）：设到指定布局 + 重启场景重建一切
//    （别手工逐个改色，会漏；两钮中生效的一颗高亮——对照 AN.redEye）
function setEyeLayout(scene, redEye) {   // 'left' = 左红右蓝；'right' = 左蓝右红
  if (AN.redEye === redEye) return;      // 按的是已生效的那颗：不动
  AN.redEye = redEye;
  scene.registry.set('anRedEye', AN.redEye);
  scene.scene.restart();
}

// ⑨ 通道贴图：抠好的彩图帧 → "只被某眼看到"的副本（彩图 × 本眼通道，内部细节保留）
//    half: undefined 整形 | 'l'/'r' 左/右半 | 't'/'b' 上/下半个——half 半裁 = 融合类（§5.2）用
//    红片眼=(R,0,0)；另一眼=(0,0.4G,B)——40% 绿补偿是 #0066FF 色值自带的（multiply 自动
//    得到，不要特殊代码）；红青镜的另一层天然是 (0,G,B)，同一条路。
//    合成顺序：填色 → multiply 源图 → destination-in 源图。
//    ⚠ multiply 在透明区会垫底填色 → 必须 destination-in 蒙回 alpha。
//    锚点与剪影化逐字一致：等比 min 缩放、底部居中、px 默认 128（两条路线换着用也不挪位置）。
function channelTexture(scene, outKey, srcKey, eye, px = 128, half) {
  const col = colorForEye(eye);
  // cache key 必须带解析后的色值：换布局（⑧）重启后必须拿到新色副本
  // （Phaser 纹理不随 scene.restart 销毁，不带色值会一直用翻布局之前的旧副本）
  // half 也要进 key：左右半是两张独立贴图，不带会串缓存
  const texKey = 'ch:' + outKey + ':' + eye + ':' + px + ':' + col.toString(16) +
    (half ? ':' + half : '');
  if (scene.textures.exists(texKey)) return texKey;
  const src = scene.textures.get(srcKey).getSourceImage();
  const k = Math.min(px / src.width, px / src.height);
  const w = src.width * k, h = src.height * k;
  const tex = scene.textures.createCanvas(texKey, px, px);
  const ctx = tex.getContext();
  ctx.save();
  if (half === 'l') ctx.rect(0, 0, px / 2, px);
  if (half === 'r') ctx.rect(px / 2, 0, px / 2, px);
  if (half === 't') ctx.rect(0, 0, px, px / 2);
  if (half === 'b') ctx.rect(0, px / 2, px, px / 2);
  if (half) ctx.clip();                             // 半裁：只画该半（融合类，§5.2）
  ctx.fillStyle = '#' + col.toString(16).padStart(6, '0');
  ctx.fillRect(0, 0, px, px);
  ctx.globalCompositeOperation = 'multiply';        // ×通道
  ctx.drawImage(src, (px - w) / 2, px - h, w, h);
  ctx.globalCompositeOperation = 'destination-in';  // 蒙回 alpha（清掉 multiply 垫的底色）
  ctx.drawImage(src, (px - w) / 2, px - h, w, h);
  ctx.restore();
  tex.refresh();
  return texKey;
}

// ⑩ 两眼配平：两眼亮度失衡（素材偏暖常见）时用常量压亮侧——只许 ≤1（压暗），
//    口径 = 压好眼（差眼侧保持 1）；不许改色值 / 手调 setAlpha / 加 bloom（色值一改分视就废）
AN.balance = { left: 1, right: 1 };
function balanceForEye(eye) { return AN.balance[eye]; }

// ⑪ 配镜两钮（半固化组件 · 融合类暂停页必备）：左红右蓝 / 左蓝右红——标签、顺序、行为、
//    高亮全写死，只有位置和尺寸可调；用法就一行：addLensButtons(this, W / 2, H * 0.6)
function addLensButtons(scene, x, y, { w = 132, h = 56, gap = 16 } = {}) {
  const mk = (redEye, label, dx) => {
    const on = AN.redEye === redEye;               // 生效的一颗：满亮 + 深框，另一颗压暗
    scene.add.rectangle(x + dx, y, w, h, 0xffffff)
      .setStrokeStyle(on ? 4 : 2, on ? 0x111111 : 0x888888)
      .setAlpha(on ? 1 : 0.55)
      .setInteractive({ useHandCursor: true })
      .on('pointerdown', () => setEyeLayout(scene, redEye));     // ⑧（按已生效那颗：不动）
    scene.add.text(x + dx, y, label, { fontSize: '20px', color: '#111111', fontFamily: 'sans-serif' })
      .setOrigin(0.5).setAlpha(on ? 1 : 0.55);
  };
  mk('left', '左红右蓝', -(w + gap) / 2);
  mk('right', '左蓝右红', +(w + gap) / 2);
}

// ⑫ 视差带：立体对象（⑥）的 d 只许落带内——约 60cm 视距下 40~60px = 可融合且训练
//    信号够；越带 = 重影（>60）或深度感不足（<40）。d = 0 一律禁——"屏面感"内容走
//    共用层颜色（§2.2）。越带 console.error → 冒烟判红（跑一局就打回）。
AN.disp = { min: 40, max: 60 };
```

**两套配色与镜片配套，禁止混用**：`redblue`（红蓝镜，默认，#FF0000 / #0066FF）与
`redcyan`（红青镜，#FF0000 / #00FFFF）。`AN.scheme` 不设 = `'redblue'`；红片左右可选
（`setEyeLayout` 只换颜色、不动几何）。

### 3.2 绘制纪律

1. **逻辑坐标 ≠ 屏幕坐标**：玩家持有的位置是 `pair.x / pair.y`；屏幕上两份副本各自偏移。
   碰撞 / 判定 / 计分**只许用逻辑坐标**——别拿 `getBounds()` 去撞。
2. **改位置只许走 `pair.move / setD`**：两份副本必须同一时刻同一逻辑坐标（错一帧融合就破），
   不许单独 `setPosition` 其中一份。
3. 共用层（UI / 说明文字等）与场景元素正常画，颜色不设限（§2）；**背景层**默认透明、不画（§4.3）；不用进这个模块（唯一例外：配镜两钮 ⑪——它要动 `AN.redEye`）。
4. 对象贴图按 key 缓存（模块已做）；同屏几十个红蓝对象没有性能问题。
5. 配镜布局 = `setEyeLayout(scene, 'left' | 'right')`（两钮各自设值 + 重启场景；只换颜色、
   不动几何）。镜片布局与难度档一起存 localStorage，重开恢复。

## 4. 训练对象出图（AI 图 · 转换规格）

训练对象 = 这局玩法 / 主题要用的对象。**背景除外——同视 / 融合默认透明、不出图；立体视按设计（§4.3）**。
出不出图、出什么图看挡位（分档表见 `spec-design.md`）；**出图 / 抠图 / 切帧流水线照旧**，
本文只加转换（都在游戏侧几十行 canvas 里做）：

### 4.1 通道化（默认）

**训练层对象 = 通道化（默认）**：抠好底的帧 → `channelTexture`（§3.1 ⑨）——彩图 × 本眼通道，
**细节保留**在亮度里（花纹 / 明暗都看得见，不是一块纯色）；逐帧动画每帧过一遍通道化：

```js
// 训练层对象一行接上；配平 / 缓存都随模块走，px 默认 128
const img = addEyeImage(scene, 'crate-f1', x, y, 'left');
```

### 4.2 剪影化（备选）

**训练层对象 = 剪影化（备选）**：抠好底的帧 → 白剪影贴图（alpha 填白）→ 照常 tint 红 / 蓝。
适用：入门玩法、靠外形辨认对象、色块大而整：

```js
// 帧贴图 → 白剪影。等比缩放、底部居中（贴图锚点对得上地面/顶部落点）
function silhouetteTexture(scene, outKey, srcKey, px = 128) {
  const tex = scene.textures.createCanvas(outKey, px, px);
  const ctx = tex.getContext();
  const src = scene.textures.get(srcKey).getSourceImage();
  const k = Math.min(px / src.width, px / src.height);
  const w = src.width * k, h = src.height * k;
  ctx.drawImage(src, (px - w) / 2, px - h, w, h);
  ctx.globalCompositeOperation = 'source-in';   // 只保留 alpha
  ctx.fillStyle = '#fff';
  ctx.fillRect(0, 0, px, px);
  tex.refresh();
  return outKey;
}
```

剪影化会丢内部细节（熊猫只剩轮廓）——靠**外形**区分对象，一局里轮廓差异要大
（猫 vs 球；不要两只圆脸）。逐帧动画也可以这么走，但每帧都要剪影化。

### 4.3 背景（同视 / 融合：默认透明，不出图；立体视：不要求）

**同视 / 融合**：**背景默认透明**——**背景层不画**：纯色 / 渐变 / 风景底、底图、铺满的大面
**都不画**、**不进 AI 出图**（覆盖 `spec-design.md` 分档表"背景整幅转出图"的档位口径）。取向同 `amblyopia-fine.md`
§1 的透明背景纪律——目的不是光栅透光，而是让训练画面干净：滤片下大面背景最容易添乱，
AI 背景还容易出接缝 / 色偏。

**场景里的元素（月亮、星星、太阳、飞机这类主题 / 装饰对象）不受本条限制**——照常规画、
颜色不设限；是玩法对象的照常走训练对象路线（§4）。

透明怎么落（与精细类同款，写法一字不差见 `amblyopia-fine.md` §1）：
`transparent: true` + `backgroundColor: 'rgba(0,0,0,0)'`、页面壳背景 transparent、
画布无背景——**背景层什么都不画**。

**唯一例外**：**用户提示词里指定了背景色**（如"白底""深灰背景"）→ 用该色**平涂**整幅——
程序化画、色值写死；无渐变、无纹理、不交给 AI 发挥。

**立体视**：本条**不适用**——背景**不要求透明**，按游戏设计走（透明 / 平涂 / 照
`spec-design.md` 分档表出图都行）；共用层颜色纪律照旧见 §2。

### 4.4 禁项（红线）

**不许把未经通道化的彩图直接当训练对象**：tint 是 source-in 只留 alpha——直塞 = 白丢细节
（结果和剪影一样）；原样贴 = 绿 / 黄串眼。训练对象两条路：`channelTexture`（默认）、
剪影化（备选）。

## 5. 四类玩法

公共骨架（四类共用）：

- 逻辑 / 渲染分离：`logic.js` 纯函数（判定 / 计分 / 档位）+ 场景摆对象；随机全走
  mulberry32 种子（同 `casual-platformer.md` 的口径）；
- 参数进 `plan.gameplay`（各节表格 → 落实进 TASK.md，治疗师改档只改这一处）；
- 冒烟 / 探针 / 交付形态照 `verification.md` §7 与 SKILL.md 既有口径，本节只写差异；
- 难度自适应：连对升档、连错降档；档位存 localStorage，重开恢复。

### 5.1 同视（同时视）

**训练内核**：双眼各看不同内容——闭上任一只眼任务都做不成（破抑制、建立双眼同时视）。

两个变体，挑一个做：

**变体 A · 躲避**

- 主角只被**一眼**看到（默认弱视眼层，`amblyopicEye`）；障碍只被**另一眼**看到，从顶部下落。
- 左右移动躲障碍；碰撞 = 逻辑圆判定（主角半径 + 障碍半径），用逻辑坐标。
- 命 3 条；存活到时限（60s）结算得分。
- 难度：障碍速度 / 出现间隔 / 是否双列同时。

**变体 B · 送笼**（入门首选，无失败惩罚）

- N 只小动物（弱视眼层，3 种不同对象，图案差异要大）散在屏上方；图案对应的筐（另一眼层）在下方。
- 拖动物到"图案相同"的筐 → 入筐得分；错了弹回。
- 判定（纯函数）：拖放点与筐逻辑坐标距离 < 筐半径 且 图案匹配。
- 难度：动物数 / 筐数（加干扰筐）/ 拖拽距离。

参数（`plan.gameplay.simultaneous`）：

| 字段 | 说明 | 默认 |
|---|---|---|
| variant | `'dodge'` / `'sort'` | sort |
| amblyopicEye | 弱视眼（任务/趣味内容给它）：'left' / 'right' | 与 redEye 相同 |
| speed | 障碍下落速度 px/s（dodge） | 70 |
| spawnGap | 障碍出现间隔 ms（dodge） | 1400 |
| lives | 命数（dodge） | 3 |
| targetCount | 动物数（sort） | 3 |

**专项验收点**：闭任意一只眼 → 任务信息缺一半（dodge 只见主角或只见障碍；sort 只见动物或只见筐）。

### 5.2 / 5.3 融合：集合 与 散开

两类共用一个实现（**屏幕摆位完全相同**）——方向 = 半→眼分配：集合 = 左眼左半 / 右眼右半（交叉视差、显近）；
散开 = 对眼交换（非交叉、显远）。方向**在入口选定**：预览面板给融合游戏摆「（集合）/（散开）」两条目
（`?dir=in` / `?dir=out`）；plan 里的值只是无参数打开时的回落。

**训练内核**：双眼各看到**半个**图形，把两半融合成一个完整图形——练双眼融合与辐辏范围。

**两半怎么来**：同一对象图、两种裁剪（`addFusionPiece`，§3.1 ⑦）。集合：左眼只看到左半、右眼只看到右半；
散开：对眼交换（左眼看右半）。屏幕上"红半 + 蓝半"拼成完整图形、两半往中间合 d（两方向同摆位）。

**四行呈现表**（红鸟左 = 红色半形的可见位置在左）：

| 方向 | 配镜 | 期望（用户） | 推演 | ✓ |
|---|---|---|---|---|
| 集合 | 左红右蓝 | 红鸟左、蓝鸟右 | 红=左眼='l'乘位 +d/2（内容居中偏左）、蓝='r'乘位 −d/2（偏右） | ✓ |
| 集合 | 左蓝右红 | 蓝鸟左、红鸟右 | 蓝=左眼='l' → 左 | ✓ |
| 散开 | 左红右蓝 | 蓝鸟左、红鸟右 | 红=左眼='r'乘位 −d/2（内容偏右）、蓝='l'乘位 +d/2（偏左） | ✓ |
| 散开 | 左蓝右红 | 红鸟左、蓝鸟右 | 蓝=左眼='r' → 右；红='l' → 左 | ✓ |

**入口方向读取**（放 `main.js`）：

```js
// 方向来自面板入口（?dir=in|out）；无参数 / 非法值回落 plan（抄进 main.js 的 FUSION.dir，默认 'in'）
function fusionDir() {
  const q = new URLSearchParams(location.search).get('dir');
  return (q === 'in' || q === 'out') ? q : (FUSION.dir === 'out' ? 'out' : 'in');
}
// 开场读一次，传给每个 addFusionPiece：addFusionPiece(scene, key, x, y, d, px, split, fusionDir())
```

两个变体，挑一个做：

**变体 A · 补全点击**（入门）

- 屏上一个融合目标，玩家点击"完整图形的中心"判定；点击处与逻辑坐标距离 < 对象半径 → 成功。
- 连对：d 按 dStep 升一档（训练更大的集合 / 散开需求）；失败或超时：降一档。
- 每轮换对象图 + 换裁剪方向（'lr' 左右 → 偶尔 'tb' 上下）。

**变体 B · 找完整**

- 3~5 张卡片；每张卡 = 红半来自某对象图 + 蓝半来自某对象图（对象图各自随机组合）。
- **只有一张卡两半同源**（融合后是完整图形）——选它得分。
- 干扰卡：两半来自**不同对象图**（图案差异要大），或同对象图但两半错位摆放。
- 闭一只眼只看到一半信息 → 选不出"哪张完整"：这是"只有融合能做对"的硬保证。

参数（`plan.gameplay.fusion`）：

| 字段 | 说明 | 默认 |
|---|---|---|
| dir | `'in'` 集合 / `'out'` 散开（**无 'alt'**；面板两入口覆盖两方向，此值只是无参数打开时的回落） | 'in' |
| variant | `'complete'` / `'find'` | complete |
| dStart / dStep / dMin / dMax | 视差起点 / 步长 / 下限 / 上限（px） | 16 / 8 / 8 / 64 |
| roundMs | 每轮时限 | 5000 |
| cards | 卡片数（find） | 4 |
| shapePx | 对象显示尺寸 px | 128 |

**专项验收点**：闭单眼只能看到半个图形（A）/ 无法判断哪张完整（B）；方向呈现符合 §5.2 四行表
（集合 = 集合配对；散开 = 对眼交换；构造性验证见 §7）。

### 5.4 立体视

**训练内核**：同一对象图画两份、视差 d 表达深度；**玩法必须让深度参与判断**（对准 / 选层）。

**玩法不限于接物**：提示词指定了别的玩法（射击 / 躲避 / 节奏类…）→ 按提示词做，本节纪律照套
——深度全落 ⑫ 带内、层差恒 > `tol`、判定纯函数 node 可测；`plan.gameplay.stereo` 的
`dBase / dStep / tol` 保留、其余参数随玩法定。动态玩法：x/y 随便动，**深度 d 的变化要平滑**
——融合依赖视差的连续性，别在层间瞬跳（换层加个短过渡）；工程与验收照 §6 / §7 不变。

**默认规格 · 接物**（单局 60~90 秒）：

- 篮子固定在底部，自带深度 `dBase`；玩家左右移动篮子（拖动 / 方向键）。
- 水果从顶部下落，每颗随机深度：`d_i ∈ { dBase, dBase + D, dBase − D }`——只有 `dBase` 层的该接。
- 接住判定（**纯函数，node 可测**）：

```js
function judgeFruit(f, basket, stereo) {
  if (Math.abs(f.x - basket.x) > stereo.catchR) return 'miss-x';     // 横向没对上
  if (Math.abs(f.d - basket.d) > stereo.tol) return 'miss-depth';    // 深度不同 → 弹开
  return 'catch';
}
```

- 接对：得分 + 连击；接错深度：弹开（提示音 / 动画反馈，不扣命）；横向错过 / 落地：错过。
- 波次：每波 6 颗、同屏最多 3 颗；每波结算 → 连对升档、连续错降档。
- **防闭眼穷举**：错深度弹开"无害"，但波次限时 + 水果落下即错过——逐个试错的成本是分数。
- **深度全落带内**：篮子 50、水果三层 40 / 50 / 60——全在 ⑫ 的 40~60（约 60cm 视距可融合 + 训练信号够；越带模块直接报错，§3.1 ⑥）。

**难度档**（三层一起动）：深度差 D：10 → 5 px（D 越小越难；受带宽限制 D 最大 10）；下落速度：60 → 110 px/s；tol 固定 3（恒 < 最小 D=5，防错层误判接住）。

参数（`plan.gameplay.stereo`）：

| 字段 | 说明 | 默认 |
|---|---|---|
| dBase | 篮子深度（带中值，40~60 内） | 50 |
| dStep | 深度层间距（起始档） | 10 |
| tol | 深度判定容差 px（必须恒 < 最小 D） | 3 |
| speed | 下落速度 px/s | 60 |
| wave | 每波水果数 | 6 |
| catchR | 接住横向半径 px | 34 |

**专项验收点**：

- `judgeFruit` 单测：同层同 x = catch；差 x = miss-x；`|Δd| ≤ tol(3)` = catch、之外 = miss-depth（错层偏移最小 D=5 恒 > tol）。

## 6. 工程与交付（差异清单）

在既有工程纪律（逻辑 / 渲染分离、种子随机、探针、无频闪、交付形态）
之上，红蓝训练游戏再加：

1. **模块落位**：`game/js/anaglyph.js`（§3）；训练对象图走素材流水线（§4，`game/sprites/` 由 process_strip 生成）；**背景**：同视 / 融合默认透明、不进流水线（§4.3；立体视按设计）。
2. **暂停页必备**：校准色块（§2.5）+ 重开——**四类玩法都要有**；**配镜两钮（左红右蓝 /
   左蓝右红）只有融合类（§5.2 / §5.3）要摆**——其他三类不摆；模块自带 `addLensButtons`，
   一行摆上（§3.1 ⑪）。游戏内不加配平 / 单眼预览这类向导组件（它们属于校准工具的事）。
3. **玩法说明.txt 追加**（交付形态的一部分）：
   - "请佩戴红蓝眼镜：红片戴__眼（融合类：与本游戏内配镜两钮设成一致）"；
   - 建议视距 ≈60cm、屏幕高度与视线平齐；
   - 单次训练建议 15~20 分钟（用"再来一局"多轮）；
   - "请遵医嘱 / 在治疗师指导下使用；佩戴后重影明显时调低难度档"；
   - 「打开方式：从 GameForge 预览面板选『（集合）/（散开）』进入对应方向；直接双击 index.html 走预设方向」。
4. **难度自适应**：连对升档 / 连错降档；档位 + 镜片布局一起存 localStorage。
5. **收尾冒烟照旧**：照 SKILL.md 的收尾口径跑。

## 7. 红蓝专项验收清单（写进 TASK.md）

- [ ] 训练层色值只来自 `AN.RED` / `AN.BLUE`（冒烟自动跑颜色闸：字面量只允许出现在 anaglyph.js）
- [ ] 背景按 §4.3：同视 / 融合默认透明（背景层不画、无 AI 背景图；场景元素不受限；用户指定背景色 → 平涂该纯色）；立体视按设计
- [ ] 闭单眼信息不完整：同视缺任务信息 / 融合只见半形 / 立体无深度线索（人工项，做法写进玩法说明）
- [ ] 视差方向构造验证：集合入口（?dir=in）左眼副本中心在右眼副本中心的右侧（交叉 → 显近）；散开入口（?dir=out）相反（非交叉 → 显远）
- [ ] 配镜钮只重新着色（融合类）：翻后颜色对调、几何不变；两入口 × 两配镜的四行呈现照 §5.2 表
- [ ] 判定纯函数跑过 node 测试（融合匹配 / 同视碰撞 / 立体：默认规格 = judgeFruit）
- [ ] 无频闪；色块大而整（没有 2px 以下的线条元素）
- [ ] 校准屏 + 玩法说明（佩戴 / 视距 / 时长）齐
- [ ] `gameplay` 参数齐（§5 各表），治疗师改档只改这一处
- [ ] 立体类所有 `addStereo` 的 d（含深度层）在 40~60 内（⑫ 视差带）。
