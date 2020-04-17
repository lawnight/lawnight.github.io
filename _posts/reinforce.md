为了达到某个目标，我们通常需要根据环境采取一些列决策。而强化学习就是要通过不断得与环境交互，学习出根据环境做决策的能力。

强化学习和监督学习和非监督学习都不一样。监督学习需要标记好的数据，非监督学习在一堆数据中发现隐藏的structure。而强化学习是以目标为导向的。

**reinforcement learning:**
![](/assets/RL1.png)

agent的主要组成：

- policy：行为函数，a = f(s) 根据状态决定行动。(can derive a policy from value function)
- value：价值函数，预测未来的奖励，衡量状态的好坏。v(s)
- model：预测环境接下会做什么。预测下一个状态。



![](/assets/rl2.png)

## Markov Decision Process

MDP的框架是高度抽象和灵活的。你可以应用在各种不同的问题上。定义各种不同的状态和action。

将目标抽象为奖励。

Markov Decision Process formally describe an environment for  强化学习。

key elements:returns,value functions,bellman equations.

`Markov property`: 未来状态的概率只和当前状态有关系。

### state

state作为policy和value函数的输入参数，也作为model的输入输出参数。

如何定义agent state，决定了如何预测未来的结果。环境状态是markov的。

full observability：代表了环境的状态和agent的状态是一样的。
$$O_t=S^a_t=S^e_t$$

partial observability：agent状态不等于环境状态，我们必须构建自己的agent state。
## Policy Gradients

## credit assignment problem

因为延迟奖励，我们想知道的是过去一段时间里，到底哪一步才是最能导致奖励的步骤。


强化学习除了估计价值函数，还有进化算法（evolutionary methods：such as genetic algo-rithms, genetic programming, simulated annealing）。但是强化学习书的作者认为进化算法并不能很好的完成强化学习的任务。进化算法只关心结果，而价值估算会对每场游戏结果的所有状态，估算价值。所以价值估算更符合不断和环境交互的过程。

## 参考

>1. http://karpathy.github.io/2016/05/31/rl/