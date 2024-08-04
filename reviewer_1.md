We are deeply grateful for your recognition of our paper's motivation, performance, and potential academic impact. Your positive feedback is highly encouraging. We improved our work with your valuable questions.

If you think the following response addresses your concerns, we would appreciate it if you could kindly consider raising the score.
### Questions
**Q1: It seems relevant to "Learning Fine-Grained Bimanual Manipulation with Low-Cost Hardware" (https://arxiv.org/pdf/2304.13705). How do you compare with MARS and this work?.**

Thanks for sharing this related research, the paper you provided further motivates us to improve our work. And in the new submission, we discussed this question in detail.

The difference between the two works (ours is referred to as MARS, and related work is referred to as ACT) :
- *Focus:* ACT addresses cutting-edge challenges in robotics: Can learning enable low-cost and imprecise hardware to perform these fine manipulation tasks? However, our primary objective is to develop a plug-in module that enhances the RL algorithm's proficiency in Intermitted MDP tasks.
- *Training style:* ACT is an end-to-end supervised training method, and MARS is an unsupervised training method. 
- *Form of application:* ACT, *a multi-step decision model*, primarily leverages the generative capability of C-VAE and depends on high-quality expert data to enhance the model's multi-step decision-making proficiency through imitation learning. MARS, *a multi-step action space construction model*, primarily utilizes the latent space construction capability of C-VAE to build the action space using low-quality random transition data. Subsequently, it aids in the training of RL-style training. Therefore, it is more suitable for scenarios where reinforcement learning excels.
- *Technic:* ACT creatively introduces action chunking and temporal ensemble to address the compounding errors associated with imitation learning in a manner that aligns with pixel-to-action policies. MARS, on the other hand, assists in action space construction by introducing action transition scale and state residual guidance.

ACT inspired us in two points:
- We found ACT to be a dependable and valuable baseline, and we included it in the main experiments of the new submission.
  - Based on the code provided in the paper, we migrated it to our setup. Initially, we utilized PPO to gather expert data for fully supervised training. We set the chunking number to the maximum interval for our task and configured the Temporal Ensemble to the recommended value in the paper,i.e. 4. Following the paper's suggestions, we trained using L1 loss.
  - Benchmarks: Consistent with the Mujoco tasks in the original version.
  
  Table 1. Performance score of random Intermitted MDP setting (Average of 4 runs):
  | Method  |Ant-v2|Walker2d-v2| HalfCHeetah-v2| Hopper-v2|
  | :-----------: | :-----------: | :------------: | :-----------: | :-----------: | 
  | **Ours**|$4354.61\pm 158.44$|$5436.42\pm 217.83$|$6175.63\pm 273.95$|$2613.58\pm 177.96$|
  | ACT|$3279.82\pm 127.83$|$4892.18\pm 383.07$|$5811.51\pm 108.33$|$2684 .31\pm 238.27$|
  | frameskip TD3|$492.53\pm 31.95$|$2584.18\pm 106.24$|$3281.45\pm 139.42$|$381.74\pm 53.67$|
  | Multi-step TD3|$408.25\pm 31.76$|$529.26\pm 68.21$|$112.73\pm 17.82$|$1173.93\pm 78.28$|

  - We find that ACT performs better than the original baselines we compare, but underperforms our method on most tasks.
- ACT inspired us to employ a transformer architecture (similar to a BERT-like training style) instead of an MLP to construct the C-VAE. This transition is expected to enhance the representation capabilities of MARS in future work.
  - We conducted a set of experiments on the Ant-v2, and we observed that the transformer-based MARS shows promise in enhancing RL algorithms and shows a more significant increase in representation ability when the interval time step $c$ becomes longer.
    
  Table 2. Performance (Average of 4 runs):
  | Method      | $c=4$ | $c=8$ | $c=12$ |$c=16$ | $c=20$ |
  | :-----------: | :-----------: | :------------: | :-----------: |:-----------: |:-----------: |
  |Transformer based |$5281.771\pm 231.59$|$5417.26\pm 193.18$|$5513.47\pm 337.52$|$5604.31\pm 246.52$|$5337\pm 114.23$|
  | MLP based|$5309\pm 143.85$|$5283\pm 171.46$|$5309\pm 143.85$|$5194\pm 201.52$|$4758\pm 106.73$|
**Q2: All the experiments in Figure 4 are conducted in the same time interval. How does the performance difference change over time interval?**

Thanks for your valuable advice. In the appendix of the new version, we tested MARS with varying interval step $c$ in the Walker2d environment. The other environmental parameters and network hyperparameters remained consistent with the main experiment.

Table 3. Performance (Average of 4 runs):
| Method      | $c=4$|$c=8$|$c=12$|$c=16$|$c=20$|$c=24$|
| :-----------: | :-----------: |:-----------: |:-----------: |:-----------: |:-----------: |:-----------: |
| MARS|$5309\pm 143.85$|$5283\pm 171.46$|$5309\pm 143.85$|$5194\pm 201.52$|$4758\pm 106.73$|$4514\pm 377.25$|

The Above results reveal a diminishing performance of MARS as the decision step size increases beyond $c=20$. We attribute this trend to the limitations in representation capacity imposed by the MLP architecture in the VAE. In the future, we plan to investigate alternative effective networks like Transformers to enhance the construction capabilities of MARS within action spaces under very long interval step setting.

**Q3: Line 181: What's the upper limit of action change?** 

We covered this in detail in the new version. The upper action limit $B$ varies according to each task. The semantics is the maximum scale that an action can change. For example, The range of actions in mujoco is $[-1,1]$, then $B=1- (-1)$

**Q4:Eq 3 defines how the action transition scale is computed, but in Figure 3, why does the policy need to output the action transition scale?.** 

This is an in-depth problem, and we emphasize it in the new version. In the second stage, the decoder is employed to decode the latent action selected by the policy, with the action transition scale (ATS) functioning as a condition term that dynamically adjusts to guide the decoder's generation process. This condition term helps constrain the latent variable within a smaller subspace to rectify any erroneous decisions made by the policy. Hence, ATS can be regarded as another decision space akin to the action latent space $z$. However, the ATS is not constructed through deep learning but according to our formulated approach in Sec. 3.1. To enable adaptive selection of ATS, which would be inefficient and costly to manually provide for each scene or state, we leverage the adaptive decision-making capability of RL. This allows the system to decide on actions within the latent space and allocate a separate decision head to select a number from ATS.

This output structure has been extensively validated in the area of hybrid action space control [1][2]. In the new version, we have included ablation experiments on Ant-v2 for this module to demonstrate the viability of entrusting the selection of ATS to the RL policy.

The baseline method involves selecting appropriate ATS through pre-defined manual scripts without altering other modules. Results indicate that the RL policy can identify the suitable ATS and achieve commendable performance after a certain exploration step.

Table 4. Performance (Average of 3 runs):
| Method      |  training step = 50k|training step = 1m|training step = 2m|
| :-----------: | :-----------: | :-----------: | :-----------: | 
| Ours |$1962\pm 421.42$|$3243\pm 109.25$|$4183\pm 171.46$|
| Baseline |$2213\pm 311.97$|$3126\pm 128.36$|$4207\pm 136.61$|

**Q5: Line 245: It's confusing to say "choosing optimal z" since you just sample z from a policy.** 

Thanks for pointing out our imprecise presentation. In the new version, "optimal" is removed.

**Q6: You denote the reconstruction layer as $g_{\phi_1}$. What does "1" in the subscript mean? This is confusing since you use subscript to denote timestep as well. Similarly, what's the purpose "2" of $h_{\phi_2}$?** 

We apologize for any confusion caused by our presentation. We intend to use the number $i=\{1,2\}$ to represent the $i_{th}$ parallel output head (reconstruction layer) connected after the decoder. Redundant symbols are removed in the new edition.
### Weakness
**W1: Section 4 is unnecessarily long and consists of a lot of redundant text.** 

 In the latest version, we streamlined the content of Section 4 to enhance its conciseness and clarity.

