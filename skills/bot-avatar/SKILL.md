---
name: bot-avatar
description: >-
  Use when a bot needs a new or redone profile avatar in the unified peeking
  anime style (still → seamless loop video → ≤1MB GIF → install on itself).
---
# Bot 头像（统一探头风循环 GIF）

每个 bot 用自己的会话和 desktop 跑完，最后装到**自己**身上。风格参考图在本 skill 文件夹：`style-ref-fullbleed.png`（主参考），`ref-frames/` 里是已装头像的首帧（只看画风，发色已被占用，不要照抄）。

## 1. 风格（必须全满足）
- 和 Chief of Staff 一样的探头二次元风：从画面底边往上探头，平涂 2D、柔和赛璐珞阴影、无硬描边；大号简单椭圆眼（默认纯黑，角色设定要求时可换色，如金瞳）、粉色腮红、齐刘海。不写实。
- 全出血：头发碰到左右两条边，不要白底上贴小图、不要边框。
- 头部一个**大元素**改变整体轮廓（兜帽、帽子、发髻、花环、或从道具后探头），一眼可认；标杆是 Coding bot 的兜帽加耳机。
- 专属背景色，不和其他 bot 撞色。不要文字。
- 镜头别贴太近：露出整个头顶轮廓，脸至少到眼睛下方，否则认不出是谁。

## 2. 静帧：Cohub gpt-image-2.5
```bash
D=/workspace/bot-avatars/<名字>; mkdir -p $D/still
cohub -s <你的 cohub space> generate "<英文 prompt>" -m gpt-image-2.5 \
  --image <本 skill 文件夹>/style-ref-fullbleed.png --inline \
  --param size=1024x1024 --param quality=high -o $D/still --json > $D/gen.json
```
- **只传统一风格参考图**。角色原图（写实立绘等）不要当 `--image`，会把画风带偏；角色特征（发色、发型、眼睛、眼镜等）用文字写进 prompt。
- prompt 里写明：构图和画风照参考图、不要照抄参考图发色、NO inset/border、hair touches left and right edges。
- 出 2–3 张（每张约 $0.05–0.24），用 Read 亲眼看，挑最像统一风格又认得出角色的。把候选拼成对比图给用户挑。
- 头发没碰到两边：把选中图从底部居中裁 85–90% 再放大回 1024，比重出便宜。

## 3. 循环视频：grok.com Imagine（见 grok-imagine skill，用 computerUse）
1. `cp still.png $D/upload-<名字>-$(date +%H%M%S).jpg`（Grok 会复用旧上传，必须新文件名）。
2. Video 模式 → 上传该文件，**确认缩略图是这张新图** → Add last frame 选同一张（首尾同帧才无缝）。
3. 6s、480p、音频关、尽量 1:1。不点 Share。
4. 运动 prompt：`Subtle seamless loop for a profile avatar: soft idle motion only, tiny hair sway, one slow gentle blink, <道具> barely moving. Camera locked, no zoom, no pan, no new objects, keep framing full-bleed edge to edge. First and last frame identical.`
5. 下载到 `$D/loop.mp4`。出成竖图就裁底部正方形：`ffmpeg -i loop.mp4 -vf "crop=iw:iw:0:ih-iw" -an loop-sq.mp4`。
6. Grok 步骤串行：开前 `free -h`，available ≥2.5Gi；做完关标签。

## 4. GIF：200×200、10fps、≤1MB
```bash
ffmpeg -t 6 -i loop.mp4 -vf "fps=10,scale=200:200:flags=lanczos,palettegen=max_colors=64:stats_mode=diff" pal.png
ffmpeg -t 6 -i loop.mp4 -i pal.png -lavfi "fps=10,scale=200:200:flags=lanczos[x];[x][1:v]paletteuse=dither=none" -loop 0 avatar.gif
```
超 1MB 改 `max_colors=48` 或 fps=8。抽中间帧 Read 看一眼。

## 5. 安装与交付
- 用自己的 UpdateAvatar 设为 avatar.gif，只装自己。核对自己 agent 目录里的 avatar 已更新。
- 把 GIF 发给用户，一句话说明角色、配色、头部大元素。

## 小改动别重跑
只改颜色（比如眼睛换金色）这类局部修改：用 PIL 在眼睛区域按暗度混色改像素，零花费、其他完全不变，再走第 3–5 步。
