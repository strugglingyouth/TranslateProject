
翻译中 by ideas4u
### 理解数据
我们来简单看一下原始数据文件。下面是2012年1季度前几行的采集数据。
```
100000853384|R|OTHER|4.625|280000|360|02/2012|04/2012|31|31|1|23|801|N|C|SF|1|I|CA|945||FRM|
100003735682|R|SUNTRUST MORTGAGE INC.|3.99|466000|360|01/2012|03/2012|80|80|2|30|794|N|P|SF|1|P|MD|208||FRM|788
100006367485|C|PHH MORTGAGE CORPORATION|4|229000|360|02/2012|04/2012|67|67|2|36|802|N|R|SF|1|P|CA|959||FRM|794
```

下面是2012年1季度的前几行执行数据
```
100000853384|03/01/2012|OTHER|4.625||0|360|359|03/2042|41860|0|N||||||||||||||||
100000853384|04/01/2012||4.625||1|359|358|03/2042|41860|0|N||||||||||||||||
100000853384|05/01/2012||4.625||2|358|357|03/2042|41860|0|N||||||||||||||||
```
在开始编码之前，花些时间真正理解数据是值得的。这对于操作项目优为重要，因为我们没有交互式探索数据，将很难察觉到细微的差别除非我们在前期发现他们。在这种情况下，第一个步骤是阅读房利美站点的资料：
- [概述][15]
- [有用的术语表][16]
- [问答][17]
- [采集和执行文件中的列][18]
- [采集数据文件样本][19]
- [执行数据文件样本][20]

在看完这些文件后后，我们了解到一些能帮助我们的关键点：
- 从2000年到现在，每季度都有一个采集和执行文件，因数据是滞后一年的，所以到目前为止最新数据是2015年的。
- 这些文件是文本格式的，采用管道符号“|”进行分割。
- 这些文件是没有表头的，但我们有文件列明各列的名称。
- 所有一起，文件包含2200万个贷款的数据。
由于执行文件包含过去几年获得的贷款的信息，在早些年获得的贷款将有更多的执行数据（即在2014获得的贷款没有多少历史执行数据）。
这些小小的信息将会为我们节省很多时间，因为我们知道如何构造我们的项目和利用这些数据。

### 构造项目
在我们开始下载和探索数据之前，先想一想将如何构造项目是很重要的。当建立端到端项目时，我们的主要目标是：
- 创建一个可行解决方案
- 有一个快速运行且占用最小资源的解决方案
- 容易可扩展
- 写容易理解的代码
- 写尽量少的代码

为了实现这些目标，需要对我们的项目进行良好的构造。一个结构良好的项目遵循几个原则：
- 分离数据文件和代码文件
- 从原始数据中分离生成的数据。
- 有一个README.md文件帮助人们安装和使用该项目。
- 有一个requirements.txt文件列明项目运行所需的所有包。
- 有一个单独的settings.py 文件列明其它文件中使用的所有的设置
    - 例如，如果从多个Python脚本读取相同的文件，把它们全部import设置和从一个集中的地方获得文件名是有用的。
- 有一个.gitignore文件，防止大的或秘密文件被提交。
- 分解任务中每一步可以单独执行的步骤到单独的文件中。
    - 例如，我们将有一个文件用于读取数据，一个用于创建特征，一个用于做出预测。
- 保存中间结果，例如，一个脚本可输出下一个脚本可读取的文件。

    - 这使我们无需重新计算就可以在数据处理流程中进行更改。
    

我们的文件结构大体如下：

```
loan-prediction
├── data
├── processed
├── .gitignore
├── README.md
├── requirements.txt
├── settings.py
```

### 创建初始文件
To start with, we’ll need to create a loan-prediction folder. Inside that folder, we’ll need to make a data folder and a processed folder. The first will store our raw data, and the second will store any intermediate calculated values.

Next, we’ll make a .gitignore file. A .gitignore file will make sure certain files are ignored by git and not pushed to Github. One good example of such a file is the .DS_Store file created by OSX in every folder. A good starting point for a .gitignore file is here. We’ll also want to ignore the data files because they are very large, and the Fannie Mae terms prevent us from redistributing them, so we should add two lines to the end of our file:

```
data
processed
```

[Here’s][21] an example .gitignore file for this project.

Next, we’ll need to create README.md, which will help people understand the project.  .md indicates that the file is in markdown format. Markdown enables you write plain text, but also add some fancy formatting if you want. [Here’s][22] a guide on markdown. If you upload a file called README.md to Github, Github will automatically process the markdown, and show it to anyone who views the project. [Here’s][23] an example.

For now, we just need to put a simple description in README.md:

```
Loan Prediction
-----------------------

Predict whether or not loans acquired by Fannie Mae will go into foreclosure.  Fannie Mae acquires loans from other lenders as a way of inducing them to lend more.  Fannie Mae releases data on the loans it has acquired and their performance afterwards [here](http://www.fanniemae.com/portal/funding-the-market/data/loan-performance-data.html).
```

Now, we can create a requirements.txt file. This will make it easy for other people to install our project. We don’t know exactly what libraries we’ll be using yet, but here’s a good starting point:

```
pandas
matplotlib
scikit-learn
numpy
ipython
scipy
```

The above libraries are the most commonly used for data analysis tasks in Python, and its fair to assume that we’ll be using most of them. [Here’s][24] an example requirements file for this project.

After creating requirements.txt, you should install the packages. For this post, we’ll be using Python 3. If you don’t have Python installed, you should look into using [Anaconda][25], a Python installer that also installs all the packages listed above.

Finally, we can just make a blank settings.py file, since we don’t have any settings for our project yet.
