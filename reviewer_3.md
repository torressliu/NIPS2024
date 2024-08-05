We are immensely grateful to you for recognizing the importance and value of our research. Your suggestions inspire us to improve our work further. Based on your feedback, we analyzed the difference between our setting (intermittent MDP) and Delayed-MDP through a demo example and detailed explanation in the revised version. Additionally, we compared our method with several SOTA delayed MDP methods on multiple complex scenarios in the main experiment section. (4 more robotic motion control tasks and 2 more manipulation tasks). 

If you think the following response addresses your concerns, we would appreciate it if you could kindly consider raising the score.
### Questions
**Q1: Compare Intermitted MDP and Delayed MDP: Why define intermittent MDP separately while delayed MDP is widely adopted in the community?**

Thanks for your in-depth question, we describe our setting more clearly in the new version. Both the Intermittent MDP and the traditional Delayed MDP models aim to represent non-ideal (unstable) environments. However, the tasks differences that both focus on also make the following distinction：
- *Training Data:* Delayed MDP focuses on scenarios where information is transmitted asynchronously (yet not lost) because of high network latency. In this context, **the agent has access to complete data during training but cannot utilize the most recent data when making decisions**[1]. Intermittent MDP addresses situations where messages are transmitted instantaneously (with negligible delays that can be efficiently managed using standard communication layer techniques [2]). However, the communication channel between the executor and the decision-making side may be sporadically or systematically obstructed [3], resulting in information that fails to arrive being considered a direct loss. **Consequently, the agent can only utilize incomplete data during both training and decision-making processes.** 
  - 我们在两个mujoco场景上观测了当无法在训练阶段使用密集状态转移时延迟MDP方法是否可以完成Intermitted MDP 任务。
  
  Table 1. Performance (Average of 5 runs):
  | Method      | Ninja | Chaser | Heist |
  | :-----------: | :-----------: | :------------: | :-----------: |
  | Ours |$6.16\pm 0.11$|$3.27\pm 0.08$|$3.27\pm 0.08$|
  | K-means  |$5.22\pm 0.43$|$2.86\pm 0.31$|$2.71\pm 0.25$|
  | DBSCAN  |$5.79\pm 0.26$|$2.53\pm 0.28$|$3.04\pm 0.11$|
  | DPC  |$4.82\pm 0.51$|$2.61\pm 0.41$|$3.10\pm 0.14$|

- *Reward feedback:* Mostly, in delayed MDP, the reward associated with each state-action pair can be acquired independently (**dense reward**)[5]. However, Intermittent MDP regards reward feedback as the cumulative rewards of a sequence of actions (**sparse reward**), a depiction that aligns more realistically with scenarios in practice. 
  - We compare the performance of the delayed MDP methods with our method on Ant-v2 when dense rewards are not available. The results show that our method is more effective in dealing with fuzzy reward feedback.
  
  Table 1. Performance (Average of 5 runs):
  | Method      | Ninja | Chaser | Heist |
  | :-----------: | :-----------: | :------------: | :-----------: |
  | Ours |$6.16\pm 0.11$|$3.27\pm 0.08$|$3.27\pm 0.08$|
  | K-means  |$5.22\pm 0.43$|$2.86\pm 0.31$|$2.71\pm 0.25$|
  | DBSCAN  |$5.79\pm 0.26$|$2.53\pm 0.28$|$3.04\pm 0.11$|
  | DPC  |$4.82\pm 0.51$|$2.61\pm 0.41$|$3.10\pm 0.14$|

- *Motion mode requirements (described in Fig.1)* The real-world tasks that Intermittent MDP targets typically always involve high-frequency operational demands, e.g. game NPC control and robot control[4]. Consequently, **Intermittent MDP expects the action to be executed at every time step to guarantee the smoothness and stability motion. In contrast, delayed MDP does not incorporate this constraint and permits certain time steps to be non-excution points.** For instance, in a scenario where a robot needs to continue walking despite a blocked interaction channel, it cannot afford to wait for the next state to be successfully transmitted before taking action. Delaying the decision in such cases could result in the movement abruptly halting within a specific timeframe, leading to a loss of balance and a potential fall. So the focus is not solely on the efficiency of discretely executing actions. Rather, the emphasis lies on ensuring that each action smoothly transitions with its neighboring actions while maintaining validity. 
  - We compared the motion smoothness and temporal performance between our method and delayed MDP methods in the Humanoid scenario. Humanoid demands exceptionally high motion balance and necessitates smooth fine-tuning at each time step to maintain balance effectively.  The results show that our method can make the movement of the executive side smoother and more stable.

  Table 1. Performance (Average of 5 runs):
  | Method      | Ninja | Chaser | Heist |
  | :-----------: | :-----------: | :------------: | :-----------: |
  | Ours |$6.16\pm 0.11$|$3.27\pm 0.08$|$3.27\pm 0.08$|
  | K-means  |$5.22\pm 0.43$|$2.86\pm 0.31$|$2.71\pm 0.25$|
  | DBSCAN  |$5.79\pm 0.26$|$2.53\pm 0.28$|$3.04\pm 0.11$|
  | DPC  |$4.82\pm 0.51$|$2.61\pm 0.41$|$3.10\pm 0.14$|

- *The information used for decisions* Delayed MDP methods typically involve accessing prior information, such as the time to be delayed or even intermediate states, to enhance decision-making. Our constraint is more stringent, where the agent is only permitted to make advanced decisions based on the current state.
  - We compare the effect of the delayed MDP method without auxiliary information and our method, on the Intermitted control task. The results show that our method is somewhat more robust to sparse information.
    
  Table 1. Performance (Average of 5 runs):
  | Method      | Ninja | Chaser | Heist |
  | :-----------: | :-----------: | :------------: | :-----------: |
  | Ours |$6.16\pm 0.11$|$3.27\pm 0.08$|$3.27\pm 0.08$|
  | K-means  |$5.22\pm 0.43$|$2.86\pm 0.31$|$2.71\pm 0.25$|
  | DBSCAN  |$5.79\pm 0.26$|$2.53\pm 0.28$|$3.04\pm 0.11$|
  | DPC  |$4.82\pm 0.51$|$2.61\pm 0.41$|$3.10\pm 0.14$|

- *Decision step number per step* The delay scenario addressed by delayed MDP involves brief durations of delay (statistics from [3] indicate that the majority of delays are less than one second), making the future decision action length relatively short in this context. In contrast, our setup necessitates accounting for instances where the communication channel may remain non-functional for an extended time due to channel breakdowns. Therefore, Intermittent MDP method must consider longer durations (in real scenarios, there could be intervals exceeding 5 seconds [4]) in our deliberations, i.e. Deciding on a lengthy action sequence in a single step.
  - We tested the ability upper bound of single-step decision action sequence length for the corresponding methods of the two MDPS on the Ant-v2 scenario. The results show that our method performs better in the long decision sequence setting.
  Table 1. Performance (Average of 5 runs):
  | Method      | Ninja | Chaser | Heist |
  | :-----------: | :-----------: | :------------: | :-----------: |
  | Ours |$6.16\pm 0.11$|$3.27\pm 0.08$|$3.27\pm 0.08$|
  | K-means  |$5.22\pm 0.43$|$2.86\pm 0.31$|$2.71\pm 0.25$|
  | DBSCAN  |$5.79\pm 0.26$|$2.53\pm 0.28$|$3.04\pm 0.11$|
  | DPC  |$4.82\pm 0.51$|$2.61\pm 0.41$|$3.10\pm 0.14$|

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
