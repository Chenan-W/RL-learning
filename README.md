[TOC]

# 0. 记录

## 0.1 路线图

<img src="https://pic3.zhimg.com/80/v2-2056d6eb693ebcddd0fe90bbbe6309f2_720w.jpg" alt="img" style="zoom: 80%;" />

参考资料：[知乎-张斯俊：强化学习线路图](https://zhuanlan.zhihu.com/p/111869532)

## 0.2 算法选择
- 对于连续控制任务，推荐SAC、TD3和PPO，三种算法都值得试一试并从中择优
- 对于离散控制任务，推荐SAC-Discrete（即离散版SAC）和PPO




# 1. 马尔科夫链

## 1.1 强化学习一般步骤

- 智能体在环境中，观察到状态(S)
- 状态(S)被输入到智能体，智能体经过计算，选择动作(A)
- 动作(A)使智能体进入另外一个状态(S)，并返回奖励(R)给智能体
- 智能体根据返回，调整自己的策略。 重复以上步骤，一步一步创造马尔科夫链

## 1.2 马尔科夫链

1. 马尔科夫链是用来描述智能体和环境互动的过程。
2. 马尔科夫链包含三要素：state，action，reward

> - state：只有智能体能够观察到的才是state
> - action：智能体的动作
> - reward：引导智能体工作学习的主观的值
> - 马尔科夫链的不确定性
> - 核心思想：如果你不希望孩子有某种行为，那么当这种行为出现的时候就进行惩罚。如果你希望孩子坚持某种行为，那么就进行奖励。这也是整个强化学习的核心思想。



# 2. Q值和V值

## 2.1 定义

- 评估**动作**的价值，我们称为**Q值**：它代表了智能体选择这个动作后，一直到最终状态**奖励总和**的**期望**；用大白话总结就是：从某个状态，按照策略 ，走到最终状态很多很多次；最终获得奖励总和的平均值，就是V值

- 评估**状态**的价值，我们称为**V值**：它代表了智能体在这个状态下，一直到最终状态的**奖励总和**的**期望**。用大白话总结就是：从某个状态选取动作A，走到最终状态很多很多次；最终获得奖励总和的平均值，就是Q值

## 2.2 总结

1. 从V值的计算，我们可以知道，V值代表了这个状态的今后能获得奖励的期望。从这个状态出发，到达最终状态，平均而言能拿到多少奖励。所以我们轻易比较两个状态的价值
2. V值跟我们选择的策略有很大的关系，**V值是会根据不同的策略有所变化**
3. 与V值不同，**Q值和策略并没有直接相关**，而与环境的状态转移概率相关，而环境的状态转移概率是不变的
4. 从以上的定义，我们可以知道Q值和V值的意义相通的：
   - 都是马尔科夫树上的节点；
   - 价值评价的方式是一样的：从当前节点出发—— 一直走到最终节点——所有的奖励的期望值

## 2.3 Q 和 V 转换

- Q 到 V

<img src="/home/wca/Pictures/2021-07-19 17-11-15 的屏幕截图.png" style="zoom: 50%;" />

- V 到 Q

  <img src="/home/wca/Pictures/2021-07-19 17-11-24 的屏幕截图.png" style="zoom:50%;" />

- V 到 V

  <img src="/home/wca/Pictures/2021-07-19 17-11-32 的屏幕截图.png" style="zoom:50%;" />



# 3. 蒙特卡洛 (Monte-Carlo)

## 3.1 蒙特卡洛算法 (MC)

1. 我们把智能体放到环境的任意状态
2. 从这个状态开始按照策略进行选择动作，并进入新的状态
3. 重复步骤2，直到最终状态
4. 我们从最终状态开始向前回溯：计算每个状态的G值
5. 重复1-4多次，然后平均每个状态的G值，这就是我们需要求的V值

图解

<img src="/home/wca/Pictures/2021-07-19 18-07-53 的屏幕截图.png" style="zoom: 50%;" />

- 第一步，我们根据策略往前走，一直走到最后，期间我们什么都不用算，还需要记录每一个状态转移，我们获得多少奖励R即可。
- 第二步，我们从终点往前走，一遍走一遍计算G值。G值等于上一个状态的G值(记作G'),乘以一定的折扣(gamma),再加上R。

<img src="/home/wca/Pictures/2021-07-19 18-08-52 的屏幕截图.png" style="zoom: 67%;" />

所以G值的意义在于，在这一次游戏中，某个状态到最终状态的奖励总和(理解时可以忽略折扣值)

## 3.2 理解

1. **G的意义：在某个路径上，状态S到最终状态的总收获**
2. V和G的关系：V是G的平均数

## 3.3 增量更新

V是G的平均值，但我们可以用**增量更新**的方式： 现在我们只需要记录之前的平均值V，新加进来的G，和次数N。我们把V和G的差，除以N，然后再加到原来的平均值V上，就能计算到新的V值。 

<center> V_new = (V_old - G) * (1 / N) + V_old </center>

- V_old：原来的V值，也就是平均值
-  G：这一次回溯后，计算出来的G值，也就是新加进来的香蕉 
- N：这个状态被经过多少次了
-  V_new：新计算出来的V值，也就是平均值

理解MC的更新公式：

<img src="/home/wca/Pictures/2021-07-19 17-33-56 的屏幕截图.png" style="zoom:50%;" />

## 3.4 时序差分算法 (TD)

TD算法对蒙特卡洛(MC)进行了改进。

1. 和蒙特卡洛(MC)不同：TD算法只需要走N步。就可以开始回溯更新
2. 和蒙特卡洛(MC)一样：需要先走N步，每经过一个状态，把奖励记录下来。然后开始回溯。
3. 那么，状态的V值怎么算呢？其实和蒙地卡罗一样，我们就假设N步之后，就到达了最终状态了。 
   - 假设“最终状态”上我们之前没有走过，所以这个状态上的纸是空白的，这个时候我们就当这个状态为0
   - 假设“最终状态”上我们已经走过了，这个状态的V值，就是当前值，然后我们开始回溯

在MC中，G是更新目标，而在TD，我们只不过把更新目标从G，改成了 R + gamma*V

<img src="/home/wca/Pictures/2021-07-19 17-38-29 的屏幕截图.png" style="zoom: 67%;" />

图解

<img src="/home/wca/Pictures/2021-07-19 18-10-38 的屏幕截图.png" style="zoom: 40%;" />



# 4. Qlearning

## 4.1 TD之于Q值估算

用上TD的思路。 我们在St，智能体根据策略pi，选择动作At，进入S(t+1)状态，并获得奖励R，那么不难理解下面这张图。

<img src="/home/wca/Pictures/2021-07-19 18-11-12 的屏幕截图.png" style="zoom:40%;" />

但是在St+1下，可能有很多动作At+1。不同动作的Q值自然是不同的。 所以Q(St+1,At+1)并不能等价于V(St+1)。

但是人们认为有个可能的动作产生的Q值能够一定程度代表V(St+1)。

**1. 在相同策略下产生的动作At+1。这就是SARSA**
**2. 选择能够产生最大Q值的动作At+1。这就是Qlearning**

## 4.2 SARSA

SARSA和TD估算V值几乎一模一样，只不过从 V 改为了 Q

<img src="/home/wca/Pictures/2021-07-19 17-44-08 的屏幕截图.png" style="zoom: 67%;" />

- 这里的At+1是在同一策略产生的。也就是说,St选At的策略和St+1选At+1是同一个策略。这也是SARSA和Qlearning的唯一区别。

图解

<img src="/home/wca/Pictures/2021-07-19 18-12-59 的屏幕截图.png" style="zoom: 55%;" />

## 4.3 Qlearning

<img src="/home/wca/Pictures/2021-07-19 18-14-17 的屏幕截图.png" style="zoom: 60%;" />

Qlearning能够产生最大Q值的动作At+1的Q值作为V(St+1)的替代。

<img src="/home/wca/Pictures/2021-07-19 17-52-09 的屏幕截图.png" style="zoom: 67%;" />

图解

<img src="/home/wca/Pictures/2021-07-19 18-06-09 的屏幕截图.png" style="zoom: 55%;" />

## 4.4 总结

1. `Qlearning` 和 `SARSA` 都是基于 TD(0) 的。不过在之前的介绍中，我们用 TD(0) 估算状态的V值。而 `Qlearning` 和 `SARSA` 估算的是动作的Q值
2. `Qlearning` 和 `SARSA` 的核心原理，是用下一个状态St+1的V值，估算Q值
3. 既要估算Q值，又要估算V值会显得比较麻烦。所以我们用下一状态下的某一个动作的Q值，来代表St+1的V值
4. `Qlearning` 和 `SARSA` 唯一的不同，就是用什么动作的Q值替代St+1的V值
   - `SARSA` 选择的是在St同一个策略产生的动作
   - `Qlearning` 选择的是能够产生最大的Q值的动作。

- 可以简单的理解为 `Qlearning` 比较“大胆”， `SARSA` 就显得比较“谨慎”：

  例如在Cliff-walking环境里，`Qlearning` 会紧贴着悬崖壁走到终点，因为这种寻路的方法是路径最短的。但是智能体在训练初期相比于`SARSA` 很容易掉入悬崖。而对于 `SARSA`，智能体会远离悬崖移动到终点，因为掉入悬崖会得到很坏的回报，由于算法考虑“长期”的回报，所以智能体会尽可能的远离悬崖。



# 5. DQN

## 5.1 深度神经网络

觉得讲的挺有意思，记录一下：

现在我们可以把深度神经网络的Magic函数，看成是一个数据加工厂。而X就是要进行加工的数据。

<img src="https://pic4.zhimg.com/80/v2-c9ad25f1c893fecd30c82e9a17506227_720w.jpg" alt="img" style="zoom: 80%;" />

为了让这个数据加工厂运行得更快，通常我们需要把要加工的数据X变得更‘标准’一些。

例如图片的尺寸大小，有多少通道的颜色等等，然后分批(一般称为batch)，输入工厂。

在输入工厂的时候，会有一个‘大门’，我们称为输入层，去检查数据是否已经按照工厂的标准整理好。

数据工厂里有很多车间，按照流水线排列。和一般的自动化车间一样，我们需要定义好这个车间的操作标准。

我们一般称这些车间叫**层**。这些层都已经封装好在tensorflow、tensorlayer、pytorch等里面了。常用的层包括：Dense、Conv2D、LSTM、Reshape、Flatten等。

最终，数据工厂会把原数据X，加工成产品y'(也叫做：logits)。从源数据加工成产品的过程，我们叫**正向传播**。

但产品y'是否是一个合格的产品，我们还需要我们真正的y(也叫做:lables)作为标准去鉴定。我们把鉴定出来的差距就是loss。

工厂根据鉴定结果，以**梯度下降**的方式，**反向传递**给每个车间，告诉车间要如何调整各自的参数，让源数据和产出y'能够对应起来。

经过N个批次（batch）的数据输入，然后鉴别，工厂调整。最后工厂就能达到我们的生产标准了。也就是说magic函数已经被训练好了。

## 5.2 理解

**Deep network + Qlearning = DQN**

Qlearning中通过Qtable记录各个状态下的Q值，其作用是输入状态S，通过查表返回能够获得最大Q值的动作A，即找出S-A的对应关系。

但在现实生活中，很多状态并不是**离散**而是**连续**的，遇到这些情况，Qtable就没有办法解决。所以可以用一个函数F表示：F(S) = A，这样就可以不用查表了，而且函数还允许**连续**状态的表示。神经网络就可以看成一个万能的函数。

这个万能函数接受输入一个状态S，它能告诉我，每个动作的Q值是怎样的。

例如，对Qtable三维化：

<img src="/home/wca/Pictures/2021-07-20 11-10-22 的屏幕截图.png" style="zoom: 60%;" />

用函数来表示，**相当于我们要扭曲一条曲线，这条曲线穿过了离散状态下的所有点：

<img src="/home/wca/Pictures/2021-07-20 11-11-44 的屏幕截图.png" style="zoom: 60%;" />

此时不但可以取状态3和状态4，我们还可以取状态3.5的Q值

- 其实 `Qlearning` 和 `DQN` 并没有根本的区别
- 只是 `DQN` 用神经网络，也就是一个函数替代了原来Qtable而已

## 5.3 神经网络的目标

首先看，在 `Qlearning`，我们用下一状态 St+1 的最大Q值替代 St+1 的V值，V(St+1) 加上状态转移产生的奖励R，就是 Q(S,a) 的更新目标。

那么再看 `DQN`：

<img src="/home/wca/Pictures/2021-07-20 11-20-30 的屏幕截图.png" style="zoom: 67%;" />

- 假设我们需要更新当前状态St下的某动作A的Q值：Q(S,A)，我们可以这样做： 
  1. 执行 `A`，往前一步，到达 `St+1` 
  2. 把 `St+1` 输入`Q网络`，计算 `St+1` 下所有动作的 `Q值` 
  3. 获得最大的 `Q值` 加上 `奖励R` 作为更新目标
  4. 计算损失 `-Q(S,A)` 相当于有监督学习中的logits - maxQ(St+1) + R 相当于有监督学习中的lables - 用mse函数，得出两者的 `loss` 
  5. 用 `loss` 更新 `Q网络`

也就是，我们用 Q网络 估算出来的两个相邻状态的 Q值，他们之间的距离，就是一个 R 的距离。

<img src="/home/wca/Pictures/2021-07-20 11-27-41 的屏幕截图.png" style="zoom: 67%;" />

## 5.4 总结

1. 其实 `DQN` 就是 `Qlearning` 扔掉 `Qtable`，换上 `深度神经网络`。
2. 我们知道，解决连续型问题，如果表格不能表示，就用函数，而最好的函数就是深度神经网络。
3. 和有监督学习不同，深度强化学习中，我们需要自己找更新目标。通常在马尔科夫链体系下，两个相邻状态状态差一个奖励R经常能被利用。

## 5.5 应用

1. 初始化一个网络，用于计算Q值。（在Qlearning，用Qtable）F开始一场游戏
2. 随机从一个状态S开始
3. 我们把S输入到Q，计算S状态下，A的Q值 Q(S)
4. 我们选择能得到最大Q值的动作A
5. 我们把a输入到环境，获得新状态S’，R，done
6. 计算目标 y = R + gamma * maxQ(S') 
7. 训练Q网络，缩小 Q(S, A) 和 y 的大小
8. 开始新一步，不断更新

## 5.6 固定Q目标 (Fixed Q-targets)

DQN的目标：

<center> gamma * maxQ(s') + R </center>

可见目标本身就包含一个Q网络，这样会造成Q网络的学习效率比较低，而且不稳定。

> 如果把训练神经网络比喻成射击游戏，在target中有Q网络的话，就相当于在射击一个移动靶，因为每次射击一次，靶就会挪动一次。相比起固定的靶，无疑加上了训练的难度。那怎么解决这个问题呢？既然现在是移动靶，那么我们就把它弄成是固定的靶，先停止10秒。10后挪动靶再打新的靶。这就是Fixed Q-targets的思路。

<img src="/home/wca/Pictures/2021-07-20 11-44-14 的屏幕截图.png" style="zoom: 67%;" />

在实做的时候，其实和原来的DQN一样，唯一不同点是，我们用两个Q网络：

- 一个是原来的Q网络，用于估算Q(s);

- 另外一个叫targetQ网络, targetQ自己并不会更新，也就是它在更新的过程中是固定的，用于计算更新目标。

<center>y = R + gamma * max(targetQ(s'))</center>

我们进行N次更新后，就把新Q的参数赋值给旧Q。

## 5.7 Double DQN

DQN有一个显著的问题，就是DQN估计的Q值往往会偏大。

这是由于Q值是以下一个s'的Q值的最大值来估算的，但下一个state的Q值也是一个估算值，也依赖它的下一个state的Q值……这就导致了Q值往往会有偏大的的情况出现。

- 如果有两个Q网络，因为两个Q网络的参数有差别，所以对于同一个动作的评估也会有少许不同。我们选取评估出来较小的值来计算目标。这样就能避免Q值偏大的情况发生了。
- 用两个Q网络：Q1网络**推荐**能够获得最大Q值的动作；Q2网络计算这个动作在Q2网络中的Q值。

恰好，此时用上Fixed Q-targets，就有两个Q网络了。

代码分析见：[Double DQN原理是什么，怎样实现？（附代码）](https://zhuanlan.zhihu.com/p/110769361)，包含了2个对于DQN很重要的技巧：经验回放和固定目标网络，也用上了双Q网络计算目标。



# 6. 策略梯度 (Policy Gradient) 

在之前的学习中，从 `动态规划` 到 `蒙特卡洛` ，到 `TD` 到` Qlearning` 再到 `DQN` ，一路为计算Q值和V值绞尽脑汁，但算Q值和V值并不是我们最终目的呀，我们要找一个策略，能获得最多的奖励。我们可以抛弃掉Q值和V值么？

策略梯度(Policy Gradient)算法就是这样一个算法

## 6.1 策略梯度算法 (PG)

- 如果说 DQN 是一个 TD + 神经网络 的算法
- 那么 PG 就是一个 MC + 神经网络 的算法

PG算法通过MC的G值来衡量智能体动作的对错

## 6.2 直观感受PG算法

从某个state出发，可以采取三个动作

假设当前智能体对这一无所知，那么，可能采取平均策略 0 = [33%,33%,33%]

智能体出发，选择动作A，到达最终状态后开始回溯，计算得到 G = 1

<img src="/home/wca/Pictures/2021-07-21 16-56-34 的屏幕截图.png" style="zoom: 67%;" />

我们可以更新策略，因为该路径**选择了A**而产生的，**并获得G = 1**；因此我们要更新策略：让A的概率提升，相对地，BC的概率就会降低。 计算得新策略为： 1 = [50%,25%,25%]

虽然B概率比较低，但仍然有可能被选中，第二轮刚好选中B

智能体选择了B，到达最终状态后回溯，计算得到 G = -1

<img src="/home/wca/Pictures/2021-07-21 16-57-19 的屏幕截图.png" style="zoom:67%;" />

所以我们对B动作的评价比较低，并且希望以后会少点选择B，因此我们要降低B选择的概率，而相对地，AC的选择将会提高

计算得新策略为： 2 = [55%,15%,30%]

最后随机到C，回溯计算后，计算得G = 5

<img src="/home/wca/Pictures/2021-07-21 16-57-59 的屏幕截图.png" style="zoom:67%;" />

C比A还要多得多。因此这一次更新，C的概率需要大幅提升，相对地，AB概率降低。 3 = [20%,5%,75%]

## 6.3 结论

PG用一个全新的思路解决了问题。但实际效果显得不太稳定，在某些环境下学习较为困难。另外由于采用了MC的方式，需要走到最终状态才能进行更新，而且只能进行一次更新，这也是GP算法的效率不高的原因。



# 7. Actor-Critic

## 7.1 AC算法

**AC 源于 PG**

由于蒙特卡洛需要完成整个游戏过程，直到最终状态，才能通过回溯计算G值。这使得PG方法的效率被限制

于是将MC改为TD可以提高效率，可以用神经网络估算每一步的Q值

也就是说，Actor-Critic，其实是用了两个网络：

- 一个输入状态S，输出策略，负责选择动作，我们把这个网络成为Actor
- 一个输入状态S，负责计算每个动作的分数，我们把这个网络成为Critic

可以理解为Critic根据Actor的行为打分，Actor通过Critic给出的分数，去学习：如果Critic给的分数高，那么Actor会调整这个动作的输出概率；相反，如果Critic给的分数低，那么就减少这个动作输出的概率。

可以理解为AC是TD版本的PG

## 7.2 TD-error

在DQN预估的是Q值，而在AC中的Critic，估算的是V值。这是因为直接用network估算的Q值作为更新值，效果会不太好，原因如图：

<img src="/home/wca/Pictures/2021-07-21 17-22-15 的屏幕截图.png" style="zoom:50%;" />

假设我们用Critic网络，预估到S状态下三个动作A1，A2，A3的Q值分别为1,2,10

但在开始的时候，我们采用平均策略，于是随机到A1。于是我们用策略梯度的带权重方法更新策略，这里的权重就是Q值

于是策略会更倾向于选择A1，意味着更大概率选择A1。结果A1的概率就持续升高...

这是因为Q值用于是一个正数

要解决这个问题，我们可以通过减去一个baseline，令到权重有正有负。而通常这个baseline，我们选取的是权重的平均值。减去平均值之后，值就变成有正有负了

**而Q值的期望(均值)就是V**

<img src="/home/wca/Pictures/2021-07-21 17-25-15 的屏幕截图.png" style="zoom: 60%;" />

所以我们可以得到更新的权重：

<center>Q(s,a) - V(s)</center>

根据马尔科夫链，Q(s,a) 可以用 gamma * V(s') + r 来代替

得到

<center>gamma * V(s') + r - V(s)</center>

对比DQN，DQN的更新用了Q，而TD-error用的是V

**因此，Critic的任务就是让TD-error尽量小。然后TD-error给Actor做更新**

现在我们再总结一下TD-error的知识：

1. 为了避免正数陷阱，我们希望Actor的更新权重有正有负。因此，我们把Q值减去他们的均值V，有：Q(s,a) - V(s)
2. 为了避免需要预估V值和Q值，我们希望把Q和V统一；由于Q(s,a) = gamma * V(s') + r，所以我们得到TD-error公式： TD-error = gamma * V(s') + r - V(s)
3. TD-error就是Actor更新策略的时候，带权重更新中的权重值；
4. 现在Critic不再需要预估Q，而是预估V。而根据马可洛夫链所学，我们知道TD-error就是Critic网络需要的loss，也就是说，Critic函数需要最小化TD-error。

## 7.3 算法

1. 定义两个网络：Actor 和 Critic

2. 进行 N 次更新。

3. 1. 从状态 s 开始，执行动作 a，得到奖励 r，进入状态 s'
   2. 记录的数据。
   3. 把输入到 Critic，根据公式： TD-error = gamma * V(s') + r - V(s) 求TD-error，并缩小TD-error
   4. 把输入到 Actor，计算策略分布 。

<img src="/home/wca/Pictures/2021-07-21 17-32-38 的屏幕截图.png" style="zoom:60%;" />

## 7.4 总结

理解 TD-error 本质是 Q(s,a) - V(s) 来的，而 Q(s,a) 转为由 V(s') + r 的形式表示

AC很强，强在它有无限的潜力，AC出来之后，就有很多厉害的算法基于AC框架，其中就包括PPO算法。



# 8. PPO

PPO算法适用度特别广

PPO是基于AC架构的，也就是说，PPO也有两个网络，分别是Actor和Critic，这是因为AC架构有一个好处。这个好处就是解决了连续动作空间的问题。

## 8.1 连续动作的输出

在离散动作空间的问题中，最终输出的策略呈现这样的形式：

<img src="https://pic2.zhimg.com/80/v2-e125990a3eb3a9e1d88bd2aa6a61c085_720w.jpg" alt="img" style="zoom: 80%;" />

同理，假如有多个动作，同样可以这样表示：

<img src="https://pic2.zhimg.com/80/v2-5095e3e1c64efba0fac609fc82416fe9_720w.jpg" alt="img" style="zoom:80%;" />

连续动作的表示可以理解为把连续型概率切成很多很多份，每一份的数值，就代表该动作的概率：

<img src="https://pic4.zhimg.com/80/v2-551ad9aef5390691aded75b6ae9d70f7_720w.jpg" alt="img" style="zoom: 60%;" />

在连续型中，我们不再用数组表示，而是用函数表示。例如，策略分布函数：P = (action) 代表在策略下，选择某个action的概率P。

但是用神经网络预测输出的策略是一个固定的shape，而不是连续的

我们可以假定策略分布函数服从正态分布，就只需要 sigma 和 mu 两个参数表示即可

- mu 表示平均数，也就是整个正态分布的中轴线。mu 的变化，表示整个图像向左右移动。
- sigma 表示方差，当 sigma 越大，图像越扁平；sigma 越小，图像越突出。而最大值所在的位置，就是中轴线。

神经网络可以直接输出 mu 和 sigma，就能获得整个策略的概率密度函数

这样要按概率选出一个动作时，只需要按照整个密度函数抽样出来即可

## 8.2 AC的问题 

上节证明了 AC 的框架既能处理连续状态空间问题，也能处理连续动作空间问题

但 AC 还有一个问题。AC 产生的数据，只能进行1次更新，更新完就只能丢掉，等待下一回合的数据。此时数据是浪费的，为什么不能像 DQN 那样把以前的数据存起来，更新之后用呢？

首先理清两个概念：

- **行为策略**——不是当前策略，用于**产出数据**

- **目标策略**——会更新的策略，是需要**被优化的策略**

如果两个策略是**同一个策略**，那么我们称为**On Policy**，**在线策略**。如果**不是同一个策略**，那么**Off Policy**，**离线策略**。

例如 PG 就是一个在线策略。因为 PG 用于产生数据的策略（行为策略）和需要更新的策略（目标策略）是一致。

而 DQN 则是一个离线策略。我们会让智能体在环境互动一定次数，获得数据。用这些数据优化策略后，继续跑新的数据。但老版本的数据我们仍然是可以用的。也就是说，我们产生数据的策略，和要更新的目标策略不是同一个策略。所以 DQN 是一个离线策略。

但为什么 PG 和 AC 中的 Actor 更新，就不能像 DQN 一样，把数据存起来，更新多次呢？

- 简化的例子：

现在两个策略，分别是P和B：

<center>P: [0.5, 0.5]   B: [0.1, 0.9]</center>

现在我们按照两个策略，进行采样；也就是分别按照这两个策略，以S状态下出发，与环境进行10次互动，获得如图数据。

![img](https://pic2.zhimg.com/80/v2-70462cacabead8c76b1db4328711d1fd_720w.jpg)

如果用行动策略 B[0.1, 0.9] 产出的数据，对目标策略P进行更新，动作1会被更新1次，而动作2会更新9次。虽然动作A的 TD-error 比较大，但由于动作2更新的次数更多，最终动作2的概率会比动作1的要大。

总结：

1. 在 DQN 中更新Q值，和策略无关。 在同一个动作出发，可能会通往不同的state，但其中的概率是有环境所决定的，而不是我们的策略所决定的。所以我们产生的数据和策略并没有关系。
2. 在 DQN 的更新中，我们是有"目标"的。 虽然目标比较飘忽，但每次更新，其实都是尽量向目标靠近。无论更新多少次，最终都会在目标附近徘徊。但 PG 算法，更新是不断远离原来的策略分布的，所以远离多少，远离的次数比例，我们都必须把握好。

## 8.3 Important-sampling

PPO 是通过 **重要性采样技术**（Important-sampling）做到的离线更新策略。

例如，如果我们想用策略B抽样出来的数据，要把TD-error乘以一个重要性权重（IW: importance weight）

<center>IW = P(a) / B(a)</center>

将其应用在PPO，就是 `目标策略 P` 出现动作a的概率除以 `行为策略 B` 出现a的概率

还是上节的例子，P: [0.5, 0.5]; B: [0.1, 0.9]

<img src="https://pic2.zhimg.com/80/v2-fe8a235dd4626287e0bbf7d49a674be5_720w.jpg" alt="img" style="zoom:90%;" />

把重要性权重乘以 TD-error，就会发现，a1 的 TD-error 大幅提升，而 a2 的 TD-error 减少了。现在即使我们用P策略[0.5,0.5] 进行更新，a1 提升的概率也会比 a2 的更多。

> PPO应用了importance sampling，使得我们用行为策略获取的数据，能够更新目标策略，把AC从在线策略，变成离线策略。

## 8.4 N步更新

<img src="/home/wca/Pictures/2021-07-22 11-57-12 的屏幕截图.png" style="zoom: 67%;" />

## 8.5 总结

1. 我们可以用AC来解决连续型控制问题。方法是输入mu和sigma，构造一个正态分布来表示策略； 
2. PPO延展了TD(0)，变成TD(N)的N步更新； 
3. AC是一个在线算法，但为了增加AC的效率，我们希望把它变成一个离线策略，这样就可以多次使用数据了。为了解决这个问题，PPO使用了重要性采样。

但实际上，P策略和B策略差异并不能太大，为了能处理这个问题，有两个做法，PPO1 和 PPO2 。













# 9. DDPG









































