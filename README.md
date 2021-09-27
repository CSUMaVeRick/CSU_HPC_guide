# CSU​高性能计算:computer:平台使用指南

去年我南开放了HPC平台的使用，这个平台的性能是很:cow:的​，也就全国第一叭。

最近由于业务上的原因，笔者申请了账号，并进行了体验，整个流程中出现了不少问题。所幸最后终于掌握了平台的基本使用方法，故将心得体会整理成了本文。（就是半夜提交任务排队要很久，大伙都是夜猫子选手吗？哈哈哈哈哈哈哈）

## Slurm以及MobaXterm

学校的平台使用Slurm作业调度系统（Simple Linux Utility for Resource Management）。这是开源、具有容错性和高度可扩展大型和小型Linux集群资源管理和作业调度系统。所有需运行的作业无论是用于程序调试还是业务计算均必须通过交互式并行`srun`、批处理式`sbatch`或分配式`salloc`提交，提交后可以利用相关命令查询作业状态等。**请不要在登录节点直接运行作业**。

MobaXterm是一款简单好用且强大的远程登录软件。使用它来登录CSUHPC十分方便。下载安装完成后，只需新建一个session，点击连接，就能连接到CSUHPC上。**注意，一定要使用校内IP**。

成功连接后显示的内容如下。

```
################################################################################

 Welcome to visit High Performance Computing Platform of Central South University!
 Email: hpc@mail.csu.edu.cn
 Tel: (0731)-88877295,88830428
 Web: http://hpc.csu.edu.cn/

################################################################################

(*)Hardware information:
 node0001~node1022: 2 x Intel(R) Xeon(R) Gold 6248R CPU @ 3.0GHz, 48 Cores, Mem 192G.
 gpu201~gpu210: 2 x Intel(R) Xeon(R) Gold 6248 CPU @ 2.50GHz, 40 Cores, Mem 384 G, 2 x Nvidia Tesla V100s.
 gpu401~gpu412: 2 x Intel(R) Xeon(R) Gold 6248 CPU @ 2.50GHz, 40 Cores, Mem 384 G, 4 x Nvidia Tesla V100.
 gpu801~gpu804: 2 x Intel(R) Xeon(R) Gold 6248 CPU @ 2.50GHz, 40 Cores, Mem 384 G, 8 x Nvidia Tesla V100.
 fat01~fat10: 4 x Intel(R) Xeon(R) Gold 6248 CPU @ 2.50GHz, 80 Cores, Mem 1536 G.
(*)Important directory:
 Softwares:/public/software;
 Job templates: /public/job_templates;
(*)To enable Intel Compiler environment:
source /public/software/intel/intel2019/intel2019-env.sh
(*)Quick start manual & FAQ:
https://nic.csu.edu.cn/info/1146/1790.htm
https://nic.csu.edu.cn/info/1146/1806.htm
```

可以看到CSUHPC的配置还是很硬的。

## 快速使用步骤

首先来到你的用户文件，其目录为/public/home/<账号>。新建一个文件夹，如document，在右侧的终端中cd到document里，方便我们后面使用。

接着将自己想要运行的程序文件，如test.py，拖动到document中。**请务必确认程序能够在unix系统中运行**。由于笔者是Windows选手，所以研究了很久才弄明白正确运行程序需要事先进行转换。一个方便的命令是`dos2unix`。

在document下新建一个名为job.sh的文件，使用MobaXterm的默认编辑器打开进行编辑。输入以下内容

```sh
#!/bin/bash

#SBATCH -o test.%j.out ##作业的输出信息文件
#SBATCH -J test ##作业名
#SBATCH -p cpuQ ##使用cpuQ计算队列
#SBATCH --nodes=1 ##申请1个节点
#SBATCH --ntasks-per-node=10 ##每节点运行10个任务（每节点调用10核）

source activate py3.9
python test.py
```

在终端中输入`sbatch job.sh`就可以用批处理方式提交任务啦。

## 环境搭建

使用超算真有这么简单？`sbatch`一下就可以嗖嗖嗖运算了？当然不是。在提交任务前，我们要确保任务执行所需要的环境是齐全的。

以Python为例，就是要确保正确搭建Python，并在环境中安装必要的第三方库。幸运的是，CSUHPC已经在公共区域内安装了Anaconda3。这大幅减少了我们的工作量。

只要用conda命令检查我们要用到的库在不在已安装的库列表中。如果没有的话，就`conda install`。很有意思的一点在于，也可以使用`pip install`来安装。学校已经接入清华tuna的镜像源，所以一般不需要再额外配置国内镜像。

环境搭建完成后，我们就可以开心快乐地`sbatch`了。

## 检查过程，确认结果

我们可以使用以下常用命令来查看任务的状态或控制任务

| 命令                | 功能                                                         |
| ------------------- | ------------------------------------------------------------ |
| squeue              | 查看任务队列的状态，需要关注JOBID以及NODES NODELIST(REASON)。 |
| sacct -j <任务编号> | 查看指定id的任务的运行情况，有PENDING等待、RUNNING运行中、COMPLETED完成、FAILED失败。 |
| sacct -a            | 查看系统中全部任务的运行情况，可以借此推测还需要排多久的队。 |
| scancel <任务编号>  | 取消指定任务编号的任务。                                     |

除此以外，Slurm还有许多的控制命令，详请参见附录。

## 参考

1. 一位学长的CSDN帖子https://blog.csdn.net/weixin_42279314/article/details/109677459

2. 中国科大的Slurm操作手册http://hmli.ustc.edu.cn/doc/userguide/slurm-userguide.pdf
3. 中南大学信网中心的使用指南https://nic.csu.edu.cn/info/1146/1790.htm
