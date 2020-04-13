为了达到某个目标，我们通常需要根据环境采取一些列决策。而强化学习就是要通过不断得与环境交互，学习出根据环境做决策的能力。

强化学习和监督学习和非监督学习都不一样。非监督学习一般在一堆数据中发现隐藏的structure。

**reinforcement learning:**
![](/assets/RL1.png)

agent的主要组成：

- policy：行为函数，a = f(s) 根据状态决定行动。(can derive a policy from value function)
- value：价值函数，预测未来的奖励，衡量状态的好坏。
- model：预测环境接下会做什么。预测下一个状态。



![](/assets/rl2.png)

## Markov Decision Process

### state

state作为policy和value函数的输入参数，也作为model的输入输出参数。

## Policy Gradients

## credit assignment problem

因为延迟奖励，我们想知道的是过去一段时间里，到底哪一步才是最能导致奖励的步骤。



## 参考

>1. http://karpathy.github.io/2016/05/31/rl/