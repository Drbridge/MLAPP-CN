# MLAPP 读书笔记 - 10 离散图模型(Directed graphical models)(贝叶斯网络(Bayes nets))

> A Chinese Notes of MLAPP,MLAPP 中文笔记项目 
https://zhuanlan.zhihu.com/python-kivy

记笔记的人：[cycleuser](https://www.zhihu.com/people/cycleuser/activities)

2018年7月1日18:59:12

## 10.1 概论

>以简单方式对待复杂系统的原则,我基本知道两个:首先就是模块化原则,其次就是抽象原则.我是机器学习中计算概率的辩护者,因为我相信概率论对这两种原则都有深刻又有趣的实现方式,分别通过可分解性和平均.在我看来,尽可能充分利用这两种机制,是机器学习的前进方向.                      Michael Jordan, 1997 (转引自 (Frey 1998)).

假如我们观测多组相关变量,比如文档中的词汇,或者图像中的像素,再或基因片段上的基因.怎么能简洁地表示联合分布$p(x|\theta)$呢?利用这个分布,给定其他变量情况下,怎么能以合理规模的计算时间来推导一系列的变量呢?怎么通过适当规模的数据来学习得到这个分布的参数呢?这些问题就是概率建模(probabilistic modeling),推导(inference)和学习(learning)的核心,也是本章主题了.

### 10.1.1 链式规则(Chain rule)

通过概率论的链式规则,就可以讲一个联合分布写成下面的形式,使用任意次序的变量都可以:

$p(x_{1:V} ) = p(x_1)p(x_2|x_1)p(x_3|x_2, x_1)p(x_4|x_1, x_2, x_3) ... p(x_V |x_{1:V −1})$(10.1)

其中的V是变量数目,MATLAB风格的记号$1:V$表示集合{1,2,...,V},为了简洁,上面狮子中去掉了对固定参数$\theta$的条件.这个表达式的问题在于随着T变大,要表示条件分布$p(x_t|x_{1:t-1})$就越来越复杂了.

例如,设所有变量都有K个状态(states).然后可以将$p(x_1)$表示为$O(K)$个数字的表格,表示了一个离散分布(实际上只有K-1个自由参数,因为总和为一的约束条件,不过这里为了写起来简单就写成O(K)了).然后还可以将$p(x_2|x_1)$写成一个$O(K^2)$个数值的表格,写出$p(x_2=j|x_1=i)=T_{ij}$;T叫做随机矩阵(stochastic matrix)对$0\le T+{ij}\le 1$的所有条目以及所有列i满足约束条件$\sum_jT_{ij}=1$.与此类似,还可以将$p(x_3|x_1, x_2)$表示成一个有$O(K^3)$个数值的三维表格.这些表格就叫做条件概率表格(conditional probability tables,缩写为CPT).然后在我们这个模型里面就有$O(K^V)$个参数.要学习得到这么多参数需要的数据量就要多得可怕了.

要解决这个问题可以将每个条件概率表(CPT)替换成条件概率分布(conditional probability distribution,缩写为CPD),比如多想逻辑回归,也就是$p(x_t=k|x_{1:t-1})=S(W_tx_{1:t-1})_k$.这样全部参数就只有$O(K^2V^2)$个了,就得到了一个紧凑密度模型(compact density model)了(Neal 1992; Frey 1998).如果我们要评估一个全面观测向量$x_{1:t-1}$的概率,这已经足够了.比如可以使用这个模型定义一个类条件密度(class-conditional density)$p(x|y)=c$,然后建立一个生成分类器 (generative classifier,Bengio and Bengio 2000).不过这个模型不能用于其他预测任务,因为这个模型里面每个变量都依赖之前观测的全部变量.所以其他问题要另寻办法.

### 10.1.2 条件独立性(Conditional independence)

高效率表征一个大规模联合分布的关键就是对条件独立性(Conditional independence,缩写为CI)进行假设.回忆本书2.2.4当中,在给定Z的情况下,X和Y的条件独立记作$X \bot Y|Z$,当且仅当条件联合分布可以写成条件边缘分布乘积的时候才成立,如下所示:

$X \bot Y|Z \iff p(X,Y|Z)=p(X|Z)p(Y|Z)$(10.2)

这有啥用呢?假如有$x_{t+1}\bot x_{1:t-1}|x_t$,也就是说在给定当前值的条件下,未来值和过去值独立.这就叫(一阶(first order))马尔科夫假设(Markov assumption).利用这个假设,加上链式规则,就可以写出联合分布形式如下所示:

$p(x_{1:V})=p(x_1)\prod^V_{t=1}p(x_t|x_{t-1})$(10.3)

这就叫做一个(一阶(first order))马尔科夫链(Markov chain).可以通过在状态上的初始分布(initial distribution)$p(x_1=i)$来表示,另外加上一个状态转换矩阵(state transition matrix)$p(x_t=j|x_{t-1}=i)$.更多细节参考本书17.2.

### 10.1.3 图模型

虽然一阶马尔科夫假设对于定义一维序列分布很有用,但对于二维图像.或者三维视频,或者更通用的任意维度的变量集(比如生物通路上的基因归属等等),要怎么定义呢?这时候就需要图模型了.

图模型(Graphical models,缩写为GM)是通过设置条件独立性假设(CI assumption)来表示一个联合分布(joint distribution).具体来说就是图上的节点表示随机变量,而(缺乏的)边缘表示条件独立性假设(CI assumption)(对这类模型的更好命名应该是独立性图(independence diagrams),不过图模型这个叫法已经根深蒂固了.)有几种不同类型的图模型,取决于图是有向(directed)/无向(undirected)/或者两者结合.在本章只说有向图(directed graphs).到第19章再说无向图(undirected graphs).

此处参考原书图10.1

### 10.1.4 图模型术语(Graph terminology)


在继续讨论之前,先要定义一些基本术语概念,大部分都很好理解.

一个图(graph)$G=(V,\mathcal{E})$包括了一系列节点(node)或者顶点(vertices),$V=\{1,...,V\}$,还有一系列的边(deges)$\mathcal{E} =\{ (s,t):s,t\in V \}$.可以使用邻近矩阵(adjacency matrix)来表示这个图,其中用$G(s,t)$来表示$(s,t)\in \mathcal{E}$,也就是$s\rightarrow t$是凸中的一个遍.如果当且仅当$G(t,s)=1$的时候$G(s,t)=1$,就说这个图是无向的(undirected),否则就是有向的(directed).一般假设$G(s,s)=0$,意思是没有自我闭环(self loops).

下面是其他一些要常用到的术语:


* 父节点(Parent)对一个有向图,一个节点的父节点就是所有节点所在的集合:$pa(s) \overset{\triangle}{=} {t : G(t, s) = 1}$.
* 子节点(Child)对一个有向图,一个节点的子节点就是从这个节点辐射出去的所有节点的集合:$ch(s) \overset{\triangle}{=} {t : G(s,t) = 1}$.
* 族(Family)对一个有向图,一个节点的族是该节点以及所有其父节点:$fam(s)= \{s\}\cup pa(s)$.
* 根(root)对一个有向图,根是无父节点的节点.
* 叶(leaf)对一个有向图,叶就是无子节点的节点.
* 祖先(Ancestors)对一个有向图,祖先包括一个节点的父节点/祖父节点等等.也就是说t的祖先是所有通过父子关系向下连接到t的节点:$anc(t)\overset{\triangle}{=} \{s:s\rightsquigarrow t\}$.
* 后代(Descendants)对一个有向图,后代包括一个节点的子节点/次级子节点等等.也就是s的后代就是可以通过父子关系上溯到s的所有节点集合:$desc(t)\overset{\triangle}{=} \{t:s\rightsquigarrow t\}$.
* 邻节点(Neighbors),对于任意图来说,所有直接连接的节点都叫做邻节点:$nbr(s) \overset{\triangle}{=}  \{t : G(s, t) = 1 \vee G(t, s) = 1\}$.对于无向图(undirected graph)来说,可以用$s \sim t$来表示s和t是邻节点(这样$(s,t)\in \mathcal{E}$就是图的边(edge)了).
* 度数(Degree)一个节点的度数是指该节点的邻节点个数.对有向图,又分为入度数(in-degree)和出度数(out-degree),指代的分别是某个节点的父节点和子节点的个数.
* 闭环(cycle/loop)顾名思义,只要沿着一系列节点能回到初始的位置,顺序为$s_1 \rightarrow s_2 ... \rightarrow s_n \rightarrow s_1, n \ge 2$,就称之为一个闭环.如果图是有向的,闭环也是有向的.图10.1(a)中没有有向的闭环,倒是有个无向闭环$1\rightarrow 2\rightarrow 4 \rightarrow 3 \rightarrow 1$.
* 有向无环图(directed acyclic graph,缩写为DAG)顾名思义,就是有向但没有有向闭环的,比如图10.1(a)就是一例.
* 拓扑排序(Topological ordering)对一个有向无环图(DAG),拓扑排序(topological ordering)或者也叫全排序(total ordering)是所有父节点比子节点数目少的节点的计数.比如在图10.1(a)中,就可以使用(1, 2, 3, 4, 5)或者(1, 3, 2, 5, 4)等.
* 路径(Path/trail)对于$s\rightsquigarrow t$来说路径就是一系列从s到t的有向边(directed edges).
* 树(Tree)无向树(undirected tree)就是没有闭环的无向图.有向树(directed tree)是没有有向闭环的有向无环图(DAG).如果一个节点可以有多个父节点,就称为超树(polytree),如果不能有多个父节点,就成为规范有向树(moral directed tree).
* 森林(Forest)就是树的集合.
* 子图(Subgraph)(包括节点的)子图$G_A$是使用A中的节点和对应的边(edges)创建的$G_A=(\mathcal{V}_A,\mathcal{E}_A)$.
* 团(clique)对一个无向图,团是一系列互为邻节点的节点的集合.在不损失团性质的情况下能达到的最大规模的团就叫做最大团(maximal clique).比如图10.1(b)
当中的{1,2}就是一个团,但不是最大团,因为把3加进去依然保持了团性质(clique property).图10.1(b)中的最大团:{1, 2, 3}, {2, 3, 4}, {3, 5}.

### 10.1.5 有向图模型

有向图模型(directed graphical model,缩写为DGM)是指整个图都是有向无环图(directed acyclic graph,缩写为DAG)的图模型.更广为人知的名字叫贝叶斯网络(Bayesian networks).不过实际上这个名字并不是说这个模型和贝叶斯方法有啥本质上的联系:只是定义概率分布的一种方法而已.这些模型也叫作信念网络(belief networks).这里的信念这个词(belief)指的是主观的概率.关于表征有向图模型(DGM)的概率分布的种类并没有什么本质上的主管判断.这些模型有时候也叫作因果网络(causal networks),因为有时候可以将有向箭头解释成因果关系.不过有向图模型本质上并没有因果关系(关于因果关系有向图模型的讨论参考本书26.6.1.)名字这么多这么乱,咱们就选择最中性的称呼,就叫它有向图模型(DGM).

有向无环图(DAG)的关键性之就是节点可以按照父节点在子节点之前来排序.这也叫做拓扑排序(topological ordering),在任何有向无环图中都可以建立这种排序.给定一个这样的排序,就定义了一个有序马尔科夫性质(ordered Markov property),也就是假设一个节点只取决于其直接父节点,而不受更早先辈节点的影响.也就是:

$x_s \bot  x_{pred(s) / pa(s)}| x_{pa(s)}$(10.4)

(注:上面公式中应该是反斜杠 "\",但是我不知在LaTex里面怎么打出来.)

上式中的$pa(s)$表示的是节点s的父节点,而$pred(s)$表示在排序中s节点的先辈节点.这是对一阶马尔科夫性质(first-order Markov property)的自然扩展,目的是构成一个链条来泛华有向无环图(DAG).

此处参考原书图10.2

例如,在图10.1(a)中编码了下面的联合分布:

$$
\begin{aligned}
p(x_{1:5}) &= p(x_1)p(x_2|x_1)p(x_3|x_1,\cancel{x_2})p(x_4|\cancel{x_1}, x_2, x_3)p(x_5|\cancel{x_1},\cancel{x_2},x_3,\cancel{x_4}) &\text{(10.5)}\\
&=  p(x_1)p(x_2|x_1)p(x_3|x_1)p(x_4|x_2, x_3)p(x_5|x_3)   &\text{(10.6)}
\end{aligned}
$$

其中每个$p(x_t|x_{pa(t)})$都是一个条件概率分布(CPD).将分布写成$p(x|G)$是要强调只有当有向无环图(DAG)G当中编码的条件独立性假设(CI assumption)是正确的时候登时才能成立.不过通常为了简单起见都会把这个条件忽略掉.每个节点都有$O(F)$个父节点以及K个状态,这个模型中参数总数就是$O(VK^F)$,这就比不做独立性假设的模型所需要的$O(K^V)$个要少多了.

## 10.2 样例

这一节展示一些可以用有向图模型(DGM)来表示的常见概率模型.

### 10.2.1 朴素贝叶斯分类器(Naive Bayes classifiers)

朴素贝叶斯分类器是在本书3.5就讲到了.这个模型中假设了给定类标签(calss label)条件下特征条件独立.此假设如图10.2(a)所示.这样就可以将联合分布写成下面的形式:
$p(y,x)=p(y)\prod^D_{j=1}p(x-j|y)$(10.8)

朴素贝叶斯加设很朴素,因为假设了所有特征都是条件独立的.使用图模型就是捕获变量之间相关性的一种方法.如果模型是属性的,就称此方法为树增强朴素贝叶斯分类器(tree-augmented naive Bayes classifiers,缩写为TAN,Friedman et al. 1997).如图10.2(b)所示.使用树形结构而不是一般的图模型的原因有两方面.首先是树形结构使用朱刘算法(Chow-Liu algorithm,1965年有朱永津和刘振宏提出)来优化,具体如本书26.3所示.另外就是树结构模型的缺失特征好处理,具体会在本书20.2有解释.

此处参考原书图10.3


此处参考原书图10.4

### 10.2.2 马尔科夫和隐形马尔科夫模型

图10.3(a)所示为将一阶马尔科夫连表示成了一个有向无环图(DAG).其中的假设是最邻近的上一个特征$x_{t-1}$包含了之前所有历史特征$x_{1:t-2}$的所有需要我们知道的内容,这个假设有点太强了.为了减弱一点这个假设,可以再添加一下从$x_{t-2}$到$x_t$的依赖;这样就成了一个二阶马尔科夫链(second order Markov chain),如图10.3(b)所示.对应的联合分布也就是:

$p(x_{1:T}) = p(x_1, x_2)p(x_3|x_1, x_2)p(x_4|x_2, x_3) . . . = p(x_1, x_2)\prod^T_{t=3}p(x_t|x_{t−1}, x_{t−2})$ (10.9)

用类似方法还可以建立更高阶的马尔科夫链.关于马尔科夫模型的更多细节参考本书17.2.
然而很不幸,就算使用二阶马尔科夫假设,面对观测有长程相关性的情况也可能会不适用.也不能持续无限提高阶数,因为这样参数规模就太大了.另外一种方法就是假设存在潜在的隐藏过程,而这个过程可以用一个一阶马尔科夫链来建模,但数据是这个过程中的有噪音观测.这样就得到了隐形马尔科夫模型(hidden Markov model,缩写为HMM),如图10.4所示.其中的$z_t$是在"次数(time)"t上的隐含变量(hidden variable),而$x_t$是观测变量.(这个模型可以用于任意的有序数据,比如基因序列或者语言文字等等,所以t指代的可以使位置而不一定只是观测时间.)条件概率分布(CPD)$p(z_t|z_{t-1})$是转换模型(transition model),而条件概率分布(CPD)$p(x_t|z_t)$是观测模型(observation model).

此处参考原书表10.1

这些隐含变量往往是有用变量,比如某人讲话过程中某个词汇的作用.观测变量就是我们观测到的值,比如声音波形.我们需要做的就是在给定数据的情况下估计隐含状态,也就是计算$p(z_t|x_{1:t},\theta)$.这就叫做状态估计(state estimation).也是一种概率推导的形式.关于隐形马尔科夫模型的更多细节会在本书17章讲解.

### 10.2.3 医学诊断

接下来考虑这样一个场景,医院重症监护室(intensive care unit,缩写为ICU)中衡量各种人体参数变量,比如心率/呼吸频率/血压等等,对这些变量之间的关系进行建模.图10.5(a)所示的报警网络(alarm network)就是表征这些变量的相关性或者无关性的一种方法(Beinlich et al. 1989).这个模型有37个变量,504个参数.

这个模型是人工构建的,通过了一个叫做知识工程(knowledge engineering)的过程,这个系统也就叫做概率专家系统(probabilistic expert system).在本书10.4会讲到在假设图结构未知的情况下,如何从数据中学习得到有向图模型的参数.然后在本书26章会讲到如何学习图结构本身.

还有另外一种医疗诊断用的网络模型,叫做快速医疗指南(quick medical reference,缩写为QMR)网络(Shwe et al. 1991),如图10.5(b)所示.这个模型通常用于对传染病的建模.这种QMR模型往往是二分图结构(bipartite graph structure),疾病种类或者致病因素在顶层,而症状或者发病表现在底层.所有的节点都是二值化的(binary).可以将这个分布写成虾米的形式:

$p(v,h)=\prod_s p(h_s)\prod_t p(v_t|h_{pa(t)})$(10.10)

上式中的$h_s$就是隐藏节点(hidden nodes)(疾病),而$v_t$表示的是可见节点(visible nodes)(症状).

根节点(root nodes)的条件概率分布(CPD)就是伯努利分布,表示了对应疾病的先验概率.对叶节点(leaves)(症状)使用条件概率表格(CPT)来表示条件概率分布(CPD)会需要太多参数,因为很多叶节点(leaf node)都有特别多的父节点数.一个很自然的替代方法就是使用逻辑回归来对条件概率分布建模,$p(v_t=1|h_{pa(t)})=sigm(w^T_th_{pa(t)})$.(条件概率分布为逻辑回归分布的有向图模型也叫作S形信念网络(sigmoid belief net,Neal 1992).)上面这个模型的参数是认为创建的,还有另外一个可用条件概率分布(CPD)叫做"噪音或模型"(noisy-OR model,这里的**或**是指**或门,or gate**).

此处参考原书图10.5
此处参考原书表10.2

噪音或模型(noisy-OR model)假设如果父节点为开(on),子节点通常也为开(on)(因为这是一个或门or gate),但有时候从父节点到子节点的连接也可能会失败,这种情况随机独立发生.这种情况下畸变父节点为开(on),子节点也可能为关(off).要对这类情况进行更确切地建模,可以设$s\rightarrow t$连接失败的概率为$\theta_{st}=1-q_{st}$,所以$q_{st}=1-\theta_{st}= p(v_t=1| h_s=1,h_{-s}=0)$为s可单独激活t的概率(也就是因果律,causal power").而子节点为关(off)则只会是所有从父节点的链接都随机独立失败.也就是:

$p(v_t=0|h) =\prod_{s\in pa(t)} \theta_{st}^{I(h_s=1)}$(10.11)

很明显$p(v_t=1|h)=1-p(v_t=0|h)$.

如果我们观测到$v_t=1$,而素有父节点都为关(off)那么这就违背(contradicts)了这个模型.这样的一个数据案例会在这个模型下得到零概率.这就有问题了,因为很可能某个人并没有患上任何特定疾病,但也可能表现出某个症状.要处理这种情况,就要添加一个傻傻的遗漏节点(dummy leak node)$h_0$,这个节点总处于开状态(on);这就代表着所有其他原因.参数$q_{0t}$表示的是背景泄露本身导致这个效果的概率.修改过的条件概率分布(CPD)就成了$p(v_t=0|h)=\theta_{0t}\prod_{s\in pa(t)}\theta^{h_s}_{st}$.数值样本参考表10.1.

如果定义$w_{st}\overset{\triangle}{=} \log(\theta_{st})$,就可以将条件概率分布(CPD)重写成:

$p(v_t=1|h)=1- \exp(w_{ot}+\sum_s h_sw_{st})$(10.12)

可见这和逻辑回归模型就很相似了.

带有噪声或门的二分模型(Bipartite models with noisy-OR)的条件概率分布(CPD)也简写作BN2O模型.通常借助相关领域的专业经验来人为设置$\theta_{st}$参数都比较简单.不过也可以从数据中学习来进行设置参数(Neal 1992; Meek and Heckerman 1997).有噪音或条件概率分布(Noisy-OR CPDs)在对人类的因果学习建模的时候也很有用(Griffiths and Tenenbaum 2005),另外也适用于通用二值化分类背景(Yuille and Zheng 2009).


### 10.2.4 基因链接分析(Genetic linkage analysis)*


对有向图模型的应用还有另外一个很重要也很悠久的领域,就是基因链接分析(genetic linkage analysis).先从谱系系树(pedigree graph)开始,这是一种表示了父子节点关系的有向无环图模型(DAG)如图10.6(a)所示.然后将其转换成有向图模型(DGM),接下来就会讲解这个过程.最后在得到的模型中进行概率推导.

此处参考原书图10.6

更详细说,对于每个人或者动物和沿着基因上的位置j都建立三个节点:观测标记(observed marker)$X_{ij}$(可以使血型啊或者一个能衡量的DNA片段),以及另外的两个隐藏的等位基因(hidden alleles)$G_{ij}^m$和$G_{ij}^p$,一个是来自i的母本(母本等位基因,maternal allele),另一个来自i的父本(父本等位基因,paternal allele)结合在一起就形成了有序基因对$G_{ij}=(G_{ij}^m,G_{ij}^p)$,这就是i在位置j上的隐藏基因型(hidden genotype).

很明显必须添加$(G_{ij}^m \rightarrow X_{ij}$和$G_{ij}^p\rightarrow X_{ij}$来表示基因对性状(性状就是基因型的表现效应)的控制.条件概率分布(CPD)$p(X_{ij}|G_{ij}^m ,G_{ij}^p)$就叫做外显率模型(penetrance model).比如$X_{ij}\in \{A, B, O, AB\}$表示了第i个人的观测学习,而$G _{ij}^m , G_{ij}^p \in \{A, B, O\}$表示的是他们的基因型(genotype).就可以使用表10.2所示的确定性条件概率分布(deterministic CPD)来表示其外显率模型.比如基因A的优先级高于O,所以一个人基因型如果为AO或者OA,他的血型就是A型.

另外还要在$G_{ij}$上加上i的父母,这反映了一个人从父母得到遗传物质的孟德尔遗传定律(Mendelian inheritance).具体来说就是设$m_i=k$为i的母本.然后$G_{ij}^m$就可以等于$G_{kj}^m$或者$G_{kj}^p$,也就是i的母本等位基因是其母本两个基因当中的一个的复制品.设$Z_{ij}^m$为隐藏变量确定了具体所选择的复制对象.然后可以用下面的条件概率分布对其进行建模,这个模型就是遗传模型(inheritance model):
$p(G_{ij}^m|G_{kj}^m,G_{kj}^p,Z_{ij}^m) =\begin{cases} I(G_{ij}^m=G_{kj}^m) \text{ if } Z_{ij}^m=m\\I(G_{ij}^m=G_{kj}^p) \text{ if } Z_{ij}^m=p \end{cases}$(10.13)

类似的方法还可以定义$p(G_{ij}^p|G_{kj}^m,G_{kj}^p,Z_{ij}^p)$,其中的$k=p_i$表示的是i的父本.$Z_{ij}$的值可以用来确定基因类型的相(phase).而$G_{ij}^p,G_{ij}^m,Z_{ij}^p,Z_{ij}^m $构成了第i个人在第j个位置上的单倍型(haplotype).

然后要指定根节点的先验$p(G_{ij}^m$和$p(G_{ij}^p)$.这就叫祖先模型(founder model),表示了在人口中不同基因型的总体比例.通常假设这些祖先基因在不同位置上相互独立.

最后要对控制遗传过程的转换变量指定先验.这些变量在空间上相关(spatially correlated),因为基因组上邻近的位置通常是一同遗传的,重新结合的情况很罕见.可以在Z上引入一个二阶马尔科夫链来对此进行建模,其中在位置j上的转换状态的概率由$\theta_j=\frac{1}{2}(1-e^{-2d_j})$给出,其中的$d_j$是位置j和j+1之间的距离.这个模型就叫做重新组合模型(recombination model).

这样得到的涂抹些如图10.6(b)所示:一个同血缘的有向无环图(replicated pedigree DAG),通过转换Z变量增强(augmented),使用了马尔科夫链.(有个相关的模型叫做进化阴性马尔科夫模型(phylogenetic HMM,Siepel and Haussler 2003),模拟的是进化过程中的演化.)

为了简单,这里举例只看一个基因位置,也就是对应血型的片段.为了简单就去掉j索引了.假如观测到了$x_i=A$.那么就有三种可能的基因组$G_i$:(A, A), (A, O) ,(O, A).
这里会有多解性,因为基因组到基因表型的映射是多到一的.要将这个映射关系逆转,就要面对逆转问题(inverse problem).还好可以使用亲戚的学醒来降低甚至消除这些多解性.信息就会通过学院有向无环树(pedigree DAG)从其他的$x_i$留到他们的$G_i$中,然后传递到第i个人的$G_i$上.因此可以结合局部证据(local evidence)$p(x_i|G_i)$和先验$p(G_i|X_{-i})$,以其他数据为条件,然后得到一个更低熵(less entropic)的局部后验$p(G_i|x)\propto p(x_i|G_i)p(G_i|x_{-i})$.
在实际应用中,这个模型一般用来根据给定疾病的致病基因来检测是否有疾病,也就是基因关联检测任务(genetic linkage analysis task).这种方法的工作流程如下.首先假设模型中所有参数,包括标记位置之间的距离,都是已知的.而唯一未知的是致病基因的位置.如果有L个标记位置,就构建L+1个模型:在模型l中,假定致病基因在标记l后出现,$0< l < L+1$.然后可以通过切换参数$\hat \theta_l$来估计马尔科夫模型,然后是致病基因和其最邻近位置之间的距离$d_l$.通过似然函数$p(D|\hat\theta_l)$来衡量模型质量.然后选出来最高似然率的模型(在均匀先验(uniform prior)下等价于最大后验分布模型(MAP model).

不过要记住,计算似然率的时候需要边缘化掉所有的隐藏的Z和G这些变量.关于这个模型的具体信息细节参考(Fishelson and Geiger 2002);这些是基于变量估计算法的模型,我们会在本书20.3讲到.不过很不幸,由于一些会在本书20.5讲到的原因,实际上如果个体规模或者基因位置太多的话,具体的模型在计算上可能会很困难.关于计算似然率的近似方法参考(Albers et al. 2006);这种方法是变分推导(variational inference)的一种形式,具体会在本书22.4.1讲到.

### 10.2.5 有向高斯图模型(Directed Gaussian graphical models)*

考虑一个有向图模型(DGM),其中全部变量都是实数值的,而所有条件概率分布(CPD)都有如下的形式:

$p(x_t|x_{pa(t})=N(x_t|\mu_t+w_t^Tx_{pa(t)},\sigma_t^2)$(10.14)

这样的条件概率分布就叫做线性高斯条件概率分布(linear Gaussian CPD).将所有这些条件概率分布(CPD)乘到一起,就得到了一个大规模的连和高斯分布,形式为$p(x)=N(x|\mu,\Sigma)$.这就叫一个有向高斯图模型(缩写为 directed GGM),或者也叫做高斯贝叶斯网络(Gaussian Bayes net).

接下来解释一下如何从条件概率参数中推出$\mu$和$\Sigma$,这部分的内容参考了(Shachter and Kenley 1989, App. B).为了简单,将条件概率分布写成下面的形式:
$x_t =\mu_t +\sum_{s\in pa(t)} w_{ts}(x_s-\mu_s)+\sigma_tz_t$(10.15)

其中$z_t\sim N(0,1)$,$\sigma_t$为给定亲本下$x_t$的条件标准偏差(conditional standard deviation),$w_s$是$s\rightarrow t$的强度,而$\mu_t$是局部均值(local mean).

很明显局部均值的连接(concatenation)就是全局均值了,即$\mu=(\mu_1,...,\mu_D)$.然后再推导全局协方差(global covariance)$\Sigma$.设$s\overset{\triangle}{=} diag(\sigma)$是一个对角矩阵,包含了标准偏差.然后可以将等式10.15写成矩阵向量乘积的形式如下所示:
$(x − \mu) = W(x − \mu) + Sz$(10.16)

然后设e为噪音项目的向量:
$e \overset{\triangle}{=} Sz$(10.17)

重新整理就得到了:
$e=(I-W)(x-\mu)$(10.18)

因为W是下三角矩阵(因为如果在拓扑排序(topological ordering)中$t>s$,则$w_{ts}=0$),所以则有$I-W$是一个对角线为1的下三角矩阵.因此:
$\begin{pmatrix} e_1\\e_2\\.\\.\\.\\e_d \end {pmatrix} =\begin{pmatrix} 1&&&&\\-w_{21}&1&&&\\-w_{32}&-w_{31}&1&&\\ ...\\ -w_{d1}&-w_{d2}&...&-w_{d,.d-1}& 1\end {pmatrix} \begin{pmatrix} x_1-\mu_1\\ x_2-\mu_2\\ .\\.\\.\\ x_d-\mu_d \end {pmatrix} $(10.19)



由于 I-W 总是可逆的(invertible),就可以写出:

$x-\mu = (I-W)^{-1}e \overset{\triangle}{=} Ue =USz$(10.20)

其中定义$U=(I-W)^{-1}$.另外回归权重(regression weights)对应着对$\Sigma$的柯列斯基分解(Cholesky decomposition),如下所示:

$$
\begin{aligned}
\Sigma &= cov [x] = cov [x − \mu ]   &\text{(10.21)}\\
&= cov [USz] = US cov [z] SU^T = US^2 U^T &\text{(10.22)}\\
\end{aligned}
$$


