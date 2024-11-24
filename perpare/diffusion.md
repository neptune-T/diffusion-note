扩散模型的出现重塑了生成建模的格局，为解决复杂数据分布提供了一种创新方法。受扩散的物理过程启发，这些模型在正向过程中将数据转换为噪声，然后逆向转换以生成连贯而真实的输出。这种迭代去噪过程可以实现无与伦比的高维数据重建精度，使扩散模型成为现代生成框架的基石。

与 GAN 等对抗性训练方法不同，扩散模型依赖于基于可能性的优化框架，从而避免了对抗性设置中经常出现的不稳定性。强大的训练范式，加上其生成多样化和高保真输出的能力，使扩散模型成为从图像合成到文本到图像生成和风格转换等领域的多功能工具。

![[Pasted image 20241124150124.png]]

在正向过程中，目标是通过在 \( T \) 个离散时间步中添加高斯噪声来迭代破坏数据样本 \( x_0 \)。给定上一步 \( t-1 \)，步骤 \( t \) 处损坏数据的条件分布定义为：

\[
q(x_t|x_{t-1}) = \mathcal{N}(x_t; \sqrt{1 - \beta_t}x_{t-1}, \beta_t I),
\]

其中 \( \beta_t \) 表示控制每一步添加的噪声量的方差计划。在连续的步骤中，数据逐渐转变为纯高斯噪声分布​​。这种累积效应是通过将 \( x_t \) 直接关联到初始数据 \( x_0 \) 来捕获的：

\[
q(x_t|x_0) = \mathcal{N}(x_t; \sqrt{\bar{\alpha}_t}x_0, (1 - \bar{\alpha}_t)I),
\]

其中 \( \bar{\alpha}_t = \prod_{s=1}^t (1 - \beta_s) \) 为噪声计划的累积乘积。此属性确保了数学上可处理的正向过程并有助于高效采样。

旨在撤消正向扩散的反向过程学习逐步对 \( x_t \) 进行去噪以重建 \( x_0 \)。与正向过程不同，反向过程使用神经网络 \( p_\theta(x_{t-1}|x_t) \) 进行参数化，因为真正的反向分布 \( q(x_{t-1}|x_t) \) 是难以处理的。此反向过程可以表示为：

\[
p_\theta(x_{t-1}|x_t) = \mathcal{N}(x_{t-1}; \mu_\theta(x_t, t), \Sigma_\theta(x_t, t)),
\]

![[Pasted image 20241124150401.png]]

与朗之万之间的联系

朗之万动力学是物理学中的一个概念，用于对分子系统进行统计建模。结合随机梯度下降，_随机梯度朗之万动力学_（[Welling & Teh 2011](https://www.stats.ox.ac.uk/~teh/research/compstats/WelTeh2011a.pdf)）可以从概率密度中产生样本页（十）仅使用渐变∇十
日志⁡页（十）在马尔可夫更新链中：
![[Pasted image 20241124150437.png]]

与标准 SGD 相比，随机梯度朗之万动力学将高斯噪声注入参数更新中，以避免陷入局部最小值。![[Pasted image 20241124150518.png]]



反向过程旨在消除正向扩散，它学习逐步消除 \( x_t \) 的噪声以重建 \( x_0 \)。与正向过程不同，反向过程使用神经网络 \( p_\theta(x_{t-1}|x_t) \) 进行参数化，因为真正的反向分布 \( q(x_{t-1}|x_t) \) 是难以处理的。这个逆过程可以表示为：

\[
p_\theta(x_{t-1}|x_t) = \mathcal{N}(x_{t-1}; \mu_\theta(x_t, t), \Sigma_\theta(x_t, t)),
\]

其中 \( \mu_\theta \) 和 \( \Sigma_\theta \) 是学习到的参数，表示逆高斯分布的均值和方差。通过迭代应用这些学习到的变换，模型从纯噪声开始生成真实的数据。

训练扩散模型涉及最小化实际反向转换 \( q(x_{t-1}|x_t, x_0) \) 与其参数化近似值 \( p_\theta(x_{t-1}|x_t) \) 之间的差异。这是通过变分下限 (VLB) 目标实现的：

\[
\begin{aligned}
L_{\text{VLB}} = \mathbb{E}_q \bigg[
\sum_{t=1}^T & D_{\text{KL}}(q(x_{t-1}|x_t, x_0) \| p_\theta(x_{t-1}|x_t)) \\
& - \log p_\theta(x_0|x_1)
\bigg].
\end{aligned}
\]

此目标确保逆向过程能够学会从噪声中准确地重建数据分布。通过利用这种逐步细化，扩散模型不仅可以在捕捉高维数据分布的复杂性的同时实现稳定高效的训练，还可以避免 GAN 中常见的对抗性训练陷阱，例如模式崩溃。