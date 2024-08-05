We are immensely grateful to you for recognizing the importance and value of our research. Your suggestions inspire us to improve our work further. Based on your feedback, we analyzed the difference between our setting (intermittent MDP) and Delayed-MDP through a demo example and detailed explanation in the revised version. Additionally, we compared our method with several SOTA delayed MDP methods on multiple complex scenarios in the main experiment section. (4 more robotic motion control tasks and 2 more manipulation tasks). 

If you think the following response addresses your concerns, we would appreciate it if you could kindly consider raising the score.
### Questions
**Q1: Compare Intermitted MDP and Delayed MDP: Why define intermittent MDP separately while delayed MDP is widely adopted in the community?**

We apologize for any confusion caused by our vague statements, we describe our setting more clearly in the new version. Both the Intermittent MDP and the traditional Delayed MDP models aim to represent non-ideal (unstable) environments. However, the tasks differences that both focus on also make the following distinction：

*Training Data:* Delayed MDP focuses on scenarios where information is transmitted asynchronously (yet not lost) because of high network latency. In this context, **the agent has access to complete data during training but cannot utilize the most recent data when making decisions**[1]. Intermittent MDP addresses situations where messages are transmitted instantaneously (with negligible delays that can be efficiently managed using standard communication layer techniques [2]). However, the communication channel between the executor and the decision-making side may be sporadically or systematically obstructed [3], resulting in information that fails to arrive being considered a direct loss. **Consequently, the agent can only utilize incomplete data during both training and decision-making processes.** 
- We compare the delayed MDP method trained on incomplete transition with our method on Ant-v2 and find that our method performs somewhat better. We use two recent SOTA delayed MDP methods, i.e. DCAC[3] and State Augmentation(SA)[6], as representatives of this class of methods. The setup of the envrionment is the same as that in the main experiments.
  
Table 1. Performance (Average of 3 runs):
| Method      | Intermittent Control Ant-v2|
| :-----------: | :-----------: | 
| Ours |$4354.61\pm 158.44$|
| DCAC  |$3495.68\pm 244.51$|
| SA |$3108.29\pm 194.35$|

*Reward feedback: (described in)* Mostly, in delayed MDP, the reward associated with each state-action pair can be acquired independently (**dense reward**)[5]. However, Intermittent MDP regards reward feedback as the cumulative rewards of a sequence of actions (**sparse reward**), a depiction that aligns more realistically with scenarios in practice. 
- We compare the performance of the delayed MDP methods with our method on Ant-v2 when dense rewards are not available (simulate lost state by repeating the last reward in the sequence). The results show that our method is more effective in dealing with fuzzy reward feedback.
  
  Table 2. Performance (Average of 3 runs):
| Method      | Intermittent Control Ant-v2|
| :-----------: | :-----------: | 
| Ours |$4354.61\pm 158.44$|
| DCAC  |$3281.93\pm 544.17$|
| SA |$3446.21\pm 262.59$|

- *Motion mode requirements (described in Fig.1)* The real-world tasks that Intermittent MDP targets typically always involve high-frequency operational demands, e.g. game NPC control and robot control[4]. Consequently, **Intermittent MDP expects the action to be executed at every time step to guarantee the smooth and stable motion. In contrast, delayed MDP does not incorporate this constraint and permits certain time steps to be non-excution points.** For instance, in a scenario where a robot needs to continue walking despite a blocked interaction channel, it cannot afford to wait for the next state to be successfully transmitted before taking action. Delaying the decision in such cases could result in the movement abruptly halting within a specific timeframe, leading to a loss of balance and a potential fall. So the focus is not solely on the efficiency of discretely executing actions. Rather, the emphasis lies on ensuring that each action smoothly transitions with its neighboring actions while maintaining validity. 
- We compared the motion smoothness and temporal performance between our method and delayed MDP methods in the Humanoid scenario. Humanoid demands exceptionally high motion balance and necessitates smooth fine-tuning at each time step to maintain balance effectively.  We use the Smoothness rate to measure the motion (whether the motion is coherent and stable, invalid jitter and stagnation will reduce the score, please refer to Sec.4.2 for detail). The results show that our method can make the movement of the executive side smoother and more stable.

  Table 3. Performance (smoothness rate (%)) (Average of 3 runs):
| Method      | Intermittent Control Ant-v2|
| :-----------: | :-----------: | 
| Ours |$4354.61\pm 158.44$ (78)|
| DCAC  |$3495.68\pm 244.51$ (42)|
| SA |$3108.29\pm 194.35$ (51)|

*The information used for decisions* Delayed MDP methods typically involve accessing prior information, such as the time to be delayed or even intermediate states, to enhance decision-making. Our constraint is more stringent, where the agent is only permitted to make advanced decisions based on the current state.
- We compare the effect of the delayed MDP method without auxiliary information (simulate lost state by using zero mask) and our method, on the Intermitted control task. The results show that our method is somewhat more robust to sparse information.
    
Table 4. Performance (Average of 3 runs):
| Method      | Intermittent Control Ant-v2|
| :-----------: | :-----------: | 
| Ours |$4354.61\pm 158.44$|
| DCAC  |$2795.68\pm 485.83$|
| SA |$3371.46\pm 169.27$|

*Decision step number per step* The delay scenario addressed by delayed MDP involves brief durations of delay (statistics from [3] indicate that the majority of delays are less than one second), making the future decision action length relatively short in this context. In contrast, our setup necessitates accounting for instances where the communication channel may remain non-functional for an extended time due to channel breakdowns. Therefore, Intermittent MDP method must consider longer durations (in real scenarios, there could be intervals exceeding 5 seconds [4]) in our deliberations, i.e. Deciding on a lengthy action sequence in a single step.
- We tested the ability upper bound of single-step decision action sequence length for the corresponding methods of the two MDPS in the Walker2d scenario. The results show that our method performs better in the long decision sequence setting. DCAC is the easiest method to change to multi-step decision mode in delayed MDP setting.
Table 5. Performance (Average of 3 runs):
| Method      | Decision step $c=4$|$c=8$|$c=12$|$c=16$|
| :-----------: | :-----------: |:-----------: |:-----------: |:-----------: |
| MARS|$5309\pm 143.85$|$5283\pm 171.46$|$5309\pm 143.85$|$5194\pm 201.52$|
| DCAC|$5134\pm 118.64$|$4796\pm 208.72$|$2962\pm 571.07$|$2836\pm 485.11$|

**Q2: How does the method compare to existing delay MDP methods.**

Thanks for your suggestion. we add a comparison of our method with mainstream deferred MDP methods on multiple complex tasks from the perspective of multiple metrics in the new version.

*Baselines:* We select the latest multi-step decision-making fully supervised method ACT [1] from the robotics learning area, which requires us to build an expert dataset for it via ppo in advance; and two recent SOTA methods in the Delayed MDP domain, i.e. DCAC [2] and State Augmentation (SA) [3] (Relax the Intermitted setting restrictions, allow such methods to additionally use dense rewards and delay priors at each step, and set the delay coefficient to the interaction interval). For a more comprehensive analysis, a recent model-based approach,i.e. delayed Dreamer[4], to overcome state delay is also chosen. 

*Benchmarks:* For simulation environments, we further select four more challenging DeepMind Control (DMC) tasks focused on bionic robot locomotion: Dog Run, Dog Trot, Dog Stand and Humanoid Walk. DMC tasks demand coordinated movement from multi-joint bionic robots. Besides, two robotic arm control tasks in MetaWorld: Sweep Into and Coffee Push are used. For the real-world task, we condense the target arrangement (reducing the operational space to 85% of the original size), that is, the manipulator is required to work more smoothly and stably. The invalid jitter is more likely to knock over the target to be operated, resulting in task failure. The environmental parameters and network hyperparameters remained consistent with the main experiment.

*Metrics:* We evaluate methods in terms of performance and Smoothness (whether the motion is coherent and stable, invalid jitter and stagnation will reduce the score, please refer to Sec.4.2 for detail).

Table 6. Performance in newly added difficult tasks (smoothness (%)) (Average of 4 runs):
| Method      |Dog Run|Dog Trot| Dog Stand| Humanoid Walk|Sweep Into|Coffee Push|
| :-----------: | :-----------: | :------------: | :-----------: | :-----------: | :-----------: | :-----------: |
| **Ours**|$124.61\pm 44.92 (75)$|$574.12\pm 28.76 (88)$|$614.03\pm 17.42 (73)$|$105.47\pm 35.83 (82)$|$0.51\pm 0.13 (68)$|$0.38\pm 0.06(63)$|
| DCAC|$96.87\pm 28.44 (53)$|$426.93\pm 50.48 (72)$|$562.64\pm 22.73 (64)$|$105.32\pm 29.16 (49)$|$0.44\pm 0.27 (41)$|$0.19\pm 0.13 (48)$|
| SA |$92.74\pm 51.06 (37)$|$385.67\pm 52.49 (39)$|$503.94\pm 14.86 (27)$|$75.66\pm 31.42 (45)$|$0.52\pm 0.21 (37)$|$0.34\pm 0.09 (39)$|
| delayed Dreamer |$95.31\pm 26.74 (46)$|$428.39\pm 46.23 (46)$|$526.07\pm 21.84 (25)$|$89.25\pm 27.41 (41)$|$0.56\pm 0.17 (32)$|$0.39\pm 0.04 (37)$|

Table 7. Performance in old tasks (smoothness (%)) (Average of 4 runs):
| Method  |Ant-v2|Walker2d-v2| HalfCHeetah-v2| Hopper-v2|
| :-----------: | :-----------: | :------------: | :-----------: | :-----------: | 
| **Ours**|$4354.61\pm 158.44$ (78)|$5436.42\pm 217.83$ (82)|$6175.63\pm 273.95$ (76)|$2613.58\pm 177.96$ (83)|
| DCAC |$3279.82\pm 127.83$ (42)|$4892.18\pm 383.07$ (54)|$5811.51\pm 108.33$ (58)|$2684 .31\pm 238.27$ (63)|
| SA |$492.53\pm 31.95$ (51)|$2584.18\pm 106.24$ (59)|$3281.45\pm 139.42$ (66)|$381.74\pm 53.67$ (71)|
| delayed Dreamer (38)|$408.25\pm 31.76$ (35)|$529.26\pm 68.21$ (63)|$112.73\pm 17.82$ (68)|$1173.93\pm 78.28$ (78)|

Table 6,7 shows that our method performs better than Delayed MDP methods in almost all intermitted MDP control tasks while ensuring smooth and coherent motion of the agent. 

P.S. If you have other delayed MDP methods that you would like us to compare but are not in the scope of our investigation, please let us know and we would be happy to include them in our experimental analysis.
### Weaknesses
**W1: The method is straightforward and easy to think of.**

We add a description of the novelty of our work. The Backbone of MARS is indeed clear, essentially resembling a CVAE, thereby enhancing convenience and plug-and-play capabilities from an application standpoint. However, at the methodological level, we discovered that the original CVAE could not effectively construct the semantic smooth latent space and the representation realization of long action sequences was poor, thereby constraining the optimization of the RL algorithm. Therefore, the primary innovation of our method revolves around enhancing the capability of constructing the latent space of the original VAE through the development of more effective auxiliary techniques.

In particular, we introduce two innovative techniques for the original VAE: 1) the introduction of ATC to dynamically restrict action decoding within an effective interval to ensure policy effectiveness (Sec 3.1), and 2) the incorporation of a state residual module to encourage points in the latent space with similar environmental impacts to be closer to each other (Sec 3.2), enhancing the overall model performance.

Module validation experiments are conducted on Ant-v2 to assess the efficacy of the two key modules. The results in Tabel 8 indicate that RL optimization is quicker and more efficient with the inclusion of these modules. Additionally, results in Table 9 show that the performance of MARS surpasses that of the original VAE in long sequence decision scenarios.


Table 8. Performance (smoothness (%)) (Average of 3 runs):
| Method  |Ant-v2|
| :-----------: | :-----------: |
| VAE + ATS + SR (MARS) |$4354.61\pm 158.44$|
| VAE + SR |$4016.47\pm 213.06$|
| VAE + ATS |$4122.18\pm 65.37$|
| vanilla VAE|$3685.92\pm 362.72$|

Table 9. Performance in Walker2d  (Average of 4 runs):
| Method      | Decision step $c=4$|$c=8$|$c=12$|$c=16$|
| :-----------: | :-----------: |:-----------: |:-----------: |:-----------: |
| VAE + ATS + SR (MARS) |$5309\pm 143.85$|$5283\pm 171.46$|$5309\pm 143.85$|$5194\pm 201.52$|
| vanilla VAE |$4623.51\pm 463.81$|$3941.21\pm 297.44$|$3806.14\pm 538.24$|$3612 .23\pm 635.42$|
