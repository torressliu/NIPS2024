We are deeply grateful to you for recognizing the presentation and novelty of our work, this positive feedback is greatly encouraging. Moreover, your objective advice promote our further improvement of this work.

If you think the following response addresses your concerns, we would appreciate it if you could kindly consider raising the score.
### Weaknesses
**W1：The comparisons in the experiment are with some simple and intuitive baselines. While the authors mention that no specific solution exists for this problem, it would still be helpful to discuss related work in other research fields. It would be beneficial to include more tasks to validate the effectiveness of MARS.**

感谢您有价值的建议，您的提议促使我们进一步完善了我们的投稿。

Baselines: 我们分别从机器人学习领域选择了最新的多步决策全监督方法ACT[1] 和强化学习领域较为相关的延迟-MDP领域方法（也可以侧面缓解Intermitted MDP 场景） DCAC[2] DOMDP[3]作为基线。

Benchmarks: 在虚拟场景方面，我们增加了更加困难的四个DMC suit控制任务，相比于gym 的位移静态mujoco控制场景，DMC需要多关节仿生机器人连贯地运动起来，DMC也是最近在强化学习社区逐渐流行的新挑战环境。此外，我们增加了2个metaworld机械臂虚拟抓取任务，这两个任务也有很大的提升空间。因此也是主流的较为困难的任务。
在真实场景方面，我们修改了我们的机械臂抓取目标，将目标从6个稀疏分布的场景转化为6个较为密集分布的场景（即机械臂被要求容错率更低，无效抖动更容易碰倒待操作目标从而导致任务失败)。

Setting：为了更全面地评估方法的有效性，我们设置了两组不同交互间隔设定的实验，一组和论文中使用的常见间隔相同(B=8)，第二组设置成超长间隔 （B=16）。

Table 1. Win rate (Average of 10 runs):
| Method      | Scenario A (%)| Scenario B (%)| Scenario C (%)|
| :-----------: | :-----------: | :------------: | :-----------: |
| PPO with R1|$60$|$0$|$20$|
| PPO with R2 |$70$|$30$|$70$|
| PPO with R3 |$70$ |$60$|$70$|
| PPO with R4 |$70$ |$30$|$90$|
| **GBRL-PPO without reward** |$60$ |$50$|$80$|

Table 2. Win rate (Average of 10 runs):
| Method      | Scenario A (%)| Scenario B (%)| Scenario C (%)|
| :-----------: | :-----------: | :------------: | :-----------: |
| PPO with R1|$60$|$0$|$20$|
| PPO with R2 |$70$|$30$|$70$|
| PPO with R3 |$70$ |$60$|$70$|
| PPO with R4 |$70$ |$30$|$90$|
| **GBRL-PPO without reward** |$60$ |$50$|$80$|
实验结果表明，我们新增的基线中ACT的效果较为突出，但在绝大多数任务，我们的方法明显表现好于其他方法。

**W2: Minors in writing**

我们为我们的写作粗糙给读者带来的阅读不便诚挚的道歉，并根据您的提示，重新repolish我们的语法、单词拼写公式和符号。您细致的审稿意见大幅度地提升了我们的投稿质量。

### Questions
**Q1: Why does the input of the encoder include $s_{t:t+c}$? According to the description of the intermittent control task (Line 31: agents are unable to acquire the state sent by the executor), isn't $s_{t:t+c}$ inaccessible to the agent?**

您的提问很细致，我们在新版本中进行了进一步解释。在策略训练期间我们仅需要VAE中的解码器来配合智能体策略的训练，即在预训练完成后，编码器就不再需要了。而在预训练阶段，动作空间的构建与策略是无关的. 编码器所用的$s_{t:t+c}$ 并不是强化学习策略采集的，而是随机采样的初始数据集。VAE是自监督训练的，只要数据量足够就能表现出良好的效果，所以随机采样的数据集随机采样的状态转移不仅降低了成本而且包含更丰富的状态。

我们在一个mujoco场景设置了一组实验对比专家数据（TD3采集）和随机生成数据（random policy）对于VAE的有效性. 实验结果表明，对于固定的动作隐空间，随机采样的状态转移包含更丰富的状态，构建的空间更有效。

Table 2. Win rate (Average of 10 runs):
| Method      | Scenario A (%)| Scenario B (%)| Scenario C (%)|
| :-----------: | :-----------: | :------------: | :-----------: |
| PPO with R1|$60$|$0$|$20$|
| PPO with R2 |$70$|$30$|$70$|
| PPO with R3 |$70$ |$60$|$70$|
| PPO with R4 |$70$ |$30$|$90$|
| **GBRL-PPO without reward** |$60$ |$50$|$80$|

**Q2:Which variable is the Gaussian exploration noise added to, the decoded action sequence $u_t$, the latent variable $z_t$, or $\upsilon_t$?**

我们在$z_t$上增加高斯扰动。我们在一个mujoco场景上分别分析了您上述说几种加噪方式。

Table 2. Win rate (Average of 10 runs):
| Method      | Scenario A (%)| Scenario B (%)| Scenario C (%)|
| :-----------: | :-----------: | :------------: | :-----------: |
| PPO with R1|$60$|$0$|$20$|
| PPO with R2 |$70$|$30$|$70$|
| PPO with R3 |$70$ |$60$|$70$|
| PPO with R4 |$70$ |$30$|$90$|
| **GBRL-PPO without reward** |$60$ |$50$|$80$|

实验结果显示，在$u_t$和在$z_t$上加噪声效果相似，这是因为我们在构建隐空间时利用状态残差正则化项使其具有和原始动作空间相似的语义平滑性。此外，实验显示在 $\upsilon_t$上加噪声没有明显作用。
