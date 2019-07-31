+++
date = "2017-08-18T21:10:37+08:00"
title = "工程仿真"
showonlyimage = false
image = "/img/blog/dual-os-3-setup-arch/setup-Arch.png"
draft = true
weight = 81
+++


<!--more-->

John A. Swanson，ANSYS公司的创始人，曾获美国工学最高荣誉“福瑞兹”奖。老爷子聊起他年轻时候，是如何做模拟和仿真的——不免又是那些打孔纸片和大型机开始——逐渐模块化，封装完善为业界最牛逼的工程仿真模拟软件 ANSYS tool。老爷子讲道他和他夫人更多的是在做一些慈善，将财富回馈给大学和国家，希望他驾鹤西去的那一天所有的财富也都回馈完成。达到收支平衡的境界。

结构力学，流体力学，热传导

输入：
- 几何结构(geometry)
- mesh
- boundary conditions
- material properties

输出：
- 彩色图

不敷于表面结论，避免 garbage in garbage out，所以不要黑盒方式。知其然，知其所以然。

将现实变为一个物理问题，由此界定各种物理参数的数值
然后分析工具依据这些输入，加上一些物理原理(静力平衡分析)抽象为数据模型。

关键点是

- 抽象出的数学模型是什么
- 我们基于的物理原理是什么
- 这些数学模型的前提假设是什么

之后工具会针对这个数学模型产生一个数值解，然后针对其中哪些选中变量，再做相应计算。(这就是为啥叫有限元分析吧？)

对固体仿真来说，就选 displacement，选择的变量可能是边、角、中心
对流体仿真来说，就选 压力和流速

要了解哪些是被直接结算出来的，而哪些又是基于这些结果，再经后处理后生成的。

另外我们可以从物理问题出发
- 做一些手工计算，加上一些经验数据(empirical data),把输出结果和工具计算出的作比较
- 做一些实验，根据实验数据做比较，但这部分各个公司都是

下面讲预分析步骤

第一步，预分析

- 思考分析其数学模型和基于的物理原理，和显示、隐式的预设前提
- 数值解的策略，引入了何种误差，如何将误差最小化
- 提前手工计算出预期值，以便后续对结果、趋势的比较验证

最后一步，检验及实证

趋势是将实体测试最小乃至去除，当同时为了保证正确性，我们需要思考

- 这个模型处理的时候正确？即检验 (Verification)
  - 生成的渲染图是否符合对应的数学模型及其边界条件，是否仍符合物理定律(受力平衡、动量守恒)
  - 计算过程中误差是否可接受——比如网格化计算中近似模拟
  - 模拟结果和手工计算是否一致

持续这些检验能使得我们相关的判定越来越准确，经验越来越丰富。而在进一步了解了当前模型后，我们需要进一步检验，

- 这个模型是否选择的正确？即 validation
  - 数学模型是否漏掉了问题的某些重要的部分，是否做了过度简化
  - 能否结合实验数据做一定程度的实体验证

有限元分析

一维热传导案例

假设棒统一截面(y,z)上温度一直，仅分析其长度方面(x轴)。
仅分析一个最小控制体积 x 方向上的净热出量，见[热传导] (https://en.wikipedia.org/wiki/Thermal_conduction)
结合傅里叶热传导定律、能量守恒，推导出支配方程及边界条件。成为典型的边界值问题，支配方程决定域内，边界条件决定域的边界。解微分方程即可。

http://blog.sciencenet.cn/blog-528739-1014825.html