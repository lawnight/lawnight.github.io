为了达到某个目标，我们通常需要根据环境采取一些列决策。而强化学习就是要通过不断得与环境交互，学习出根据环境做决策的能力。

强化学习和监督学习和非监督学习都不一样。监督学习需要标记好的数据，非监督学习在一堆数据中发现隐藏的structure。而强化学习是以目标为导向的。

**reinforcement learning:**
![](/assets/RL1.png)

agent的主要组件：

- policy：行为函数，a = f(s) 根据状态决定行动。(can derive a policy from value function)
- value：价值函数，预测未来的奖励，衡量状态的好坏。v(s)
- model：预测环境接下会做什么。预测下一个状态。

![](/assets/rl2.png)

## Markov Decision Process

### 马尔科夫链

`Markov property`: 未来状态的概率只和当前状态有关系。

马尔科夫，简化了模型，让很多复杂的情况能够分析。

markov chain -> markov reward

MDP的框架是高度抽象和灵活的。你可以应用在各种不同的问题上。定义各种不同的状态和action。

将目标抽象为奖励。

Markov Decision Process formally describe an environment for  强化学习。

key elements:returns,value functions,bellman equations.



### return and reward

return来更好的定义奖励，不光现在的奖励，考虑未来可能获得的奖励。

$$G_t=R_{t+1}+rR_{t+2}+r^2R_{t+3}+...$$

r 取值范围为{0,1}。当r小于1的时候，对于无限的链，$G_t$可以求得极限值。r是强学学习可调的一个超参数。

### state

state作为policy和value函数的输入参数，也作为model的输入输出参数。

如何定义agent state，决定了如何预测未来的结果。环境状态是markov的。

full observability：代表了环境的状态和agent的状态是一样的。
$$O_t=S^a_t=S^e_t$$

partial observability：agent状态不等于环境状态，我们必须构建自己的agent state。

## value函数计算

### bellman equation

贝尔曼方程也称作动态规划方程。贝尔曼方程的介绍过于复杂，暂时当做一个递归方程。

推导出bellman equation：
![](/assets/rl3.png)
然后转换成矩阵的形式，最后对矩阵求逆来计算value。但是对大矩阵的求逆运算量很大，所以这个只适应小的MDP。

蒙特卡洛采样。

## Policy Gradients

## credit assignment problem

因为延迟奖励，我们想知道的是过去一段时间里，到底哪一步才是最能导致奖励的步骤。


强化学习除了估计价值函数，还有进化算法（evolutionary methods：such as genetic algo-rithms, genetic programming, simulated annealing）。但是强化学习书的作者认为进化算法并不能很好的完成强化学习的任务。进化算法只关心结果，而价值估算会对每场游戏结果的所有状态，估算价值。所以价值估算更符合不断和环境交互的过程。

## 参考

>1. http://karpathy.github.io/2016/05/31/rl/