We are immensely grateful to you for recognizing the importance and value of our approach. Your valuable suggestions inspire us to improve our work further. Based on your feedback, we analyzed the difference between our setting (intermittent MDP) and Delayed-MDP through a demo example and detailed explanation in the revised version. Additionally, we compared our method with several SOTA delayed MDP methods on multiple complex scenarios in the main experiment section. (6 more simulation tasks and 1 more real-world grasping task than the original version). 

If you think the following response addresses your concerns, we would appreciate it if you could kindly consider raising the score.
### Questions
**Q1: Compare Intermitted MDP and Delayed MDP: Why define intermitten MDP separately while delayed MDP is widely adopted in the community?**

Thanks for your in-depth question. Both the Intermittent MDP and the traditional Delayed MDP models aim to represent non-ideal (unstable) environments. However, these two approaches differ significantly in follow aspects：
- *Applicable scenarios* 延迟MDP主要关注网络高延迟导致的信息异步传输场景[1]。Intermittent MDP 关注的是信息传输瞬发（延迟不明显，且可以通过常用通信层面技术[2]进行有效缓解)但执行端和决策端之间交互信道会出现随机/有规律阻断的场景[3][4], 即信息直接丢失(无法用于策略训练). 这类场景主要存在于游戏NPC控制和交互高成本的机器人控制任务。
- *Trainging Data* 大多数延迟MDP默认所有数据最终都会到达，但获取方面存在时间上的异步,且每一个(s,a)对应的奖励是单独可获得的[5]（密集奖励）, 因此，这类设定的方法允许训练期间使用完整数据转移。然而, Intermittent MDP 将未及时传输的信息直接视为丢失(无法用于策略训练)，且奖励的反馈是一段动作序列的奖励总和（稀疏奖励，只能看到最后一个动作执行后的奖励，更符合现实）。
  - 我们在两个mujoco场景上观测了当无法在训练阶段使用密集状态转移时延迟MDP方法是否可以完成Intermitted MDP 任务。
  
  Table 1. Performance (Average of 5 runs):
  | Method      | Ninja | Chaser | Heist |
  | :-----------: | :-----------: | :------------: | :-----------: |
  | Ours |$6.16\pm 0.11$|$3.27\pm 0.08$|$3.27\pm 0.08$|
  | K-means  |$5.22\pm 0.43$|$2.86\pm 0.31$|$2.71\pm 0.25$|
  | DBSCAN  |$5.79\pm 0.26$|$2.53\pm 0.28$|$3.04\pm 0.11$|
  | DPC  |$4.82\pm 0.51$|$2.61\pm 0.41$|$3.10\pm 0.14$|

  - 我们在两个mujoco场景上观测了当无法获得密集奖励时延迟MDP方法是否可以完成Intermitted MDP 任务。
  
  Table 1. Performance (Average of 5 runs):
  | Method      | Ninja | Chaser | Heist |
  | :-----------: | :-----------: | :------------: | :-----------: |
  | Ours |$6.16\pm 0.11$|$3.27\pm 0.08$|$3.27\pm 0.08$|
  | K-means  |$5.22\pm 0.43$|$2.86\pm 0.31$|$2.71\pm 0.25$|
  | DBSCAN  |$5.79\pm 0.26$|$2.53\pm 0.28$|$3.04\pm 0.11$|
  | DPC  |$4.82\pm 0.51$|$2.61\pm 0.41$|$3.10\pm 0.14$|

- *Motion mode requirements* 延迟MDP关注执行者运动的准确性，对运动的连贯性和时间效率没有严格要求。Intermitted MDP需要在确保任务完成的前提下还要求运动的平滑性和时间性能,即要求执行端一直保持运动（参考论文图1），这也是为什么我们在真机实验中设置了两个分别用来评估时间性能和运动平滑性的指标。
  - 我们使用gym中的简单的迷宫导航任务(2dmaze)作为实例，对比了Intermitted MDP 方法和 delayed MDP 方法在运动平滑性和时间性能上的差异。

  Table 1. Performance (Average of 5 runs):
  | Method      | Ninja | Chaser | Heist |
  | :-----------: | :-----------: | :------------: | :-----------: |
  | Ours |$6.16\pm 0.11$|$3.27\pm 0.08$|$3.27\pm 0.08$|
  | K-means  |$5.22\pm 0.43$|$2.86\pm 0.31$|$2.71\pm 0.25$|
  | DBSCAN  |$5.79\pm 0.26$|$2.53\pm 0.28$|$3.04\pm 0.11$|
  | DPC  |$4.82\pm 0.51$|$2.61\pm 0.41$|$3.10\pm 0.14$|

- *The information used for decisions* 延迟MDP方法大多允许访问每次延迟的时间甚至是中间状态来加强决策。而我们对这一方面更加严格，仅能根据当前已有状态实现超前决策。
  - 我们在两个mujoco场景比较了去掉延迟MDP方法中使用的辅助信息后，在Intermitted MDP场景上的效果。
    
  Table 1. Performance (Average of 5 runs):
  | Method      | Ninja | Chaser | Heist |
  | :-----------: | :-----------: | :------------: | :-----------: |
  | Ours |$6.16\pm 0.11$|$3.27\pm 0.08$|$3.27\pm 0.08$|
  | K-means  |$5.22\pm 0.43$|$2.86\pm 0.31$|$2.71\pm 0.25$|
  | DBSCAN  |$5.79\pm 0.26$|$2.53\pm 0.28$|$3.04\pm 0.11$|
  | DPC  |$4.82\pm 0.51$|$2.61\pm 0.41$|$3.10\pm 0.14$|

- *Decision step number* 很多延迟MDP设定对多步决策没有刚性需求（不需要执行端保持持续性运动）[][]都只需要对未来进行单步决策，最近的DCAC可以实现短时间多步决策. 引用："ddd" 而intermittted MDP需要确保执行端能够在较长交互间隔中保持运动，因此决策步数更长。
  - 我们在两个mujoco场景上测试了两种MDP对应的方法处理多时间步决策时的效果。
  Table 1. Performance (Average of 5 runs):
  | Method      | Ninja | Chaser | Heist |
  | :-----------: | :-----------: | :------------: | :-----------: |
  | Ours |$6.16\pm 0.11$|$3.27\pm 0.08$|$3.27\pm 0.08$|
  | K-means  |$5.22\pm 0.43$|$2.86\pm 0.31$|$2.71\pm 0.25$|
  | DBSCAN  |$5.79\pm 0.26$|$2.53\pm 0.28$|$3.04\pm 0.11$|
  | DPC  |$4.82\pm 0.51$|$2.61\pm 0.41$|$3.10\pm 0.14$|


In summary, 两种 MDP分别关注了实时控制领域中的不同问题和场景，且在多个角度是有较大区别的。

**Q2: How does the method compare to existing delay MDP methods.**

感谢您的提问，我们在新版本中增加了我们的方法与主流 延迟MDP方法的比较。
Baslines: 我们选择了delayed mdp经典方法 DCAC RTAC， 2020年的经典工作EMQL以及24年最新的DOMDP方法[Reinforcement Learning from Delayed Observations via World Models]和一个最新的基于模型的方法MB state delayed RL。

Benchmarks: 为了进一步验证我们方法的有效性，我们在四个mujoco基础上增加了四个DMC机器人控制任务和两个metaword中的机械臂抓取任务.

Setting: 交互间隔设置与我们文章主实验保持一致，即最大交互间隔为8.

Table 1. Performance (Average of 5 runs):
| Method      | Ninja | Chaser | Heist |
| :-----------: | :-----------: | :------------: | :-----------: |
| Ours |$6.16\pm 0.11$|$3.27\pm 0.08$|$3.27\pm 0.08$|
| K-means  |$5.22\pm 0.43$|$2.86\pm 0.31$|$2.71\pm 0.25$|
| DBSCAN  |$5.79\pm 0.26$|$2.53\pm 0.28$|$3.04\pm 0.11$|
| DPC  |$4.82\pm 0.51$|$2.61\pm 0.41$|$3.10\pm 0.14$|

进一步，我们比较了当间隔时间步增加后，各方法的有效性（长时间步提前决策能力）。我们的方法有明显的优势。

Table 1. Performance (Average of 5 runs):
| Method      | Ninja | Chaser | Heist |
| :-----------: | :-----------: | :------------: | :-----------: |
| Ours |$6.16\pm 0.11$|$3.27\pm 0.08$|$3.27\pm 0.08$|
| K-means  |$5.22\pm 0.43$|$2.86\pm 0.31$|$2.71\pm 0.25$|
| DBSCAN  |$5.79\pm 0.26$|$2.53\pm 0.28$|$3.04\pm 0.11$|
| DPC  |$4.82\pm 0.51$|$2.61\pm 0.41$|$3.10\pm 0.14$|

**Weakness: The method is straightforward and easy to think of.**

我们在新版本中增加了对我们工作新颖性的突出段落，这个工作的宏观架构确实较为直接 (本质上是一个CVAE)，因此从应用的角度更加便捷、插件化。但在实验初期我们发现原始的CVAE无法有效构建语义平滑的隐空间从而限制了强化学习算法的优化。为此，我们方法的创新点主要是思考如何通过设计更好的辅助技术来提升原始vae的隐空间构建能力.
因为随着动作时间步的增加（纬度增加）vae的表征效果也受到影响。且原始的vae无法构建动作语音平滑的空间从而降低强化学习探索和利用的效果。
为此我们1）为vae引入了atc来自适应约束动作解码在有效区间从而确保策略的有效性（Sec3.1），2）我们引入状态残差模块，引导隐空间中对环境影响相似的点距离相近（Sec3.2）来进一步提升模型效果。

模块有效性验证实验：我们选择了四个mujoco场景对我们方法中的两个关键模块进行验证，结果表明，加了两个模块后,强化学习的优化更加快捷、稳定。且在长时许场景效果比原始vae效果更好。

Table 1. Performance (Average of 5 runs):
| Method      | Ninja | Chaser | Heist |
| :-----------: | :-----------: | :------------: | :-----------: |
| Ours |$6.16\pm 0.11$|$3.27\pm 0.08$|$3.27\pm 0.08$|
| K-means  |$5.22\pm 0.43$|$2.86\pm 0.31$|$2.71\pm 0.25$|
| DBSCAN  |$5.79\pm 0.26$|$2.53\pm 0.28$|$3.04\pm 0.11$|
| DPC  |$4.82\pm 0.51$|$2.61\pm 0.41$|$3.10\pm 0.14$|
