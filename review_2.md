We are deeply thankful to you for recognizing the presentation and originality of our work; this positive feedback is greatly encouraging. Furthermore, your objective advice motivates us to further improve this work.

If you think the following response addresses your concerns, we would appreciate it if you could kindly consider raising the score.
### Weaknesses
**W1：The comparisons in the experiment are with some simple and intuitive baselines. While the authors mention that no specific solution exists for this problem, it would still be helpful to discuss related work in other research fields. It would be beneficial to include more tasks to validate the effectiveness of MARS.**

Thanks for your valuable suggestions. In the new version, we add multiple methods in similar fields as baselines and verify the effectiveness of all the methods in more complex scenarios than the original version.

*Baselines:* We select the latest multi-step decision-making fully supervised method ACT [1] from the robotics learning area, which requires us to build an expert dataset for it via ppo in advance; and two recent SOTA methods in the Delayed MDP domain, i.e. DCAC [2] and State Augmentation (SA) [3] (Relax the Intermitted setting restrictions, allow such methods to additionally use dense rewards and delay priors at each step, and set the delay coefficient to the interaction interval). For a more comprehensive analysis, a recent model-based approach,i.e. delayed Dreamer[4], to overcome state delay is also chosen. 

*Benchmarks:* For simulation environments, we further select four more challenging DeepMind Control (DMC) tasks focused on bionic robot locomotion: Dog Run, Dog Trot, Dog Stand and Humanoid Walk. DMC tasks demand coordinated movement from multi-joint bionic robots. Besides, two robotic arm control tasks in MetaWorld: Sweep Into and Coffee Push are used. For the real-world task, we condense the target arrangement (reducing the operational space to 85% of the original size), that is, the manipulator is required to work more smoothly and stably. The invalid jitter is more likely to knock over the target to be operated, resulting in task failure.

*Metrics:* We evaluate methods in terms of performance and Smoothness (whether the motion is coherent and stable, invalid jitter and stagnation will reduce the score, please refer to Sec.4.2 for detail).

Table 1. Performance score (smoothness (%)) (Average of 4 runs):
| Method      |Dog Run|Dog Trot| Dog Stand| Humanoid Walk|Sweep Into|Coffee Push|
| :-----------: | :-----------: | :------------: | :-----------: | :-----------: | :-----------: | :-----------: |
| **Ours**|$124.61\pm 44.92 (75)$|$574.12\pm 28.76 (88)$|$614.03\pm 17.42 (73)$|$105.47\pm 35.83 (82)$|$0.51\pm 0.13 (68)$|$0.38\pm 0.06(63)$|
| ACT|$108.37\pm 36.51 (72)$|$476.95\pm 32.37 (79)$|$607.94\pm 20.56 (75)$|$110.71\pm 16.46 (76)$|$0.47\pm 0.07 (57)$|$0.21\pm 0.03 (69)$|
| DCAC|$96.87\pm 28.44 (53)$|$426.93\pm 50.48 (72)$|$562.64\pm 22.73 (64)$|$105.32\pm 29.16 (49)$|$0.44\pm 0.27 (41)$|$0.19\pm 0.13 (48)$|
| SA |$92.74\pm 51.06 (37)$|$385.67\pm 52.49 (39)$|$503.94\pm 14.86 (27)$|$75.66\pm 31.42 (45)$|$0.52\pm 0.21 (37)$|$0.34\pm 0.09 (39)$|
| delayed Dreamer |$95.31\pm 26.74 (46)$|$428.39\pm 46.23 (46)$|$526.07\pm 21.84 (25)$|$89.25\pm 27.41 (41)$|$0.56\pm 0.17 (32)$|$0.39\pm 0.04 (37)$|

Experimental results show that our method performs better than other methods in almost all intermitted MDP control tasks while ensuring smooth and coherent motion of the agent. The effect of the supervised learning method ACT outperforms the delayed MDP methods, and the delayed MDP method performs well in the robotic arm scene but cannot maintain motion smoothness and time efficiency.

**W2: Minors in writing**

We sincerely apologize for any inconvenience caused by our rough writing. We refined our grammar, spelling, formulas, and notations based on your suggestions.
### Questions
**Q1: Why does the input of the encoder include $s_{t:t+c}$? According to the description of the intermittent control task (Line 31: agents are unable to acquire the state sent by the executor), isn't $s_{t:t+c}$ inaccessible to the agent?**

This question is quite detailed, and we provide further explanation in the updated version. During the policy training stage, only the decoder of the VAE is used to collaborate with the agent's policy training, and $s_{t:t+c}$ is not required in this phase. The encoder is only used in the pre-training stage. 

During the pre-training phase, the construction of the action space is self-supervised, meaning it is independent of policy learning （VAE only needs sufficient information about the environment). Thus, $s_{t:t+c}$ utilized by the encoder does not represent expert data gathered by the RL policy; instead, it is a randomly sampled augmented dataset (abundant but of low quality, please refer to Section 3.1 for detail).

We set up a set of experiments in a random Intermitted MDP Mujoco scenario to compare the effectiveness of expert data (TD3 acquisition) and randomly generated data (random policy) for VAE training. Experimental results show that for a fixed action latent space, randomly sampled state transitions contain richer states (sufficient state transition in the environment can be sampled) and the constructed space is more effective.

Table 2. Performance (Average of 4 runs):
| Method      | Ant-v2| 
| :-----------: | :-----------: |
| MARS with expert data|$5473\pm 219.33$|
| **MARS with random data** |$5908\pm 194.61$|

**Q2:Which variable is the Gaussian exploration noise added to, the decoded action sequence $u_t$, the latent variable $z_t$, or $\upsilon_t$?**

We add Gaussian perturbations to $z_t$ in this paper. Additionally, we analyzed each of the above methods on a random Intermitted Mujoco task and added it to the new submission.

Table 3. Performance (Average of 4 runs):
| Method      | Ant-v2|
| :-----------: | :-----------: |
| $u_t$ perturbation|$5886\pm 242.68$|
| $z_t$ perturbation |$5908\pm 194.61$|
| $\upsilon_t$ perturbation |$5397\pm 206.84$|

The above results indicate that adding noise to $u_t$ and $z_t$ yields comparable effects. This similarity arises from the incorporation of the state transition residual regularization during the construction of the latent space, ensuring it possesses similar semantic smoothness to the original action space. Furthermore, experiments suggest that adding noise to $\upsilon_t$ has no good impact.
### Limitations
**L1:The authors have stated in the Conclusion and Limitation section that representing long action sequences is a limitation and future direction of this work. However, there are no results suggesting that MARS suffers from long action sequences. It would be helpful to discuss more about this limitation.**
In the appendix of the new version, we tested MARS with varying interval step $c$ in the Walker2d environment. The other environmental parameters and network hyperparameters remained consistent with the main experiment.

Table 4. Performance (Average of 4 runs):
| Method      | $c=4$|$c=8$|$c=12$|$c=16$|$c=20$|$c=24$|
| :-----------: | :-----------: |:-----------: |:-----------: |:-----------: |:-----------: |:-----------: |
| MARS|$5309\pm 143.85$|$5283\pm 171.46$|$5309\pm 143.85$|$5194\pm 201.52$|$4758\pm 106.73$|$4514\pm 377.25$|

The Above results reveal a diminishing performance of MARS as the decision step size increases beyond $c=20$. We attribute this trend to the limitations in representation capacity imposed by the MLP architecture in the VAE. In the future, we plan to investigate alternative effective networks like Transformers to enhance the construction capabilities of MARS within action spaces under very long interval step setting.
