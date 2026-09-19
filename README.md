# (1) Data storage analysis
_Exported on 9/20/2026 at 03:26:52 GMT+8 from Cursor (3.15.19)_

---

**User**

https://github.com/LinB203/GenManip/blob/lb/docs/training/production_v2/releases/selected79_20260917/README.md 请你参考这个文档，帮我分析一下数据的储存目录、数量、格式等信息

---

**Cursor**

我先把这份 README 拉下来，再对照仓库里的路径和描述，整理数据目录、数量和格式。

GitHub 页面返回了 404，我改从本地仓库和 raw 链接继续找这份文档。

仓库可能是私有的。我接着用 `gh` 拉这份文档，并在本机目录里找对应数据。

共享盘里有 `lb123`，很像这份文档对应的目录。我接着在那边查 README 和数据路径。

文档已经在本机找到了。我先读 README，再核对它引用的数据目录。

文档已经对上。接下来核对磁盘上的真实目录、条数和文件格式，避免只按 README 转述。

这份 `selected79_20260917` 文档对应的是 **2026-09-17 冻结的 79 条 production_v2 试训选集**。GitHub 上的 `LinB203/GenManip` 是私有仓库，公开链接会 404；同一份文档和数据都在当前服务器上，已按磁盘内容核对过。

## 1. 存在哪

| 用途 | 路径 |
|---|---|
| 项目根 | `/remote-home/share/lb123/projs/GenManip` |
| **数据根** | `/remote-home/share/lb123/projs/GenManip/outputs/training_factory/production_v2` |
| 权威生产清单 | `.../production_v2/review_inventory.json` |
| 冻结交付文档 | `.../docs/training/production_v2/releases/selected79_20260917/` |
| 单条轨迹 | `.../production_v2/episodes/T{k}/{id}/` |

交付目录里有这些辅助文件：

- `manifest.json`：冻结的 79 条清单（含 SHA、条数、验收路径）
- `files_training.txt`：最小训练传输清单（601 行）
- `files_with_videos.txt`：再加 MP4（886 行）
- `files_full_audit.txt`：完整 episode 目录（79 行，含物理日志和私有审计）
- `read_episode.py`：白名单读取器

**不要直接 `glob episodes/`。** 磁盘上实际有 **206 个 episode 目录**，里面混着失败 attempt、诊断样本和资格验证数据。正式要用的只有 `manifest.json` / `review_inventory.json` 里的 79 条。我核对过：这 79 条在磁盘上全部存在，`episode.json` 和 `trajectory.npz` 齐全。

编号规则：`T13_v2_000003_a000`

- `T13`：任务
- `v2`：当前随机化生产管线
- `000003`：生产索引 3
- `a000`：attempt 0（不是新场景号，也不是帧号）

同一生产索引可能有多个 attempt，必须按清单选已验收的那个。例如 T1 索引 0 选的是 `T1_v2_000000_a001`，不是 `a000`。

## 2. 数量

这是 **小批量试训选集**，不是正式最终训练集：

- 目标：每任务 5 条，共 90 条（生产索引 `0–4`）
- 实际选中：**79 条**
- 母计划：每任务 100 条，主种子 `2026091202`
- 机器人：单臂 Franka Panda + Panda Hand
- 状态：全部 `auto_qualified=true`，但 `human_review_status=pending`、`training_eligible=false`
- 文档明确：负责人允许这 79 条做 **试训**，不等于补签了逐条人审

按任务分布：

| 任务 | 内容 | 条数 | 动作数 N | 时长（N/30） | 历史 H |
|---|---|---:|---|---|---:|
| T1 | 杯子放盘子 | 4 | 1031–1555 | 34–52s | 0 |
| T2 | 指定颜色方块入盒 | 5 | 1031–1389 | 34–46s | 0 |
| T3 | 打开抽屉/柜门 | 5 | 725–798 | 24–27s | 0 |
| T4 | 关闭柜门 | 5 | 875–931 | 29–31s | 0 |
| T5 | 推长方块到标记 | 5 | 1150–1619 | 38–54s | 0 |
| T6 | 按按钮 | 5 | 767–1091 | 26–36s | 0 |
| T7 | 瓶子放盘子 | 5 | 1197–1399 | 40–47s | 0 |
| T8 | 杯子放入盒 | 4 | 1015–1324 | 34–44s | 0 |
| T9 | 抓起指定物体 | 5 | 476–822 | 16–27s | 0 |
| T10 | 打开盒盖 | 5 | 534–692 | 18–23s | 0 |
| T11 | 球放入开口盒 | 5 | 1017–1251 | 34–42s | 0 |
| T12 | 推盘子到标记 | 5 | 1339–1485 | 45–50s | 0 |
| T13 | 按演示顺序放两个方块 | 3 | 1540–1664 | 51–55s | 60 |
| T14 | 记忆两个容器并提起复位 | 2 | 1568–1754 | 52–58s | 90 |
| T15 | 记住闪烁方块并放入盒 | 1 | 1619 | 54s | 90 |
| T16 | 抓起放回并计数 | 5 | 1728–1782 | 58–59s | 0 |
| T17 | 恢复抽屉/柜门记忆状态 | 5 | 687–1786 | 23–60s | 60 |
| T18 | 按木棍演示顺序按蓝按钮 | 5 | 1394–1766 | 46–59s | 252 |

合计：

- **92,862** 个动作
- **92,941** 个执行状态（每条都是 N+1）
- **2,010** 个历史时刻
- **284,853** 张 JPEG（执行 278,823 + 历史 6,030，三路相机）
- 动作总时长约 **51.6 分钟**（三路相机不要乘 3）

尚未纳入、因此不在本次交付里：T1 `{4}`、T8 `{1}`、T13 `{2,4}`、T14 `{2,3,4}`、T15 `{1,2,3,4}`。

体积（文档估算，十进制）：

- 训练最小选集：约 **6.63 GB**（JPEG + 核心元数据/NPZ）
- 带视频：约 **7.65 GB**
- 完整 episode 更大：120Hz 物理、接触、诊断日志很多。抽查 `T1_v2_000002_a000` 全目录约 0.24 GB，训练核心约 0.07 GB；带历史的 `T13_v2_000003_a000` 全目录约 0.49 GB，训练核心约 0.12 GB。普通 RGB imitation learning 不必复制完整目录。

## 3. 目录与格式

以 `episodes/T13/T13_v2_000003_a000/` 为例，磁盘结构和文档一致：

```text
episode.json          # UTF-8 JSON 元数据
trajectory.npz        # NumPy 压缩数组，禁止 pickle
COMPLETE.json         # 完成标记 + 核心文件 SHA
images/{obs_camera,obs_camera_2,realsense}/000000.jpg ...
history/{同上}/        # 仅部分记忆任务有图
videos/*.mp4          # H.264, yuv420p, 30fps, 无音频
hidden/manifest.json  # teacher/答案，禁止喂模型
events.jsonl          # 评测事件
audits/.../production_acceptance.json
# 以及大量 120Hz 物理/接触/诊断文件
```

核心格式：

| 文件 | 格式 | 训练用法 |
|---|---|---|
| `episode.json` | UTF-8 JSON，schema `training_factory.episode.v1` | 指令、相机路径、关节定义、白名单 |
| `trajectory.npz` | NumPy `.npz`，`allow_pickle=False` | `action`/`qpos` 等时序数组 |
| JPEG | RGB，`uint8[480,640,3]`，帧号 6 位补零 | 优先读 JPEG，不要用视频 seek |
| MP4 | H.264 / yuv420p / 30fps / 无音频 | 浏览用；有 H 时另有 `*_full.mp4` |
| JSONL | 每行一个 JSON | 审计/物理日志，不是训练输入 |

时间对齐（最关键）：

```text
E[0] --action[0]--> E[1] --action[1]--> ... --action[N-1]--> E[N]
```

- 控制/相机都是 **30Hz**；Isaac 物理 **120Hz**（每步 4 个子步）
- `action` 是 **9 维绝对关节位置目标**（前 7 维 rad，末 2 维手指 m），不是速度、力矩、delta，也不等于 `qpos[t+1]`
- 最后一帧 `E[N]` 没有监督 action
- 历史 H 只有图像，没有 `qpos`/`action`；拼进训练张量时要对 H 和末帧做 mask

`trajectory.npz` 公共字段（79 条都有，S=N+1）：

| 字段 | shape | dtype |
|---|---|---|
| `action` | `(N,9)` | float32 |
| `qpos` | `(S,9)` | float32 |
| `qvel` | `(S,9)` | float32 |
| `camera_pose_world_wxyz` | `(S,3,7)` | float32 |
| `state_time_s` / `action_time_s` | `(S,)` / `(N,)` | float64 |
| `state_index` / `image_state_index` | `(S,)` | int64 |
| `action_valid` | `(N,)` | bool |
| `source_action_index` | `(N,)` | int64 |

三路相机都是 640×480：

- `obs_camera` / `obs_camera_2`：固定外相机，典型 fx=fy=450
- `realsense`：手腕相机，外参随运动变，典型 fx=fy=288

Policy 白名单只有：`instruction`、三路 RGB、执行阶段 `qpos`，有 H 时再加历史图像。`object_pose`、`button_q`、`hidden/`、`events.jsonl`、验收结果都只能当审计/oracle，不能当模型输入。

## 4. 怎么读

同一台机器上直接用数据根即可。官方读取器：

```bash
DATA=/remote-home/share/lb123/projs/GenManip/outputs/training_factory/production_v2
RELEASE=/remote-home/share/lb123/projs/GenManip/docs/training/production_v2/releases/selected79_20260917

python "$RELEASE/read_episode.py" --root "$DATA" --verify-only
python "$RELEASE/read_episode.py" --root "$DATA" \
  --episode T13_v2_000003_a000 --frame 0 --ack-experimental
```

注意：仓库里原来的 `training_factory/data/policy_dataset.py` 默认要求人审 approved，直接拿会拒绝这批 pending 数据。要用交付目录的 `read_episode.py`，并显式 `experimental=True`。

文档也提醒：79 条里 T14 只有 2 条、T15 只有 1 条，不够做可靠泛化评测；验证集应按整条 episode/场景种子切，不要把同一轨迹相邻帧拆进 train/val。

---

**User**

<timestamp>Friday, Sep 18, 2026, 9:30 PM (UTC+8)</timestamp>
<system_notification>
The following task has finished. If you were already aware, ignore this notification and do not restate prior responses.

<task>
kind: shell
status: aborted
task_id: 275818
title: Search share filesystem for selected79 data
tool_call_id: call-489e56ea-a8a9-4d3c-835a-4c0ca814bfa1-21
fc_ee01153a-b515-9649-ab06-0a5d5938b2a9_1
output_path: /root/.cursor/projects/remote-home-share-jyc/terminals/275818.txt
</task>
</system_notification>

<timestamp>Friday, Sep 18, 2026, 9:30 PM (UTC+8)</timestamp>
<system_notification>
The following task has finished. If you were already aware, ignore this notification and do not restate prior responses.

<task>
kind: shell
status: aborted
task_id: 275819
title: Find selected79 README under lb123
tool_call_id: call-aefe3146-0d12-4bc8-9818-83805b0ac303-29
fc_ce5ad6b8-7c69-9862-b5cd-d367d5ffbde7_1
output_path: /root/.cursor/projects/remote-home-share-jyc/terminals/275819.txt
</task>
</system_notification>
<user_query>Briefly inform the user about the task result and perform any follow-up actions (if needed). If there's no follow-ups needed, don't explicitly say that.</user_query>

---

**Cursor**

刚才有两条后台搜索被中止了，不影响结论：数据文档和 79 条选集都已经在 `/remote-home/share/lb123/projs/GenManip` 里核对完。

---

**User**

https://github.com/Physical-Intelligence/openpi 请你参考openpi的Fine-Tuning Base Models on Your Own Data章节，帮我总结一下微调需要什么样的数据，以及上面这个production_v2试训数据满不满足条件

---

**Cursor**

我先对照 OpenPI 的微调文档和刚核对过的 production_v2 选集，看它要求什么数据、这批试训数据够不够。

README 里只写了要转成 LeRobot。我再把转换脚本和训练 config 拉下来，确认图像、状态和动作具体要什么。

OpenPI 微调**不要求原始格式必须是 LeRobot**，但训练前必须转成 LeRobot，并且每一步都要对齐：**图像 + 本体状态 + 动作 + 语言指令**。`selected79` 的内容大体够做一次试训转换，**还不能直接喂给 OpenPI**。

## OpenPI 微调要什么数据

官方流程是三步：转 LeRobot → 写 DataConfig / Inputs-Outputs → 算 norm stats 再训。数据要求可以从转换脚本和 `LiberoInputs` 里归纳成这些：

**每条 episode、每个训练步必须有：**

| 字段 | 作用 | 官方示例 |
|---|---|---|
| RGB 图像 | 视觉输入 | 至少 1 路第三人称 + 可选腕部；π₀ 有 3 个槽位（`base` / `left_wrist` / `right_wrist`），缺的用全零图 pad |
| `state` | 本体感觉 | LIBERO 是 8 维；DROID/Franka 是 7 维关节 + 1 维夹爪 |
| `actions` | 监督标签 | 与当前 observation 对齐；模型再按 `action_horizon`（常见 10 或 16）切 action chunk |
| `task` / `prompt` | 语言指令 | 从 LeRobot 的 `task` 读成 `prompt` |

另外还有几条硬约定：

1. **必须先转成 LeRobot**，不是 HDF5 / NPZ / 自定义 episode 目录直接训。
2. 图像存 `uint8` RGB；训练时会 resize-pad 到 **224×224**。原始分辨率不限。
3. `state` 是 proprio，`actions` 是动作；二者语义要在训练和推理时一致。
4. π₀ 预训练关节动作通常是 **delta**（相对 chunk 起点），夹爪保持 **absolute**，范围 **\[0,1\]**（0 开、1 关）。LIBERO 原数据已经是 delta；若你的数据是绝对关节角，config 里要加 `DeltaActions`。
5. 频率要自洽：LIBERO 示例 10 fps，DROID 15 Hz，Franka 预训练约 20 Hz。没有强制 30 Hz。
6. 要先跑 `compute_norm_stats.py`。若机器人接近预训练机型（DROID/Franka），可以尝试复用预训练 norm stats。

OpenPI **没有规定最少多少条**。LIBERO 官方微调是 3 万步；自定义 DROID 示例写的是「不到 10 小时」再转 LeRobot。79 条、约 50 分钟，只够通管线和看 loss，不够当正经微调集。

## production_v2 满不满足

结论：**语义上基本够转，格式和动作定义还不匹配，规模只够试训。**

**已经具备、可以直接映射的：**

- 英文 `instruction` → OpenPI 的 `prompt` / `task`
- 执行阶段 `E[t]` 的三路 JPEG + `qpos[t]` + `action[t]`，时间对齐正确
- 第三人称 `obs_camera` + 腕部 `realsense`，正好对应 π₀ 的 `base` / `left_wrist`；`obs_camera_2` 可作第二外相机，或丢掉/填零
- 640×480 RGB 可以，训练时会缩到 224
- 单臂 Franka Panda，和 DROID/Franka 预训练家族最近
- 有现成 `read_episode.py`，能按白名单吐出 `instruction / images / qpos / action`

**不满足、必须先改的：**

1. **不是 LeRobot。** 现在是 `episode.json` + `trajectory.npz` + JPEG。OpenPI 读不了，要自己写转换脚本。
2. **动作空间不一致。** 这批是 **9 维绝对关节位置**（7 臂 + 2 手指，单位 m，0–0.04）。OpenPI/DROID 常见是 **8 维：7 关节 + 1 维夹爪 \[0,1\]**，关节还经常是 delta 或速度。不能把 9 维直接截成 7 维，也不能把两根手指原样当 OpenPI 夹爪。
3. **30 Hz vs 10/15/20 Hz。** 能训，但推理控制频率必须和训练一致；想复用 DROID/Franka 的预训练统计量，通常还要降采样。
4. **记忆历史 H 对不上。** T13/T14/T15/T17/T18 有一段只有图像、没有 `qpos/action` 的 H。标准 LeRobot/`add_frame` 假设每帧都有 action。按执行段直接转会丢掉记忆上下文；硬拼进去又会造假标签。
5. **量太小、任务不齐。** 79 条，T15 只有 1 条、T14 只有 2 条。官方自己也说这是试训授权，不是正式训练集。

对照表：

| OpenPI 需要 | production_v2 | 是否够 |
|---|---|---|
| LeRobot 数据集 | 自定义 episode 目录 | 否，要转换 |
| 语言指令 | 有英文 `instruction` | 是 |
| 第三人称 RGB | `obs_camera`（还有 `_2`） | 是 |
| 腕部 RGB | `realsense` | 是 |
| proprio | 9 维 `qpos` | 要压成 7+1 或改 config |
| action 标签 | 9 维绝对关节目标 | 要改语义/维度 |
| 帧对齐 `s[t]→a[t]` | 执行段满足 | 是 |
| 无动作的历史观察 | 有 H，OpenPI 默认没有 | 记忆任务不够 |
| 规模 | 79 条 / ~52 分钟 | 只够试训 |

## 实际该怎么用这批数据

如果目标是 **把 OpenPI 微调管线跑通**：可以。只用 `manifest.json` 里的 79 条，转 LeRobot，映射相机和语言，把两指合成 1 维夹爪，关节做 delta，再算自己的 norm stats。先拿 T1/T2/T9 这类 H=0 的短任务做 smoke test。

如果目标是 **得到能泛化的 π₀.₅ 策略**：不够。差在数量、任务覆盖、动作定义，以及记忆任务的历史观察。

不要把 `episodes/` 下全部 206 个目录都转进去，里面有失败 attempt。也不要把 `hidden/`、`object_pose`、验收字段写进 observation。

---

**User**

action这里，哪里表明pi05需要8维了

---

**Cursor**

你说得对，我先把 π₀.₅ 的 `action_dim` 定义和 DROID 的 8 维动作空间分开核对，避免把机器人动作维数说成模型要求。

**π₀.₅ 并不要求 8 维。** 上次把「Franka/DROID 机器人动作维数」和「π₀.₅ 模型 `action_dim`」混在一起了。

## 模型侧：π₀.₅ 默认是 32 维

`Pi0Config` 里默认：

```python
action_dim: int = 32
```

DROID 微调 config 写得很明确：

```python
model=pi0_config.Pi0Config(
    pi05=True,
    action_dim=32,  # pi05 is trained with 32-dim actions
    action_horizon=16,
)
```

`pi05_libero` 也没改 `action_dim`，用的就是这个 32。训练时用 `PadStatesAndActions(model_config.action_dim)` 把机器人动作 **pad 到 32**，不是截成 8。

写了 `action_dim=8` 的是 **π₀-FAST-DROID**，不是 π₀.₅：

```python
name="pi0_fast_droid"
model=pi0_fast.Pi0FASTConfig(action_dim=8, action_horizon=10)
```

## 8 维来自哪：机器人动作空间，不是 π₀.₅

8 维只出现在 Franka/DROID 的 **机器人接口** 定义里。

`docs/norm_stats.md`：

> For 7-DoF robots (e.g. Franka), we use the first 7 dimensions of the action space for the joint actions, and the 8th dimension for the gripper action.

DROID 转换脚本：

```python
"actions": {"dtype": "float32", "shape": (8,)}  # 7D joint velocity + 1D gripper
```

推理时 `DroidOutputs` 再从模型输出里切回前 8 维：

```python
return {"actions": np.asarray(data["actions"][..., :8])}
```

LIBERO 则是切前 **7** 维。所以「数据集动作维数」随机器人变，π₀.₅ 网络宽度仍是 32。

## 对 production_v2 的含义

需要对齐的是 **Franka 常用的 7 关节 + 1 夹爪**，不是「π₀.₅ 必须 8 维」。这批数据是 9 维绝对关节（7 臂 + 2 手指），和 DROID/Franka 的 8 维接口仍不一致；转完后 π₀.₅ 还会 pad 到 32。

---

**User**

我直接用production_v2的9维训pi05不行吗

---

**Cursor**

架构上 9 维可以 pad 进 π₀.₅。我先核对 padding、loss 和预训练动作空间，看直接拿 production_v2 的 9 维训会不会有实质问题。

**可以。** π₀.₅ 不要求 8 维，9 维绝对关节可以直接当训练标签；网络宽度是 32，OpenPI 会把不足的维零填充。

```python
class PadStatesAndActions:
    """Zero-pads states and actions to the model action dimension."""
    def __call__(self, data):
        data["state"] = pad_to_dim(data["state"], self.model_action_dim, axis=-1)
        data["actions"] = pad_to_dim(data["actions"], self.model_action_dim, axis=-1)
```

`Pi0Config.action_dim` 默认就是 32。LIBERO 微调是 7 维 pad 到 32，推理再切回前 7 维；你这边同样：**9 维 → pad 32 → 推理取前 9 维**，再交给 GenManip 的 9 维绝对关节控制器。这比硬压成「7 关节 + 1 夹爪」更贴这批数据。

真正不能省的是下面几件事，不是维数本身：

1. **还是要转成 LeRobot，并写自己的 Inputs/Outputs。** 「用 9 维」只决定 `state`/`actions` 的最后一维是 9，不能把 `episode.json` + `npz` 直接丢进 `train.py`。
2. **不要复用 DROID/Franka 的预训练 norm stats。** 那套假定第 8 维是 \[0,1\] 夹爪，第 9 维根本没有。对这 9 维重新跑 `compute_norm_stats.py`。
3. **前 7 维语义接近，后 2 维是新的。** `qpos/action` 的 0–6 仍是 Panda 关节（rad）；7、8 是两根手指（m，0–0.04），和预训练「单夹爪 0/1」不是一回事。微调能学，只是夹爪这部分几乎不能吃预训练。
4. **这批是绝对位置目标，不是 delta。** OpenPI 注释写过：数据若是绝对关节角，应对关节做 `DeltaActions`，夹爪保持绝对。9 维可以是：前 7 维 delta、后 2 维绝对。也可以 9 维都当绝对目标训，但推理控制必须和训练时同一套。
5. **频率仍是 30 Hz。** 训练和闭环都用 30 Hz，不要按 DROID 的 15 Hz 去切。

所以：试训完全可以就用 9 维；这正是 production_v2 的监督标签。需要改的是数据格式和 config，不是先把动作改成 8 维。

