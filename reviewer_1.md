We are deeply grateful for your recognition of our paper's motivation, performance, and potential academic impact. Your positive feedback is highly encouraging. We improved our work with your valuable questions.

If you think the following response addresses your concerns, we would appreciate it if you could kindly consider raising the score.
### Questions
**Q1: It seems relevant to "Learning Fine-Grained Bimanual Manipulation with Low-Cost Hardware" (https://arxiv.org/pdf/2304.13705). How do you compare with MARS and this work?.**
感谢您的论文分享！您提供的这篇工作进一步激发了我们对我们工作的完善。并会在新版本中的附录部分单独开辟“未来展望”章节来进行详细讨论。

两个工作的区别 (我们的工作简称MARS, 您提供的工作简称ACT)：
- *Problem focus* ACT主要关注机器人领域非常新颖且重要的问题：即如何为低成本Can learning enable low-cost and imprecise hardware to perform these fine manipulation tasks，而我们主要想构造一个插件话的模块提升强化学习算法在多步决策能力。
- *Method* ACT和MARS都使用了CVAE的架构，但ACT（多步决策模型）主要使用C-VAE依靠高质量的专家熟悉通过模仿学习来离线学习有效的机械臂策略。而MARS（多步动作空间构造模型）主要通过C-VAE来根据低质量的随机状态转移数据构造无策略语义的动作空间，来辅助强化学习策略的训练。信任强化学习策略不涉及决策。高质量数据无法更好的获得。
- *Technical* ACT创造性地根据action chunking 来 To combat the compounding errors of imitation learning in a way that is compatible with pixel-to-action policies。我们则通过引入asc和状态残差来辅助动作空间构建。
但是,ACT篇工作对我们非常有启发：
- 我们发现ACT是一个可靠且有价值的基线，并将其加入到了新版本的实验中。
  - 我们根据文章开源的代码迁移到我们的设定上并在四个mujoco场景（我们将chunking number设为我们设定下的最大间隔，并将tempora ensemble设置为4文章中推荐的设定，并按照论文中的建议使用L1 loss来训练), 进行对比:
    
  Table 1. Scores (Average of 5 runs, ‘*’ represents a newly added method):
  | Method      | Ninja | Chaser | Heist |Leaper | Miner | Dodgeball|
  | :-----------: | :-----------: | :------------: | :-----------: |:-----------: | :------------: | :-----------: |
  | **Ours** |$6.18\pm 0.11$|$3.27\pm 0.08$|$5.65 \pm 0.53$|$5.21 \pm 1.03$|$11.47 \pm 2.97$|$4.80 \pm 0.41$|
  | ExpGen-IDAAC*|$6.15\pm 0.19$|$3.46\pm 0.26$|$5.48\pm 0.45$|$5.17 \pm 0.94$|$11.86\pm 2.71$| $3.72\pm 0.52$|
  | IDAAC |$6.03 \pm 0.14$|$3.14 \pm 0.12$|$4.72 \pm 0.42$|$5.30 \pm 1.19 $|$10.23\pm 1.27$| $4.83\pm0.67$|
  | UCB-DrAC*|$5.65\pm 0.82$|$1.47\pm 0.83$|$4.38\pm 1.07$|$3.97 \pm 1.77$|$8.26\pm 1.95$|$4.59\pm0.53$|
  | MetaDiffuser* |$5.42\pm 0.09$|$2.47\pm 0.32$|$3.62\pm 0.31$|$4.26 \pm 0.52$|$7.63\pm 1.72$| $3.17\pm0.64$|
  | DARC |$5.51 \pm 0.24$|$2.16 \pm 0.58$|$3.82 \pm 1.04$|$4.91 \pm 0.37$|$10.39 \pm 1.04$|$4.17\pm1.06$|
  | M2TD3 |$5.13 \pm 0.32$|$1.35 \pm 0.06$|$4.31 \pm 0.73$|$2.7 \pm 0.63$|$7.17\pm 2.12$| $3.59\pm1.14$|
  - 我们发现，即使用低质量的数据（一半用随机数据）act比我们比较的原始baseline效果更好，且在短步设定上和我们的方法效果持平。但随着步长的增加和数据质量的降低，效果逐步降低。
- ACT启发我们使用transformer架构（文章使用了类似bert的训练方式）来代替mlp构建c-vae能够进一步提升MARS的表征能力。
  - 我们为此做了一组实验。我们发现transformer-based c-vae 对我们的模型有一定提升，并在间隔时间变长后产生了更加明显的表征能力增幅。我们会在未来工作中进一步对其深入探究。
  Table 1. Scores (Average of 5 runs, ‘*’ represents a newly added method):
  | Method      | Ninja | Chaser | Heist |Leaper | Miner | Dodgeball|
  | :-----------: | :-----------: | :------------: | :-----------: |:-----------: | :------------: | :-----------: |
  | **Ours** |$6.18\pm 0.11$|$3.27\pm 0.08$|$5.65 \pm 0.53$|$5.21 \pm 1.03$|$11.47 \pm 2.97$|$4.80 \pm 0.41$|
  | ExpGen-IDAAC*|$6.15\pm 0.19$|$3.46\pm 0.26$|$5.48\pm 0.45$|$5.17 \pm 0.94$|$11.86\pm 2.71$| $3.72\pm 0.52$|
  | IDAAC |$6.03 \pm 0.14$|$3.14 \pm 0.12$|$4.72 \pm 0.42$|$5.30 \pm 1.19 $|$10.23\pm 1.27$| $4.83\pm0.67$|
  | UCB-DrAC*|$5.65\pm 0.82$|$1.47\pm 0.83$|$4.38\pm 1.07$|$3.97 \pm 1.77$|$8.26\pm 1.95$|$4.59\pm0.53$|
  | MetaDiffuser* |$5.42\pm 0.09$|$2.47\pm 0.32$|$3.62\pm 0.31$|$4.26 \pm 0.52$|$7.63\pm 1.72$| $3.17\pm0.64$|
  | DARC |$5.51 \pm 0.24$|$2.16 \pm 0.58$|$3.82 \pm 1.04$|$4.91 \pm 0.37$|$10.39 \pm 1.04$|$4.17\pm1.06$|
  | M2TD3 |$5.13 \pm 0.32$|$1.35 \pm 0.06$|$4.31 \pm 0.73$|$2.7 \pm 0.63$|$7.17\pm 2.12$| $3.59\pm1.14$|
**Q2: All the experiments in Figure 4 are conducted in the same time interval. How does the performance difference change over time interval?**

感谢您从实验角度提出我们可以进一步完善的事情，我们在新版本增加了网格式遍历的不同时间间隔的实验，并将6个mujoco场景的结果进行了记录。

  Table 1. Scores (Average of 5 runs, ‘*’ represents a newly added method):
  | Method      | Ninja | Chaser | Heist |Leaper | Miner | Dodgeball|
  | :-----------: | :-----------: | :------------: | :-----------: |:-----------: | :------------: | :-----------: |
  | **Ours** |$6.18\pm 0.11$|$3.27\pm 0.08$|$5.65 \pm 0.53$|$5.21 \pm 1.03$|$11.47 \pm 2.97$|$4.80 \pm 0.41$|
  | ExpGen-IDAAC*|$6.15\pm 0.19$|$3.46\pm 0.26$|$5.48\pm 0.45$|$5.17 \pm 0.94$|$11.86\pm 2.71$| $3.72\pm 0.52$|
  | IDAAC |$6.03 \pm 0.14$|$3.14 \pm 0.12$|$4.72 \pm 0.42$|$5.30 \pm 1.19 $|$10.23\pm 1.27$| $4.83\pm0.67$|
  | UCB-DrAC*|$5.65\pm 0.82$|$1.47\pm 0.83$|$4.38\pm 1.07$|$3.97 \pm 1.77$|$8.26\pm 1.95$|$4.59\pm0.53$|
  | MetaDiffuser* |$5.42\pm 0.09$|$2.47\pm 0.32$|$3.62\pm 0.31$|$4.26 \pm 0.52$|$7.63\pm 1.72$| $3.17\pm0.64$|
  | DARC |$5.51 \pm 0.24$|$2.16 \pm 0.58$|$3.82 \pm 1.04$|$4.91 \pm 0.37$|$10.39 \pm 1.04$|$4.17\pm1.06$|
  | M2TD3 |$5.13 \pm 0.32$|$1.35 \pm 0.06$|$4.31 \pm 0.73$|$2.7 \pm 0.63$|$7.17\pm 2.12$| $3.59\pm1.14$|
实验表明，我们的方法可以在大多数现实交互间隔设定下产生较好的效果，但随着时间步的增加，例如不常见的极端长间隔场景，我们的方法性能也出现下降趋势。未来我们会尝试更换更强大的表征模型来提升方法性能，例如更换为ACT样式的transformer架构。

**Q3: Line 181: What's the upper limit of action change?.** 

我们会在新版本对这一块详细阐述。动作上限$B$根据每个任务而异。其语义是动作所能改变的最大尺度，例如mujoco中的动作范围是【-1,1] 则$B=1- -1=2$.

**Q4:Eq 3 defines how the action transition scale is computed, but in Figure 3, why does the policy need to output the action transition scale?.** 

您问的问题非常细致，我会在新版本中进行强调。是的正如您所见，我们首先定义了如何计算ats，并将其作为一个自适应改变的条件项来进一步提升解码器的生成。而这一解码器也在第二阶段用于解码强化学习策略选择的隐空间动作，即看作一个保险，来纠正策略的明显失误决策，条件项可以将隐变量约束在一个小子空间内来纠正策略的明显失误决策，这在大多数实时控制场景是需要的，例如极速避障场景需要动作序列变化尺度大，而机器人运动则相反。但想要完成有效的解码除了输入隐动作，还需要输入期望动作尺度转换作为条件项。为了实现自适应的动作尺度转移的选择(然而，若是针对每个场景或一段状态都用人工来提供ats是低效且高成本的)，为此，我们利用强化学习的自适应决策能力，令其在决策隐空间动作的同时分出一个决策头来选择ats（您可以将这一数值类比为和隐动作z一样的一种隐空间，只不过它不是通过深度表征模型学习出来的，而是通过我们提供的计算公式构建的）.这一设计结构在【hybrid action】领域得到了广泛有效性验证。且在我们的实验中也得到了有效性验证。
我们在新版本中增加了对这一模块的消融实验来证明将ats的选择交给强化学习策略是可行的。

基线方法是在其他模块不变的前提下使用先验人工脚本提供合适的ats（TD3+ MARS）。实验表明，强化学习策略在一定时间步的探索后可以找到合适的ats并达到良好的表现。
Table 1. Performance (Average of 5 runs):
| Method      | Ninja | Chaser | Heist |
| :-----------: | :-----------: | :------------: | :-----------: |
| Ours |$6.16\pm 0.11$|$3.27\pm 0.08$|$3.27\pm 0.08$|
| KNN |$5.22\pm 0.43$|$2.86\pm 0.31$|$2.71\pm 0.25$|

**Q5: Line 245: It's confusing to say "choosing optimal z" since you just sample z from a policy.** 

感谢您细致入微地指出我们不严谨的表述，我们原本的意思是想说选择合适的动作。在新版本中将optimal去掉

**Q6: You denote the reconstruction layer as $g_\phi_1$. What does "1" in the subscript mean? This is confusing since you use subscript to denote timestep as well. Similarly, what's the purpose "2" of $h_\phi_2$?** 

我为我们的表述造成的迷惑向您道歉。我们的本意是使用数字i={1,2}来代表解码器之后连接的第i个并行输出头（ reconstruction layer）。新版本会去掉冗余符号。
### Weakness
**W1: Section 4 is unnecessarily long and consists of a lot of redundant text.** 

感谢您从文章结构角度给出的宝贵意见，在新版本中我们凝练了第四章的文字，使其更加简洁直观。

