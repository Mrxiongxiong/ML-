# ML- data processing
《数据清洗》
按照我做项目的经验，来了项目，首先是分析项目的目的和需求，了解这个项目属于什么问题，要达到什么效果。然后提取数据，做基本的数据清洗。第三步是特征工程，这个属于脏活累活，需要耗费很大的精力，如果特征工程做的好，那么，后面选择什么算法其实差异不大，反之，不管选择什么算法，效果都不会有突破性的提高。第四步，是跑算法，通常情况下，我会把所有能跑的算法先跑一遍，看看效果，分析一下precesion/recall和f1-score，看看有没有什么异常（譬如有好几个算法precision特别好，但是recall特别低，这就要从数据中找原因，或者从算法中看是不是因为算法不适合这个数据），如果没有异常，那么就进行下一步，选择一两个跑的结果最好的算法进行调优。调优的方法很多，调整参数的话可以用网格搜索、随机搜索等，调整性能的话，可以根据具体的数据和场景进行具体分析。调优后再跑一边算法，看结果有没有提高，如果没有，找原因，数据 or 算法？是数据质量不好，还是特征问题还是算法问题。一个一个排查，找解决方法。特征问题就回到第三步再进行特征工程，数据质量问题就回到第一步看数据清洗有没有遗漏，异常值是否影响了算法的结果，算法问题就回到第四步，看算法流程中哪一步出了问题。如果实在不行，可以搜一下相关的论文，看看论文中有没有解决方法。这样反复来几遍，就可以出结果了，写技术文档和分析报告，再向业务人员或产品讲解我们做的东西，然后他们再提建议/该需求，不断循环，最后代码上线，改bug，直到结项。

　　直观来看，可以用一个流程图来表示：



 

　　今天讲数据清洗，为什么要进行数据清洗呢？我们在书上看到的数据，譬如常见的iris数据集，房价数据，电影评分数据集等等，数据质量都很高，没有缺失值，没有异常点，也没有噪音，而在真实数据中，我们拿到的数据可能包含了大量的缺失值，可能包含大量的噪音，也可能因为人工录入错误导致有异常点存在，对我们挖据出有效信息造成了一定的困扰，所以我们需要通过一些方法，尽量提高数据的质量。数据清洗一般包括以下几个步骤：

一.分析数据
二.缺失值处理
三.异常值处理
四.去重处理
五.噪音数据处理
六.一些实用的数据处理小工具
 
一.分析数据
　　在实际项目中，当我们确定需求后就会去找相应的数据，拿到数据后，首先要对数据进行描述性统计分析，查看哪些数据是不合理的，也可以知道数据的基本情况。如果是销售额数据可以通过分析不同商品的销售总额、人均消费额、人均消费次数等，同一商品的不同时间的消费额、消费频次等等，了解数据的基本情况。此外可以通过作图的方式了解数据的质量，有无异常点，有无噪音等。举个例子（这里数据较少，就直接用R作图了）：
复制代码
1 #一组年薪超过10万元的经理收入
2 pay=c(11,19,14,22,14,28,13,81,12,43,11,16,31,16,23.42,22,26,17,22,13,27,180,16,43,82,14,11,51,76,28,66,29,14,14,65,37,16,37,35,39,27,14,17,13,38,28,40,85,32,25,26,16,12,54,40,18,27,16,14,33,29,77,50,19,34)
3 par(mfrow=c(2,2))#将绘图窗口改成2*2，可同时显示四幅图
4 hist(pay)#绘制直方图
5 dotchart(pay)#绘制点图
6 barplot(pay,horizontal=T)#绘制箱型图
7 qqnorm(pay);qqline(pay)#绘制Q-Q图
复制代码
 


 
　　从上面四幅图可以很清楚的看出，180是异常值，即第23个数据需要清理。
 
　　python中也包含了大量的统计命令，其中主要的统计特征函数如下图所示：

 
 
二.缺失值处理
　　缺失值在实际数据中是不可避免的问题，有的人看到有缺失的数据就直接删除了，有的人直接赋予0值或者某一个特殊的值，那么到底该怎么处理呢？对于不同的数据场景应该采取不同的策略，首先应该判断缺失值的分布情况：
 
1 import scipy as sp
2 data = sp.genfromtxt("web_traffic.tsv",delimiter = "\t")
 

数据情况如下：

复制代码
 1 >>>data
 2 array([[  1.00000000e+00,   2.27200000e+03],
 3        [  2.00000000e+00,              nan],
 4        [  3.00000000e+00,   1.38600000e+03],
 5        ...,
 6        [  7.41000000e+02,   5.39200000e+03],
 7        [  7.42000000e+02,   5.90600000e+03],
 8        [  7.43000000e+02,   4.88100000e+03]])
 9 
10 >>> print data[:10]
11 [[  1.00000000e+00   2.27200000e+03]
12  [  2.00000000e+00              nan]
13  [  3.00000000e+00   1.38600000e+03]
14  [  4.00000000e+00   1.36500000e+03]
15  [  5.00000000e+00   1.48800000e+03]
16  [  6.00000000e+00   1.33700000e+03]
17  [  7.00000000e+00   1.88300000e+03]
18  [  8.00000000e+00   2.28300000e+03]
19  [  9.00000000e+00   1.33500000e+03]
20  [  1.00000000e+01   1.02500000e+03]]
21 
22 >>> data.shape
23 (743, 2)
复制代码
 

可以看到，第2列已经出现了缺失值，现在我们来看一下缺失值的数量：

1 >>> x = data[:,0]
2 >>> y = data[:,1]
3 >>> sp.sum(sp.isnan(y))
4 8
　　在743个数据里只有8个数据缺失，所以删除它们对于整体数据情况影响不大。当然，这是缺失值少的情况下，在缺失值值比较多，而这个维度的信息还很重要的时候（因为缺失值如果占了95%以上，可以直接去掉这个维度的数据了），直接删除会对后面的算法跑的结果造成不好的影响。我们常用的方法有以下几种：

1.直接删除----适合缺失值数量较小，并且是随机出现的，删除它们对整体数据影响不大的情况

2.使用一个全局常量填充---譬如将缺失值用“Unknown”等填充，但是效果不一定好，因为算法可能会把它识别为一个新的类别，一般很少用

3.使用均值或中位数代替----优点：不会减少样本信息，处理简单。缺点：当缺失数据不是随机数据时会产生偏差.对于正常分布的数据可以使用均值代替，如果数据是倾斜的，使用中位数可能更好。

4.插补法

　　1）随机插补法----从总体中随机抽取某个样本代替缺失样本
　　2）多重插补法----通过变量之间的关系对缺失数据进行预测，利用蒙特卡洛方法生成多个完整的数据集，在对这些数据集进行分析，最后对分析结果进行汇总处理
　　3）热平台插补----指在非缺失数据集中找到一个与缺失值所在样本相似的样本（匹配样本），利用其中的观测值对缺失值进行插补。
　　　　优点：简单易行，准去率较高
　　　　缺点：变量数量较多时，通常很难找到与需要插补样本完全相同的样本。但我们可以按照某些变量将数据分层，在层中对缺失值实用均值插补
　　4)拉格朗日差值法和牛顿插值法（简单高效，数值分析里的内容，数学公式以后再补 = =）
5.建模法
可以用回归、使用贝叶斯形式化方法的基于推理的工具或决策树归纳确定。例如，利用数据集中其他数据的属性，可以构造一棵判定树，来预测缺失值的值。
 

　　以上方法各有优缺点，具体情况要根据实际数据分分布情况、倾斜程度、缺失值所占比例等等来选择方法。一般而言，建模法是比较常用的方法，它根据已有的值来预测缺失值，准确率更高。
 
三.异常值处理
　　异常值我们通常也称为“离群点”。在讲分析数据时，我们举了个例子说明如何发现离群点，除了画图（画图其实并不常用，因为数据量多时不好画图，而且慢），还有很多其他方法:
 
1.简单的统计分析
　　拿到数据后可以对数据进行一个简单的描述性统计分析，譬如最大最小值可以用来判断这个变量的取值是否超过了合理的范围，如客户的年龄为-20岁或200岁，显然是不合常理的，为异常值。
在python中可以直接用pandas的describe()：
复制代码
 1 >>> import pandas as pd
 2 >>> data = pd.read_table("web_traffic.tsv",header = None)
 3 >>> data.describe()
 4                 0            1
 5 count  743.000000   735.000000
 6 mean   372.000000  1962.165986
 7 std    214.629914   860.720997
 8 min      1.000000   472.000000
 9 25%    186.500000  1391.000000
10 50%    372.000000  1764.000000
11 75%    557.500000  2217.500000
12 max    743.000000  5906.000000
复制代码
 
2.3∂原则
　　如果数据服从正态分布，在3∂原则下，异常值为一组测定值中与平均值的偏差超过3倍标准差的值。如果数据服从正态分布，距离平均值3∂之外的值出现的概率为P(|x-u| > 3∂) <= 0.003，属于极个别的小概率事件。如果数据不服从正态分布，也可以用远离平均值的多少倍标准差来描述。
 
3.箱型图分析
　　箱型图提供了识别异常值的一个标准：如果一个值小于QL01.5IQR或大于OU-1.5IQR的值，则被称为异常值。QL为下四分位数，表示全部观察值中有四分之一的数据取值比它小；QU为上四分位数，表示全部观察值中有四分之一的数据取值比它大；IQR为四分位数间距，是上四分位数QU与下四分位数QL的差值，包含了全部观察值的一半。箱型图判断异常值的方法以四分位数和四分位距为基础，四分位数具有鲁棒性：25%的数据可以变得任意远并且不会干扰四分位数，所以异常值不能对这个标准施加影响。因此箱型图识别异常值比较客观，在识别异常值时有一定的优越性。
 
4.基于模型检测
　　首先建立一个数据模型，异常是那些同模型不能完美拟合的对象；如果模型是簇的集合，则异常是不显著属于任何簇的对象；在使用回归模型时，异常是相对远离预测值的对象
优缺点：1.有坚实的统计学理论基础，当存在充分的数据和所用的检验类型的知识时，这些检验可能非常有效；2.对于多元数据，可用的选择少一些，并且对于高维数据，这些检测可能性很差。
 
5.基于距离
　　通常可以在对象之间定义邻近性度量，异常对象是那些远离其他对象的对象
优缺点：1.简单；2.缺点：基于邻近度的方法需要O(m2)时间，大数据集不适用；3.该方法对参数的选择也是敏感的；4.不能处理具有不同密度区域的数据集，因为它使用全局阈值，不能考虑这种密度的变化。
 
6.基于密度
　　当一个点的局部密度显著低于它的大部分近邻时才将其分类为离群点。适合非均匀分布的数据。
优缺点：1.给出了对象是离群点的定量度量，并且即使数据具有不同的区域也能够很好的处理；2.与基于距离的方法一样，这些方法必然具有O(m2)的时间复杂度。对于低维数据使用特定的数据结构可以达到O(mlogm)；3.参数选择困难。虽然算法通过观察不同的k值，取得最大离群点得分来处理该问题，但是，仍然需要选择这些值的上下界。
 
7.基于聚类：
　　基于聚类的离群点：一个对象是基于聚类的离群点，如果该对象不强属于任何簇。离群点对初始聚类的影响：如果通过聚类检测离群点，则由于离群点影响聚类，存在一个问题：结构是否有效。为了处理该问题，可以使用如下方法：对象聚类，删除离群点，对象再次聚类（这个不能保证产生最优结果）。
优缺点：1.基于线性和接近线性复杂度（k均值）的聚类技术来发现离群点可能是高度有效的；2.簇的定义通常是离群点的补，因此可能同时发现簇和离群点；3.产生的离群点集和它们的得分可能非常依赖所用的簇的个数和数据中离群点的存在性；4.聚类算法产生的簇的质量对该算法产生的离群点的质量影响非常大。
 

处理方法：

1.删除异常值----明显看出是异常且数量较少可以直接删除

2.不处理---如果算法对异常值不敏感则可以不处理，但如果算法对异常值敏感，则最好不要用，如基于距离计算的一些算法，包括kmeans，knn之类的。

3.平均值替代----损失信息小，简单高效。

4.视为缺失值----可以按照处理缺失值的方法来处理

 

 

四.去重处理
以DataFrame数据格式为例：
复制代码
 1 #创建数据，data里包含重复数据
 2 >>> data = pd.DataFrame({'v1':['a']*5+['b']* 4,'v2':[1,2,2,2,3,4,4,5,3]})
 3 >>> data
 4   v1  v2
 5 0  a   1
 6 1  a   2
 7 2  a   2
 8 3  a   2
 9 4  a   3
10 5  b   4
11 6  b   4
12 7  b   5
13 8  b   3
14 
15 #DataFrame的duplicated方法返回一个布尔型Series，表示各行是否是重复行
16 >>> data.duplicated()
17 0    False
18 1    False
19 2     True
20 3     True
21 4    False
22 5    False
23 6     True
24 7    False
25 8    False
26 dtype: bool
27 
28 #drop_duplicates方法用于返回一个移除了重复行的DataFrame
29 >>> data.drop_duplicates()
30   v1  v2
31 0  a   1
32 1  a   2
33 4  a   3
34 5  b   4
35 7  b   5
36 8  b   3
37 
38 #这两个方法默认会判断全部列，你也可以指定部分列进行重复项判断。假设你还有一列值，且只希望根据v1列过滤重复项：
39 >>> data['v3']=range(9)
40 >>> data
41   v1  v2  v3
42 0  a   1   0
43 1  a   2   1
44 2  a   2   2
45 3  a   2   3
46 4  a   3   4
47 5  b   4   5
48 6  b   4   6
49 7  b   5   7
50 8  b   3   8
51 >>> data.drop_duplicates(['v1'])
52   v1  v2  v3
53 0  a   1   0
54 5  b   4   5
55 
56 #duplicated和drop_duplicates默认保留的是第一个出现的值组合。传入take_last=True则保留最后一个：
57 >>> data.drop_duplicates(['v1','v2'],take_last = True)
58   v1  v2  v3
59 0  a   1   0
60 3  a   2   3
61 4  a   3   4
62 6  b   4   6
63 7  b   5   7
64 8  b   3   8
复制代码
 

 
如果数据是列表格式的，有以下几种方法可以删除：
复制代码
 1 list0=['b','c', 'd','b','c','a','a']
 2 
 3 方法1：使用set()
 4 
 5 list1=sorted(set(list0),key=list0.index) # sorted output
 6 print( list1)
 7 
 8 方法2：使用 {}.fromkeys().keys()
 9 
10 list2={}.fromkeys(list0).keys()
11 print(list2)
12 
13 方法3：set()+sort()
14 
15 list3=list(set(list0))
16 list3.sort(key=list0.index)
17 print(list3)
18 
19 方法4：迭代
20 
21 list4=[]
22 for i in list0:
23     if not i in list4:
24         list4.append(i)
25 print(list4)
26 
27 方法5：排序后比较相邻2个元素的数据，重复的删除
28 
29 def sortlist(list0):
30     list0.sort()
31     last=list0[-1]
32     for i in range(len(list0)-2,-1,-1):
33         if list0[i]==last:
34             list0.remove(list0[i])
35         else:
36             last=list0[i]
37     return list0
38 
39 print(sortlist(list0))
复制代码
 

 五.噪音处理
　  噪音，是被测量变量的随机误差或方差。我们在上文中提到过异常点（离群点），那么离群点和噪音是不是一回事呢？我们知道，观测量(Measurement) = 真实数据(True Data) + 噪声 (Noise)。离群点(Outlier)属于观测量，既有可能是真实数据产生的，也有可能是噪声带来的，但是总的来说是和大部分观测量之间有明显不同的观测值。。噪音包括错误值或偏离期望的孤立点值，但也不能说噪声点包含离群点，虽然大部分数据挖掘方法都将离群点视为噪声或异常而丢弃。然而，在一些应用（例如：欺诈检测），会针对离群点做离群点分析或异常挖掘。而且有些点在局部是属于离群点，但从全局看是正常的。
　　我在quora上看到过一个解释噪音与离群点的有趣的例子：
Outlier: you are enumerating meticulously everything you have. You found 3 dimes, 1 quarter and wow a 100 USD bill you had put there last time you bought some booze and had totally forgot there. The 100 USD bill is an outlier, as it's not commonly expected in a pocket.

Noise: you have just come back from that club and are pretty much wasted. You try to find some money to buy something to sober up, but you have trouble reading the figures correctly on the coins. You found 3 dimes, 1 quarter and wow a 100 USD bill. But in fact, you have mistaken the quarter for a dime: this mistake introduces noise in the data you have access to.

To put it otherwise, data = true signal + noise. Outliers are part of the data.
翻译过来就是：
离群点： 你正在从口袋的零钱包里面穷举里面的钱，你发现了3个一角，1个五毛，和一张100元的毛爷爷向你微笑。这个100元就是个离群点，因为并不应该常出现在口袋里..

噪声： 你晚上去三里屯喝的酩酊大醉，很需要买点东西清醒清醒，这时候你开始翻口袋的零钱包，嘛，你发现了3个一角，1个五毛，和一张100元的毛爷爷向你微笑。但是你突然眼晕，把那三个一角看成了三个1元...这样错误的判断使得数据集中出现了噪声
 
　　那么对于噪音我们应该如何处理呢？有以下几种方法：
1.分箱法
分箱方法通过考察数据的“近邻”（即，周围的值）来光滑有序数据值。这些有序的值被分布到一些“桶”或箱中。由于分箱方法考察近邻的值，因此它进行局部光滑。
用箱均值光滑：箱中每一个值被箱中的平均值替换。
用箱中位数平滑：箱中的每一个值被箱中的中位数替换。
用箱边界平滑：箱中的最大和最小值同样被视为边界。箱中的每一个值被最近的边界值替换。
一般而言，宽度越大，光滑效果越明显。箱也可以是等宽的，其中每个箱值的区间范围是个常量。分箱也可以作为一种离散化技术使用.

2.  回归法

　　可以用一个函数拟合数据来光滑数据。线性回归涉及找出拟合两个属性（或变量）的“最佳”直线，使得一个属性能够预测另一个。多线性回归是线性回归的扩展，它涉及多于两个属性，并且数据拟合到一个多维面。使用回归，找出适合数据的数学方程式，能够帮助消除噪声。

 

六.一些实用的数据处理小工具
1.去掉文件中多余的空行
空行主要指的是（\n,\r,\r\n,\n\r等），在python中有个strip()的方法，该方法可以去掉字符串两端多余的“空白”，此处的空白主要包括空格，制表符(\t)，换行符。不过亲测以后发现，strip()可以匹配掉\n,\r\n,\n\r等，但是过滤不掉单独的\r。为了万无一失，我还是喜欢用麻烦的办法，如下：
复制代码
 1 #-*- coding :utf-8 -*-                 
 2 #文本格式化处理，过滤掉空行
 3 
 4 file = open('123.txt')
 5 
 6 i = 0
 7 while 1:
 8     line = file.readline().strip()
 9     if not line:
10         break
11     i = i + 1
12     line1 = line.replace('\r','')
13     f1 = open('filename.txt','a')
14     f1.write(line1 + '\n')
15     f1.close()
16 print str(i)
复制代码
 

 

2.如何判断文件的编码格式

复制代码
 1 #-*- coding:utf8 -*-
 2 #批量处理编码格式转换（优化）
 3 import os
 4 import chardet
 5 
 6 path1 = 'E://2016txtutf/'
 7 def dirlist(path):
 8     filelist =  os.listdir(path)
 9     for filename in filelist:
10         filepath = os.path.join(path, filename)
11         if os.path.isdir(filepath):
12             dirlist(filepath)
13         else:
14             if filepath.endswith('.txt'):
15                 f = open(filepath)
16                 data = f.read()
17                 if chardet.detect(data)['encoding'] != 'utf-8':
18                     print filepath + "----"+ chardet.detect(data)['encoding']
19 
20 dirlist(path1)
复制代码
 

3.文件编码格式转换，gbk与utf-8之间的转换

这个主要是在一些对文件编码格式有特殊需求的时候，需要批量将gbk的转utf-8的或者将utf-8编码的文件转成gbk编码格式的。

复制代码
 1 #-*- coding:gbk -*-                
 2 #批量处理编码格式转换
 3 import codecs
 4 import os
 5 path1 = 'E://dir/'
 6 def ReadFile(filePath,encoding="utf-8"):
 7     with codecs.open(filePath,"r",encoding) as f:
 8         return f.read()
 9  
10 def WriteFile(filePath,u,encoding="gbk"):
11     with codecs.open(filePath,"w",encoding) as f:
12         f.write(u)
13  
14 def UTF8_2_GBK(src,dst):
15     content = ReadFile(src,encoding="utf-8")
16     WriteFile(dst,content,encoding="gbk")
17 
18 def GBK_2_UTF8(src,dst):
19     content = ReadFile(src,encoding="gbk")
20     WriteFile(dst,content,encoding="utf-8")
21     
22 def dirlist(path):
23     filelist =  os.listdir(path)
24     for filename in filelist:
25         filepath = os.path.join(path, filename)
26         if os.path.isdir(filepath):
27             dirlist(filepath)
28         else:
29             if filepath.endswith('.txt'):
30                 print filepath
31                 #os.rename(filepath, filepath.replace('.txt','.doc'))
32                 try:
33                     UTF8_2_GBK(filepath,filepath)
34                 except Exception,ex:
35                     f = open('error.txt','a')
36                     f.write(filepath + '\n')
37                     f.close()
38 
39 dirlist(path1)

《数据转化》
一.标准化的原因
二.适用情况
三.三种数据变换方法的含义与应用
四.具体方法及代码
一）标准化
　　1.1 scale----零均值单位方差
      1.2 StandardScaler
二）归一化
     2.1 MinMaxScaler(最小最大值标准化)
     2.2 MaxAbsScaler（绝对值最大标准化）
     2.3 对稀疏数据进行标准化
     2.4 对离群点进行标准化
三）正则化
　　3.1  L1、L2正则化
四）二值化
　　4.1特征二值化
五）对类别特征进行编码
六）缺失值的插补
七）生成多项式特征 
八）自定义转换
 
 

正文：

一.标准化的原因
    通常情况下是为了消除量纲的影响。譬如一个百分制的变量与一个5分值的变量在一起怎么比较？只有通过数据标准化，都把它们标准到同一个标准时才具有可比性，一般标准化采用的是Z标准化，即均值为0，方差为1，当然也有其他标准化，比如0--1标准化等等，可根据自己的数据分布情况和模型来选择
 
二.适用情况
　　看模型是否具有伸缩不变性。
     不是所有的模型都一定需要标准化，有些模型对量纲不同的数据比较敏感，譬如SVM等。当各个维度进行不均匀伸缩后，最优解与原来不等价，这样的模型，除非原始数据的分布范围本来就不叫接近，否则必须进行标准化，以免模型参数被分布范围较大或较小的数据主导。但是如果模型在各个维度进行不均匀伸缩后，最优解与原来等价，例如logistic regression等，对于这样的模型，是否标准化理论上不会改变最优解。但是，由于实际求解往往使用迭代算法，如果目标函数的形状太“扁”，迭代算法可能收敛得很慢甚至不收敛。所以对于具有伸缩不变性的模型，最好也进行数据标准化。
 
三.三种数据变换方法的含义与应用
　　Rescaling（重缩放/归一化）：通常是指增加或者减少一个常数，然后乘以/除以一个常数，来改变数据的衡量单位。例如：将温度的衡量单位从摄氏度转化为华氏温度。
　　Normalizing（正则化）：通常是指除以向量的范数。例如：将一个向量的欧氏长度等价于1 。在神经网络中，“正则化”通常是指将向量的范围重缩放至最小化或者一定范围，使所有的元素都在[0,1]范围内。通常用于文本分类或者文本聚类中。

　　Standardizing（标准化）：通常是为了消除不同属性或样方间的不齐性，使同一样方内的不同属性间或同一属性在不同样方内的方差减小。例如：如果一个向量包含高斯分布的随机值，你可能会通过除以标准偏差来减少均值，然后获得零均值单位方差的“标准正态”随机变量。

　　那么问题是，当我们在训练模型的时候，一定要对数据进行变换吗？这得视情况而定。很多人对多层感知机有个误解，认为输入的数据必须在[0,1]这个范围内。虽然标准化后在训练模型效果会更好，但实际上并没有这个要求。但是最好使输入数据中心集中在0周围，所以把数据缩放到[0,1]其实并不是一个好的选择。

　　如果你的输出激活函数的范围是[0,1](sigmoid函数的值域)，那你必须保证你的目标值也在这个范围内。但通常请款下，我们会使输出激活函数的范围适应目标函数的分布，而不是让你的数据来适应激活函数的范围。

　　当我们使用激活函数的范围为[0,1]时，有些人可能更喜欢把目标函数缩放到[0.1,0.9]这个范围。我怀疑这种小技巧的之所以流行起来是因为反向传播的标准化太慢了导致的。但用这种方法可能会使输出的后验概率值不对。如果你使用一个有效的训练算法的话，完全不需要用这种小技巧，也没有必要去避免溢出（overflow）

 
四.具体方法及代码
一）标准化
　　1.1 scale----零均值单位方差
复制代码
 1 from sklearn import preprocessing 
 2 import numpy as np  
 3 #raw_data
 4 X = np.array([[1., -1., 2.], [2., 0., 0.], [0., 1., -1.]])  
 5 X_scaled = preprocessing.scale(X) 
 6 #output
 7 X_scaled = [[ 0.         -1.22474487  1.33630621]
 8              [ 1.22474487  0.         -0.26726124]
 9              [-1.22474487  1.22474487 -1.06904497]]
10 ＃scaled之后的数据零均值，单位方差
11 X_scaled.mean(axis=0)  # column mean: array([ 0.,  0.,  0.])  
12 X_scaled.std(axis=0)  #column standard deviation: array([ 1.,  1.,  1.])
复制代码
　

　　1.2 StandardScaler----计算训练集的平均值和标准差，以便测试数据集使用相同的变换

复制代码
 1 scaler = preprocessing.StandardScaler().fit(X) 
 2 #out:
 3  StandardScaler(copy=True, with_mean=True, with_std=True)
 4 scaler.mean_  
 5 #out: 
 6 array([ 1.,  0. ,  0.33333333])  
 7 scaler.std_ 
 8 #out:
 9  array([ 0.81649658,  0.81649658,  1.24721913]) 
10 #测试将该scaler用于输入数据，变换之后得到的结果同上
11 scaler.transform(X)
12  #out: 
13 array([[ 0., -1.22474487,  1.33630621],  [ 1.22474487, 0. , -0.26726124],  [-1.22474487,1.22474487, -1.06904497]])  
14 scaler.transform([[-1., 1., 0.]])  
15 #scale the new data, out: 
16 array([[-2.44948974,  1.22474487, -0.26726124]])
复制代码
注：1）若设置with_mean=False 或者 with_std=False，则不做centering 或者scaling处理。

 　　2）scale和StandardScaler可以用于回归模型中的目标值处理。

 

二）归一化----将数据特征缩放至某一范围(scalingfeatures to a range)

　　另外一种标准化方法是将数据缩放至给定的最小值与最大值之间，通常是０与１之间，可用MinMaxScaler实现。或者将最大的绝对值缩放至单位大小，可用MaxAbsScaler实现。

使用这种标准化方法的原因是，有时数据集的标准差非常非常小，有时数据中有很多很多零（稀疏数据）需要保存住０元素。

 　　2.1 MinMaxScaler(最小最大值标准化)

　　公式：X_std = (X - X.min(axis=0)) / (X.max(axis=0) - X.min(axis=0)) ; 

　　　　　X_scaler = X_std/ (max - min) + min

 
复制代码
 1 #例子：将数据缩放至[0, 1]间
 2 X_train = np.array([[1., -1., 2.], [2., 0., 0.], [0., 1., -1.]])
 3 min_max_scaler = preprocessing.MinMaxScaler() 
 4 X_train_minmax = min_max_scaler.fit_transform(X_train)  
 5 #out: 
 6 array([[ 0.5       ,  0.        ,  1.        ], 
 7 [ 1.        ,  0.5       ,  0.33333333],        
 8 [ 0.        ,  1.        ,  0.        ]])
 9 #将上述得到的scale参数应用至测试数据
10 X_test = np.array([[ -3., -1., 4.]])  
11 X_test_minmax = min_max_scaler.transform(X_test) #out: array([[-1.5 ,  0. , 1.66666667]])
12 #可以用以下方法查看scaler的属性
13 min_max_scaler.scale_        #out: array([ 0.5 ,  0.5,  0.33...])
14 min_max_scaler.min_         #out: array([ 0.,  0.5,  0.33...])
复制代码
 

　　2.2 MaxAbsScaler（绝对值最大标准化）
　　与上述标准化方法相似，但是它通过除以最大值将训练集缩放至[-1,1]。这意味着数据已经以０为中心或者是含有非常非常多０的稀疏数据。

复制代码
 1 X_train = np.array([[ 1., -1.,  2.],
 2                      [ 2.,  0.,  0.],
 3                     [ 0.,  1., -1.]])
 4 max_abs_scaler = preprocessing.MaxAbsScaler()
 5 X_train_maxabs = max_abs_scaler.fit_transform(X_train)
 6 # out: 
 7 array([[ 0.5, -1.,  1. ], [ 1. , 0. ,  0. ],       [ 0. ,  1. , -0.5]])
 8 X_test = np.array([[ -3., -1.,  4.]])
 9 X_test_maxabs = max_abs_scaler.transform(X_test) 
10 #out: 
11 array([[-1.5, -1. ,  2. ]])
12 max_abs_scaler.scale_  
13 #out: 
14 array([ 2.,  1.,  2.])
复制代码
　　其实在scale模块里，也提供了这两种方法： minmax_scale和maxabs_scale

 

     2.3 对稀疏数据进行标准化

　　对稀疏数据进行中心化会破坏稀疏数据的结构，这样做没什么意义。但是我们可以对稀疏数据的输入进行标准化，尤其是特征在不同的标准时。MaxAbsScaler 和 maxabs_scale是专门为稀疏数据设计的，也是常用的方法。但是scale 和 StandardScaler只接受scipy.sparse的矩阵作为输入，并且必须设置with_centering=False。否则会出现 ValueError且破坏稀疏性，而且还会无意中分配更多的内存导致内存崩溃。RobustScaler不适用于稀疏数据的输入，但是你可以用 transform 方法。

　　scalers接受压缩的稀疏行（Compressed Sparse Rows）和压缩的稀疏列（Compressed Sparse Columns）的格式（具体参考scipy.sparse.csr_matrix 和scipy.sparse.csc_matrix）。其他的稀疏格式会被转化成压缩的稀疏行（Compressed Sparse Rows）格式。为了避免这种不必要的内存拷贝，推荐使用CSR或者CSC的格式。如果数据很小，可以在稀疏矩阵上运用toarray 方法。

 

     2.4 对离群点进行标准化

　　如果你的数据有离群点（上一篇我们提到过），对数据进行均差和方差的标准化效果并不好。这种情况你可以使用robust_scale 和 RobustScaler 作为替代。它们有对数据中心化和数据的缩放鲁棒性更强的参数。

 

三）正则化
　　3.1  L1、L2正则化
复制代码
 1 x=np.array([[1.,-1.,2.],
 2             [2.,0.,0.],
 3             [0.,1.,-1.]])
 4 x_normalized=preprocessing.normalize(x,norm='l2')
 5 print(x_normalized)
 6 
 7 # 可以使用processing.Normalizer()类实现对训练集和测试集的拟合和转换
 8 normalizer=preprocessing.Normalizer().fit(x)
 9 print(normalizer)
10 normalizer.transform(x)
复制代码
 

注：稀疏数据输入：

normalize 和 Normalizer 既接受稠密数据（dense array-like），也接受稀疏矩阵（from scipy.sparse）作为输入

稀疏数据需要转换成压缩的稀疏行（Compressed Sparse Rows）格式（详见scipy.sparse.csr_matrix），为了避免不必要的内存拷贝，推荐使用CSR。

 

四）二值化
　　4.1特征二值化
特征二值化是把数值特征转化成布尔值的过程。这个方法对符合多变量伯努利分布的输入数据进行预测概率参数很有效。详细可以见这个例子sklearn.neural_network.BernoulliRBM.

此外，在文本处理中也经常会遇到二值特征值（很可能是为了简化概率推理），即使在实际中正则化后的词频或者TF-IDF的值通常只比未正则化的效果好一点点。

对于 Normalizer，Binarizer工具类通常是在Pipeline阶段（sklearn.pipeline.Pipeline）的前期过程会用到。下面举一个具体的例子：

复制代码
 1 #input
 2 X = [[ 1., -1.,  2.],
 3          [ 2.,  0.,  0.],
 4          [ 0.,  1., -1.]]
 5 #binary
 6 binarizer = preprocessing.Binarizer().fit(X)  # fit does nothing
 7 binarizer
 8 Binarizer(copy=True, threshold=0.0)
 9 #transform
10 binarizer.transform(X)
11 #out:
12 array([[ 1.,  0.,  1.],
13        [ 1.,  0.,  0.],
14        [ 0.,  1.,  0.]])     
15 
16 # 调整阈值
17 binarizer = preprocessing.Binarizer(threshold=1.1)
18 binarizer.transform(X)
19 #out：
20 array([[ 0.,  0.,  1.],
21        [ 1.,  0.,  0.],
22        [ 0.,  0.,  0.]])
23   
复制代码
 

 

注：稀疏数据输入：

binarize 和 Binarizer 既接受稠密数据（dense array-like），也接受稀疏矩阵（from scipy.sparse）作为输入

稀疏数据需要转换成压缩的稀疏行（Compressed Sparse Rows）格式（详见scipy.sparse.csr_matrix），为了避免不必要的内存拷贝，推荐使用CSR。

 

五）对类别特征进行编码

 　　我们经常会遇到一些类别特征，这些特征不是离散型的数值，而是这样的：["男性","女性"],["来自欧洲","来自美国","来自亚洲"],["使用Firefox浏览器","使用Chrome浏览器","使用Safari浏览器","使用IE浏览器"]等等。这种类型的特征可以被编码为整型（int），如["男性","来自美国","使用IE浏览器"]可以表示成[0,1,3]，["女性","来自亚洲","使用Chrome浏览器"]可以表示成[1,2,1]。这些整数式的表示不能直接作为sklearn的参数，因为我们需要的是连续型的输入，而且我们通常是有序的翻译这些特征，而不是所有的特征都是有序化的（譬如浏览器就是按人工排的序列）。

 　　将这些类别特征转化成sklearn参数中可以使用的方法是：使用one-of-K或者one-hot编码（独热编码OneHotEncoder）。它可以把每一个有m种类别的特征转化成m中二值特征。举例如下：

复制代码
1 enc = preprocessing.OneHotEncoder()
2 #input
3 enc.fit([[0, 0, 3], [1, 1, 0], [0, 2, 1], [1, 0, 2]])  
4 OneHotEncoder(categorical_features='all', dtype=<... 'float'>,handle_unknown='error', n_values='auto', sparse=True)
5 #transform
6 enc.transform([[0, 1, 3]]).toarray()
7 #out
8 array([[ 1.,  0.,  0.,  1.,  0.,  0.,  0.,  0.,  1.]])
复制代码
　　默认情况下，特征的类别数量是从数据集里自动判断出来的。当然，你也可以用n_values这个参数。我们刚刚举的例子中有两种性别，三种地名和四种浏览器，当我们fit之后就可以将我们的数据转化为数值了。从结果中来看，第一个数字代表性别([0,1]代表男性，女性），第二个数字代表地名（[0,1,2]代表欧洲、美国、亚洲），最后一个数字代表浏览器（[3,0,1,2]代表四种浏览器）

　　此外，字典格式也可以编码： Loading features from dicts

　　OneHotEncoder参数：class sklearn.preprocessing.OneHotEncoder(n_values='auto', categorical_features='all', dtype=<class 'float'>, sparse=True, handle_unknown='error')

n_values : ‘auto’, int or array of ints

每个特征的数量

‘auto’ : 从训练数据的范围中得到
int : 所有特征的最大值（number）
array : 每个特征的最大值（number）
categorical_features: “all” or array of indices or mask :

确定哪些特征是类别特征

‘all’ (默认): 所有特征都是类别特征，意味着所有特征都要进行OneHot编码
array of indices: 类别特征的数组索引
mask: n_features 长度的数组，切dtype = bool
　　非类别型特征通常会放到矩阵的右边

dtype : number type, default=np.float

　　输出数据的类型

sparse : boolean, default=True

　　设置True会返回稀疏矩阵，否则返回数组

handle_unknown : str, ‘error’ or ‘ignore’

　　当一个不明类别特征出现在变换中时，报错还是忽略

 

六）缺失值的插补

　　上篇我们讲了五种方法来解决缺失值的问题，其实sklearn里也有一个工具Imputer可以对缺失值进行插补。Imputer类可以对缺失值进行均值插补、中位数插补或者某行/列出现的频率最高的值进行插补，也可以对不同的缺失值进行编码。并且支持稀疏矩阵。

复制代码
 1 import numpy as np
 2 from sklearn.preprocessing import Imputer
 3 #用均值插补缺失值
 4 imp = Imputer(missing_values='NaN', strategy='mean', axis=0)
 5 imp.fit([[1, 2], [np.nan, 3], [7, 6]])
 6 Imputer(axis=0, copy=True, missing_values='NaN', strategy='mean', verbose=0)
 7 X = [[np.nan, 2], [6, np.nan], [7, 6]]
 8 print(imp.transform(X))                           
 9 [[ 4.          2.        ]
10  [ 6.          3.666...]
11  [ 7.          6.        ]]
12 
13 #对稀疏矩阵进行缺失值插补
14 import scipy.sparse as sp
15 X = sp.csc_matrix([[1, 2], [0, 3], [7, 6]])
16 imp = Imputer(missing_values=0, strategy='mean', axis=0)
17 imp.fit(X)
18 Imputer(axis=0, copy=True, missing_values=0, strategy='mean', verbose=0)
19 X_test = sp.csc_matrix([[0, 2], [6, 0], [7, 6]])
20 print(imp.transform(X_test))                      
21 [[ 4.          2.        ]
22  [ 6.          3.666...]
23  [ 7.          6.        ]]
复制代码
　　在稀疏矩阵中，缺失值被编码为0存储为矩阵中，这种格式是适合于缺失值比非缺失值多得多的情况。此外，Imputer类也可以用于Pipeline中

 

　　Imputor类的参数：class sklearn.preprocessing.Imputer(missing_values='NaN', strategy='mean', axis=0, verbose=0, copy=True)

missing_values : int或"NaN",默认NaN（String类型）

strategy : string, 默认为mean，可选则mean、median、most_frequent

axis :int, 默认为0（axis = 0，对列进行插值；axis= 1，对行进行插值）

verbose : int, 默认为0

copy : boolean, 默认为True

　　True：会创建一个X的副本

　　False：在任何合适的地方都会进行插值。

　　但是以下四种情况，计算设置的copy = Fasle，也会创建一个副本：

　　1.X不是浮点型数组

　　2.X是稀疏矩阵，而且miss_value = 0

　　3.axis= 0，X被编码为CSR矩阵

　　　　　　 4.axis= 1，X被编码为CSC矩阵

 

　　举个实例(在用随机森林算法之前先用Imputer类进行处理)：

复制代码
 1 import numpy as np
 2 
 3 from sklearn.datasets import load_boston
 4 from sklearn.ensemble import RandomForestRegressor
 5 from sklearn.pipeline import Pipeline
 6 from sklearn.preprocessing import Imputer
 7 from sklearn.cross_validation import cross_val_score
 8 
 9 rng = np.random.RandomState(0)
10 
11 dataset = load_boston()
12 X_full, y_full = dataset.data, dataset.target
13 n_samples = X_full.shape[0]
14 n_features = X_full.shape[1]
15 
16 # Estimate the score on the entire dataset, with no missing values
17 estimator = RandomForestRegressor(random_state=0, n_estimators=100)
18 score = cross_val_score(estimator, X_full, y_full).mean()
19 print("Score with the entire dataset = %.2f" % score)
20 
21 # Add missing values in 75% of the lines
22 missing_rate = 0.75
23 n_missing_samples = np.floor(n_samples * missing_rate)
24 missing_samples = np.hstack((np.zeros(n_samples - n_missing_samples,
25                                       dtype=np.bool),
26                              np.ones(n_missing_samples,
27                                      dtype=np.bool)))
28 rng.shuffle(missing_samples)
29 missing_features = rng.randint(0, n_features, n_missing_samples)
30 
31 # Estimate the score without the lines containing missing values
32 X_filtered = X_full[~missing_samples, :]
33 y_filtered = y_full[~missing_samples]
34 estimator = RandomForestRegressor(random_state=0, n_estimators=100)
35 score = cross_val_score(estimator, X_filtered, y_filtered).mean()
36 print("Score without the samples containing missing values = %.2f" % score)
37 
38 # Estimate the score after imputation of the missing values
39 X_missing = X_full.copy()
40 X_missing[np.where(missing_samples)[0], missing_features] = 0
41 y_missing = y_full.copy()
42 estimator = Pipeline([("imputer", Imputer(missing_values=0,
43                                           strategy="mean",
44                                           axis=0)),
45                       ("forest", RandomForestRegressor(random_state=0,
46                                                        n_estimators=100))])
47 score = cross_val_score(estimator, X_missing, y_missing).mean()
48 print("Score after imputation of the missing values = %.2f" % score)
复制代码
 

　　结果：

Score with the entire dataset = 0.56
Score without the samples containing missing values = 0.48
Score after imputation of the missing values = 0.55
 

七）生成多项式特征 

　　在输入数据中增加非线性特征可以有效的提高模型的复杂度。简单且常用的方法就是使用多项式特征（polynomial features),可以得到特征的高阶交叉项：

复制代码
 1 import numpy as np
 2 from sklearn.preprocessing import PolynomialFeatures
 3 X = np.arange(6).reshape(3, 2)
 4 X                                                 
 5 array([[0, 1],
 6        [2, 3],
 7        [4, 5]])
 8 poly = PolynomialFeatures(2)
 9 poly.fit_transform(X)                             
10 array([[  1.,   0.,   1.,   0.,   0.,   1.],
11        [  1.,   2.,   3.,   4.,   6.,   9.],
12        [  1.,   4.,   5.,  16.,  20.,  25.]])
复制代码
  

　　然而有时候我们只需要特征的交叉项，可以设置interaction_only=True来得到：

复制代码
 1 X = np.arange(9).reshape(3, 3)
 2 X                                                 
 3 array([[0, 1, 2],
 4        [3, 4, 5],
 5        [6, 7, 8]])
 6 poly = PolynomialFeatures(degree=3, interaction_only=True)
 7 poly.fit_transform(X)                             
 8 array([[   1.,    0.,    1.,    2.,    0.,    0.,    2.,    0.],
 9        [   1.,    3.,    4.,    5.,   12.,   15.,   20.,   60.],
10        [   1.,    6.,    7.,    8.,   42.,   48.,   56.,  336.]])
复制代码
 　  这个方法可能大家在工作中比较少见，但世界上它经常用于核方法中，如选择多项式核时 ( sklearn.svm.SVC, sklearn.decomposition.KernelPCA)

 

八）自定义转换

　　如果以上的方法觉得都不够，譬如你想用对数据取对数，可以自己用 FunctionTransformer自定义一个转化器,并且可以在Pipeline中使用

复制代码
1 import numpy as np
2 from sklearn.preprocessing import FunctionTransformer
3 transformer = FunctionTransformer(np.log1p)#括号内的就是自定义函数
4 X = np.array([[0, 1], [2, 3]])
5 transformer.transform(X)
6 array([[ 0.        ,  0.69314718],
7        [ 1.09861229,  1.38629436]])
复制代码
 

　　告诉你怎么用：

　　如果你在做一个分类任务时，发现第一主成分与这个不相关，你可以用FunctionTransformer把第一列除去，剩下的列用PCA：

复制代码
 1 import matplotlib.pyplot as plt
 2 import numpy as np
 3 
 4 from sklearn.cross_validation import train_test_split
 5 from sklearn.decomposition import PCA
 6 from sklearn.pipeline import make_pipeline
 7 # from sklearn.preprocessing import FunctionTransformer
 8 # 如果报错ImportError: cannot import name FunctionTransformer，可以使用下面的语句
 9 from sklearn.preprocessing import *
10 
11 
12 def _generate_vector(shift=0.5, noise=15):
13     return np.arange(1000) + (np.random.rand(1000) - shift) * noise
14 
15 
16 def generate_dataset():
17     """
18     This dataset is two lines with a slope ~ 1, where one has
19     a y offset of ~100
20     """
21     return np.vstack((
22         np.vstack((
23             _generate_vector(),
24             _generate_vector() + 100,
25         )).T,
26         np.vstack((
27             _generate_vector(),
28             _generate_vector(),
29         )).T,
30     )), np.hstack((np.zeros(1000), np.ones(1000)))
31 
32 
33 def all_but_first_column(X):
34     return X[:, 1:]
35 
36 
37 def drop_first_component(X, y):
38     """
39     Create a pipeline with PCA and the column selector and use it to
40     transform the dataset.
41     """
42     pipeline = make_pipeline(
43         PCA(), FunctionTransformer(all_but_first_column),
44     )
45     X_train, X_test, y_train, y_test = train_test_split(X, y)
46     pipeline.fit(X_train, y_train)
47     return pipeline.transform(X_test), y_test
48 
49 
50 if __name__ == '__main__':
51     X, y = generate_dataset()
52     plt.scatter(X[:, 0], X[:, 1], c=y, s=50)
53     plt.show()
54     X_transformed, y_transformed = drop_first_component(*generate_dataset())
55     plt.scatter(
56         X_transformed[:, 0],
57         np.zeros(len(X_transformed)),
58         c=y_transformed,
59         s=50,
60     )
61     plt.show()
