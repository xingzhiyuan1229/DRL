# 燃机冷端 DRL 论文 · 全程操作手册（最终版 · 零基础照做）

> 本手册已按你的实际情况定制。从上往下做，每步都有"命令/点击 → 预期结果 → 完成标志"。
> 以 `#` 开头是说明，不用输入。Windows 用「Anaconda Prompt」执行命令。

---

## 〇、你的确定条件（已确认，后续都以此为准）

| 项目 | 情况 |
|---|---|
| 操作系统 | Windows（**不用管 WSL/Linux**，全程 Windows） |
| 硬件 | AMD Ryzen 5 4600H（6核12线程）/ 15.4 GB 内存 / **NVIDIA GTX 1660 Ti 6GB** |
| 已装软件 | Dymola（已安装）、VS Code（已安装） |
| Modelica 库 | **ClaRaPlus v1.7.1**（含 ClaRa_DCS、ClaRa_Grid）、ThermalSystems(TIL)、HVACLib、Cooling、HumanComfortLib、FluidDynamicsLib 等全套 |
| 冷却塔 | 库内已有组件（**具体路径待你 Search 确认后填入本手册**） |
| 编程基础 | 无 |
| 现场数据 | 约 11 月之后可获取 |
| 开题 | **10 月底**；硕士《文献综述与开题报告》≥8000 字（不含图表），宋体小四、1.25 倍行距，A4 |

## 全程总览

```
装环境(第1周) → 补基础+跑通RL例子(第2周) → 熟悉ClaRa/冷却塔(第3周)
→ 搭简化冷端模型(第4周) → 写开题报告(第5周) → 10月底开题
→ 11月数据到位后：精细建模+在线辨识 → 导出FMU → 训练SAC → 对比验证 → 写论文
```

---

# 第一部分 环境安装

## 1.1 安装 Miniconda

1. 打开 https://docs.conda.io/en/latest/miniconda.html ，下载 `Miniconda3 Windows 64-bit`；
2. 双击安装，勾选 "Add Miniconda3 to my PATH"（若不可勾则保持默认，之后用开始菜单「Anaconda Prompt」）；
3. 验证（打开 Anaconda Prompt）：
   ```bash
   conda --version
   ```
   预期显示 `conda 24.x.x`。

**设置国内加速（一次性，逐条执行）：**
```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --set show_channel_urls yes
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

## 1.2 VS Code（你已装，只需补扩展）

打开 VS Code → 左侧扩展（四个方块）→ 搜索安装：`Chinese (Simplified)`、`Python`、`Jupyter`、`Markdown Preview Enhanced`。

## 1.3 创建项目环境

```bash
conda create -n coldend python=3.10 -y
conda activate coldend
```
命令行最前面出现 `(coldend)` 即成功；**以后每次工作先执行 `conda activate coldend`**。

## 1.4 安装 GPU 版 PyTorch（配合你的 1660 Ti）

```bash
pip install torch --index-url https://download.pytorch.org/whl/cu121
python -c "import torch; print(torch.cuda.is_available(), torch.cuda.get_device_name(0))"
```
预期打印：`True NVIDIA GeForce GTX 1660 Ti`。
（若装过 CPU 版 torch，此命令会自动替换为 CUDA 版。）

## 1.5 安装其余工具包

```bash
pip install "stable-baselines3[extra]" gymnasium fmpy numpy pandas matplotlib scipy jupyter
```

## 1.6 总验证

```bash
python -c "import torch, stable_baselines3, fmpy, gymnasium; print('ALL OK', torch.cuda.is_available())"
```
预期打印 `ALL OK True`。

## 1.7 Dymola 侧确认（重要，做完发我）

1. 打开 Dymola，菜单 `File → Libraries`，确认已加载 `ClaRaPlus v1.7.1`；
2. 库窗口点 **Search**，搜 `tower`（或 `CoolingTower`），找到冷却塔组件，**把完整路径填入这里**：
   `冷却塔组件路径 = ____________________（例如 HVACLib.xxx.CoolingTower）`
3. 若该组件/库自带 Examples，双击打开 → Simulate，看出塔水温曲线，先混个脸熟。

**第一部分完成标志**：1.6 打印 `ALL OK True`；1.7 找到冷却塔路径。

---

# 第二部分 学习路线（约 2–3 周，够用即可，边做边查）

## 2.1 Python 基础（约 5–7 天）

- 视频（零基础首选）：B 站搜「Python 零基础 小甲鱼」或「黑马 Python 入门」，看前 1/3（变量、列表/字典、判断循环、函数、文件、类的概念）；
- 文字备用：廖雪峰 https://liaoxuefeng.com/books/python/introduction/index.html （看到面向对象）；
- 速查：菜鸟教程 https://www.runoob.com/python3/python3-tutorial.html
- 目标：看懂并能改写简单代码。

## 2.2 NumPy / Pandas（2–3 天）

B 站搜「NumPy Pandas 快速入门」，目标：会数组运算、会用 pandas 读 CSV/Excel 并画图（11 月处理现场数据用）。

## 2.3 强化学习（5–7 天，理解概念，不推公式）

1. B 站搜「李宏毅 强化学习」，看 Q-learning、Policy Gradient、Actor-Critic；
2. 中文书：Datawhale《EasyRL 蘑菇书》https://datawhalechina.github.io/easy-rl/ ；
3. 英文概念参考：OpenAI Spinning Up https://spinningup.openai.com/ （含 SAC）；
4. 必须吃透 5 个词：**环境、状态、动作、奖励、策略**；理解"智能体靠试错最大化累计奖励"。

## 2.4 Modelica / ClaRa（5–7 天，与建模结合）

- 官方互动入门：https://mbe.modelica.university/ （方程、连接器、组件，前几章）；
- Dymola 自带例子：`File → Libraries → Modelica`，学会"拖组件、连线、设参数、Check、Simulate、画曲线"；
- ClaRaPlus 自带示例与官方文档（官网 https://www.claralib.com/ ）。

**第二部分完成标志**：能写简单 Python；能口述 RL 在干什么；Dymola 能跑通自带模型并画曲线。

---

# 第三部分 跑通两个强化学习例子（手把手）

## 3.1 例子一：CartPole（验证环境）

新建文件夹 `D:\coldend`，VS Code 打开，新建 `test_cartpole.py`：
```python
import gymnasium as gym
from stable_baselines3 import PPO

env = gym.make("CartPole-v1", render_mode="human")
model = PPO("MlpPolicy", env, verbose=1)
model.learn(total_timesteps=10_000)
model.save("ppo_cartpole")
env.close()
```
终端运行 `python test_cartpole.py`。预期：弹出小窗口显示杆子平衡，日志中 reward 上升。

## 3.2 例子二：Pendulum + SAC（连续控制，与冷端同构，重点）

新建 `test_sac_pendulum.py`：
```python
import gymnasium as gym
from stable_baselines3 import SAC

env = gym.make("Pendulum-v1")
model = SAC("MlpPolicy", env, verbose=1,
            policy_kwargs=dict(net_arch=[256, 256]))
model.learn(total_timesteps=30_000)
model.save("sac_pendulum")

obs, _ = env.reset()
R = 0.0
for _ in range(200):
    a, _ = model.predict(obs, deterministic=True)
    obs, r, term, trunc, _ = env.step(a)
    R += r
print("累计奖励:", R)
env.close()
```
预期：训练后累计奖励明显变好（越接近 0 越好）。

**看懂**：算法和神经网络由库提供（`MlpPolicy` 自动建网）；你没写算法，只调用了它。后面只是把 Pendulum 换成你的 FMU 冷端。

---

# 第四部分 Dymola 冷端建模（ClaRaPlus，分模块边建边验）

> 原则：先零件后系统、先稳态后动态；每建一个模块就用设计数据/解析解单独验证。

## 4.1 建模与验证顺序

1. **循环水泵 + 管段**：录入 H-Q、η-Q 曲线 + 相似定律变频；验证流量、压头；
2. **管网**：供回水管道（流动惯性、热惯性/输配延迟）、阀门阻力；验证流量分配与压损；
3. **机力通风冷却塔**：用 1.7 找到的库内组件（焓差法、变速风机），集水池水量/热平衡；验证出塔水温；
4. **凝汽器**：壳侧凝结（有限体积两相）、管壁热惯性、水侧换热，可加不凝结气体；验证凝汽压力与端差；
5. **汽轮机末级/排汽**：拟合厂家背压—功率修正曲线；
6. **边界与系统集成**：燃机—余热锅炉负荷指令、环境干湿球/大气压、启停旁路蒸汽与疏水接口；连成整机。

## 4.2 校验口径

- 稳态：THA 及多个部分负荷点，背压/流量/供回水温/出塔水温误差 1%~2%；
- 动态：升降负荷或启停指令，比较背压、水温响应趋势与滞后时间；
- 报错先用 Dymola 的 `Check` 定位，再逐步加复杂度。

---

# 第五部分 关键参数在线辨识（孪生精度保障）

1. **清洁系数在线辨识**：HEI 公式求导法/标准传热系数法，实时反算水侧清洁系数；
2. **漏空气量在线辨识**：饱和蒸汽温度应达值与实际值对比 + BP 网络（或基准压力法），并与水侧脏污分离；
3. RBF/SVM 修正"负荷、流量偏离设计值"的残差；
4. 写成可周期更新模型参数的模块（让孪生跟得上现场，并为 DRL 提供退化状态输入）。

---

# 第六部分 导出 FMU 并与 Python 打通

## 6.1 Dymola 导出 FMU

1. 整机 `Check` 无误并成功 Translate/Simulate；
2. 菜单 `File → Export → FMU…`（版本不同或在 Simulation 菜单）；
3. 选 **FMI 2.0、Co-Simulation（自带求解器）**，导出 `.fmu`。

## 6.2 FMPy 图形界面检查（新手推荐）

```bash
conda activate coldend
python -m fmpy.gui
```
打开 `.fmu`，查看全部输入/输出变量名，点 Simulate 直接跑并绘图。

## 6.3 命令行查看变量

```python
from fmpy import dump
dump("cold_end_twin.fmu")
```
记录要读写的变量名后，练习"手动设泵频率 → 推进仿真 → 读背压"，确认链路通畅。

---

# 第七部分 封装环境 + 训练 SAC

1. 打开已交付的 `drl_cold_end_template.py`；
2. 按 6.3 真实变量名填写所有 `TODO`：`_read_obs()`（状态+归一化）、`step()`（动作写入）、`_reward()`（奖励+约束）；
3. 环境自检：
   ```python
   from stable_baselines3.common.env_checker import check_env
   check_env(ColdEndEnv())
   ```
4. 训练 SAC，看 `ep_rew_mean` 是否收敛；不收敛查第九部分；
5. 保存策略并在独立工况 `predict` 测试。

**奖励函数（你唯一要自己写的核心）：**
```
奖励 = (背压改善的发电增量 − 循泵电耗 − 风机电耗) × 电价 − 安全约束惩罚
约束 = 背压上限、最小循环水流量(防汽蚀)、出塔水温上限等
```

---

# 第八部分 基线对比与验证

1. 三类基线：①现状规则（台数+风机PID）②净收益枚举法（北仑方法）③MPC（可选，CasADi）；
2. 指标：背压、冷端厂用电率、气耗/供电煤耗、净收益；
3. **消融实验**（证明协同有用）：只调泵 → 泵+风机 → 泵+风机+阀门；
4. 验证等级按条件选：模型在环（必做）→ 历史数据回放（强烈建议）→ 现场影子模式（争取）→ 硬件在环/闭环投控（非必须，且需安全审批）。

---

# 第九部分 开题报告（10 月底，≥8000 字）

## 9.1 学校固定七部分

1. 选题背景及意义；
2. 国内外研究现状综述；
3. 研究内容、研究目标及拟解决的关键问题；
4. 研究方案及可行性分析；
5. 预期成果及创新点；
6. 论文工作计划；
7. 主要参考文献。

## 9.2 字数分配建议（约 8500 字，留余量）

| 部分 | 建议字数 |
|---|---|
| 一 背景及意义 | 1000 |
| 二 研究现状综述 | 3000 |
| 三 内容/目标/关键问题 | 1300 |
| 四 方案及可行性 | 1500 |
| 五 成果及创新点 | 700 |
| 六 工作计划 | 500 |
| 七 参考文献 | 不计入（单列） |

格式：题目小三号黑体居中；正文宋体小四（12pt）、1.25 倍行距；A4。

## 9.3 开题前五周计划

| 周次 | 任务 | 周末标志 |
|---|---|---|
| 周1 9/23–9/29 | 装环境、GPU torch；Python 入门 | 打印 True + 显卡名 |
| 周2 9/30–10/6 | CartPole、Pendulum+SAC；Modelica 入门 | 两例子出结果 |
| 周3 10/7–10/13 | ClaRaPlus 自带例子；定位并跑通冷却塔 | 自带模型出曲线 |
| 周4 10/14–10/20 | 搭简化冷端，跑出背压 | 简化整机闭环 |
| 周5 10/21–10/27 | 写开题报告初稿（≥8000 字） | 初稿完成 |
| 缓冲 10/28–10/31 | 修改、补图、定稿 | 可提交 |

---

# 第十部分 常见报错速查

| 现象 | 解决 |
|---|---|
| `conda 不是内部或外部命令` | 用「Anaconda Prompt」，或重装勾选 PATH |
| pip 下载慢/超时 | 加 `-i https://pypi.tuna.tsinghua.edu.cn/simple` |
| `torch.cuda.is_available()` 为 False | 更新显卡驱动；确认装的是 cu121 版命令 |
| FMU 打不开/加载报错 | 确认 FMI 2.0、64 位；升级 fmpy |
| `doStep` 报错 | 初始化/变量名问题；先用 fmpy.gui 跑通 |
| 奖励不收敛、越训越差 | 状态与奖励归一化、降低惩罚量级、增加步数 |
| Dymola 不收敛 | Check 定位；先降复杂度、加合理初值 |

---

# 第十一部分 11 月之后总体时间计划（约 6–8 个月）

| 时间 | 任务 | 产出 |
|---|---|---|
| 11 月上–12 月 | 用现场数据精细建模、在线辨识 | 高保真孪生 |
| 12 月–1 月 | 导出 FMU、Python 打通 | 可调用 FMU |
| 1 月–3 月 | 封装环境、训练 SAC、基线对比 | 控制策略与对比 |
| 3 月–4 月 | 回放/影子验证、数据回灌（配合循泵变频投运） | 验证结论 |
| 4 月–5 月 | 撰写论文、准备中期与答辩 | 学位论文 |

---

## 附录 A 关键网址汇总

- Miniconda：https://docs.conda.io/en/latest/miniconda.html
- 清华镜像：https://mirrors.tuna.tsinghua.edu.cn/
- VS Code：https://code.visualstudio.com/
- PyTorch 安装选择：https://pytorch.org/get-started/locally/
- Stable-Baselines3 文档：https://stable-baselines3.readthedocs.io/
- Stable-Baselines3 仓库：https://github.com/DLR-RM/stable-baselines3
- Gymnasium 文档：https://gymnasium.farama.org/
- FMPy 仓库：https://github.com/CATIA-Systems/FMPy
- FMI 标准：https://fmi-standard.org/
- Modelica 入门：https://mbe.modelica.university/
- EasyRL 蘑菇书：https://datawhalechina.github.io/easy-rl/
- OpenAI Spinning Up：https://spinningup.openai.com/
- 廖雪峰 Python：https://liaoxuefeng.com/books/python/introduction/index.html
- 菜鸟教程 Python：https://www.runoob.com/python3/python3-tutorial.html
- ClaRa/ClaRaPlus 官网：https://www.claralib.com/
- TLK-Thermo：https://www.tlk-thermo.com/

## 附录 B 待你确认/回填的清单

1. 冷却塔组件完整路径（1.7）= ____________
2. 论文最终题目（暂定/自定）= ____________
3. 导师姓名、本人姓名（开题报告封面用）= ____________
4. 开题具体日期 = ____________
