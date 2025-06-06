# Solar-Eclipse

2025.03.16：上传celestral_body.py 定义了CelestralBody库，初始化了各大行星的位置，包含GetInitials等函数——lrd

2025.3.22 上传了一个notebook, 里面基于celestral_body.py 尝试了三级三阶显式Heun方法计算50年轨道，单位1h为时间步长，进一步写了计算轨道动画的函数和预测日月食的函数，最后一步有报错尚未调整。——文哲

2025.3.24 https://www.overleaf.com/4832584493smgjhjxdqwfr#d5fed2 省流：增加了非球形近似、观测段近似的讨论，给了一个不需要强解N体问题的考虑大行星运动的方案，时间预测误差界4e-7±20s ——jhj

2025.4.1 
- 更新了 Runge-Kutta 方法，发现Heun数值模拟与 Runge-Kutta 在10年内几乎没有区别
- 改进步长至0.5h，更新起始速度为2025年1月1日0时0分前后3小时的平均速度
- 问题：无论采用那种近似，都会逐渐走向失控：月球脱离地球引力环绕太阳运动，进一步演示说明误差是逐步累积的（第3-3.5年月球轨迹只有四个波峰）
- 可能的原因：（感觉可靠性依次降低）
  - 起始数据有偏差，这种偏差在后面逐步扩大
  - 模型有错误（例如力的计算）
  - 地月系之间与日地系统之间的差距过大，python数值省略了？
  - 缺少外部星体影响，没有完整刻画日地月系统/系统不应该是一个三体问题
- 文哲提出的bug有待修复
- 相关文件保存在v1里 ——lrd
2025.4.2凌晨：把步长改到3min解决辣

2025.4.2
- 在v1_1中添加了大行星的影响并初步测试了性能
- 结论（待更新）：
  - 大行星对绝对位置有着不可忽略的影响，但任务关心的是日地月系统的相对位置，影响仍待确定
  - 求解4体方程的用时为3体方程用时的1.5倍，求解5体方程的用时为3体方程的2.5倍（在本地跑波动比较大），暂时没有看到直接推广带来的异常结果
  - 两种方法：解N体运动、解带外场的3体运动所得结果有细微差别，将大行星作为外场的策略在仅考虑一个额外天体的情况下无明显加速（因为主要复杂度来自多算了一对作用力），可作为需要同时考虑更多天体时的备选方案，关于精度和加速的取舍需要进一步考察对日食时间测算的影响来确定 ——jhj

2025.4.4
- 整理代码，接口和使用方法可参考main.py，之后的更新尽量和新版代码保持一致
- 修改了4阶r-k方法的代码，目前取时间步t=10即可收敛
- 写了批量判定某个几何位形是否发生日食的函数，是否发生月食的函数类似，有待后续处理将其变为直接预测开始/结束/食甚时刻的形式 
- 调试后初步用程序和标注时间对照(欲复现结果，用最新版代码直接运行python main.py)，发现预测在8y的范围内时间相差在min量级
- 发现在初始速度估计中选择不同的time range会明显改变长期预测的结果，换用直接读取初始位置和速度数据 ——jhj

2025.4.5
- 上传了possible_approach文件夹，做了一定修改
- 补全了月食检测函数
- 提出了一种计算思路——在用较粗时间步更新轨道的过程中不断检测日月食，检测到发生后切换至细步长，直到日月食结束再切换回粗步长。
- 目前不清楚为什么轨道没在更新，仍需调试。---swz

2025.4.8
- 补全了输出结果的相关函数，由于缩小时间步作模拟操作复杂且容易带来误差，采用对临界区域作线性插值的方式估算发生日/月食的准确时间
- constants中部分常数有错误或是精度不够高，已修复，同时所有的G取为1，而将天体的质量更换为GM的乘积，其余文件中相关的部分也已经改写
- check_eclipse之前写的月食检测函数有误，已修复
- 由于几何位形检测用时占比很小，在这部分使用float64进行高精度计算，避免低精度时计算结果误差过大无法复现的问题（后续如仍有此问题可进一步增加精度）
- 初步编写了输出的格式，所有输出重定向到record.log文件中，main.py运行得到的样例输出已一并上传，目前在考虑木星影响的条件下与标准数据50年尺度下相差的在10min量级，剩余误差来源有待进一步分析 ——jhj


2025.4.9
- 上传了用于对预测结果评估的函数evaluate.py,并且将NASA官网上的日食月食数据打包成了本地的json文件nasa_eclipse_data.json便于读取。
- 目前轨道精度下50年内的全部日食和月食100%预测，除了输出有两次日食出现了两个total同时输出的bug尚未修复。其余已跑通。
- 之后调参数运行evaluate.py 文件中可能需要做少许修改，目前是在main函数中直接调用了main.py预测的输出文件record.log，并且设置了1h的时间差作为事件预测匹配的条件。——swz

2025.4.10
- 上传了分析结果用于作图的plot_errors.py 和plot_trajectory_results.ipynb, 和作出的图像，轨道部分还需调整，目前采用标准数据是de440.bsp,文件太大无法上传，使用时需本地自行去官网下载数据。
- 简单写了下事件预测结果误差的分布的实验结果。数值比对实验未开展。更新了report ---swz

2025.4.9 - 4.11
- 整理代码，修补了检查日食的算法，做了理论分析和实际验证
- 发现全环食现象，加以分析，最后组内决定将其定位为日全食
- 编辑最终报告的2.5、2.6、3.1-3.3，并把大家的报告汇总，最终报告保存在reports
- 加入jacobi.py计算Runge-Kutta方法的最大模，试图说明其精确性，不知道有没有成功
- 完结撒花！ ---lrd
- 最终报告小组成员承担部分（好像是这样的）：
  - swz: 第一章
  - lrd: 2.5-2.6, 第三章
  - jhj: 2.1-2.4, 第四章
