# ER模型

## 基本概念

* 数据（data）：描述事物的符号记录称为数据。
* 数据库（DataBase，DB）：是长期存储在计算机内、有组织的、可共享的大量数据的集合，具有永久存储、有组织、可共享三个基本特点。
* 数据库管理系统（DataBase Management System，DBMS）：是位于用户与操作系统之间的一层数据管理软件。
* 数据库系统（DataBase System，DBS）：是有数据库、数据库管理系统（及其应用开发工具）、应用程序和数据库管理员（DataBase Administrator DBA）组成的存储、管理、处理和维护数据的系统。
* 模式（schema）：模式也称逻辑模式，是数据库全体数据的逻辑结构和特征的描述，是所有用户的公共数据视图。
* 外模式（external schema）：外模式也称子模式（subschema）或用户模式，它是数据库用户（包括应用程序员和最终用户）能够看见和使用的局部数据的逻辑结构和特征的描述，是数据库用户的数据视图，是与某一应用有关的数据的逻辑表示。
* 内模式（internal schema）：内模式也称为存储模式（storage schema），一个数据库只有一个内模式。他是数据物理结构和存储方式的描述，是数据库在数据库内部的组织方式。

## 实体

+ 实体 （ entity ）是现实世界中可区别于所有其他对象的一个“事物”或“对象 ” 。  

+ 实体集 （ entity set ）是相同类型 即具有相同性质（或属性）的一个实体集合 。

+ 属性（ attribute ）是实体集中每个成员所拥有的描述性性质。
    + 实体通过一组属性来表示 。     
+ 值 （ value ），每个实体的每个属性都有一个值 。  

## 联系

+ 联系 （ relationship ）是指多个实体间的相互关联 。  

+ 联系集 （ relationship set）是相同类型联系的集合 。 

    + 正规地说，联系集是 $n\ge 2$ 个（可能相同的 ）实体集上的数学关系 。 

    + 如果 $E_1,E_2,···,E_n$ 为实体集 ， 则联系集 $R$ 是 $\{(e_1,e_2,···,e_n)|e_1\in E_1,e_2\in E_2,···,e_n\in E_n\}$ 的一个子集， $(e_1,e_2,···,e_n)$ 是一个联系 。

    + 考虑两个实体集 instructor 和 studenl 。我们定义联系集 advisor 来表示教师及学生之间的关联 。 这一关联如图所示。

        ![image-20210905201211992](https://gitee.com/TuTouPower/feng-image/raw/master/image-20210905201211992.png)

        <center> 联系集 advisor

+ 参与（participate）是实体集之间的关联；即实体集 $E_1,E_2,···,E_n$ 参与联系集 R。 

+ 联系实例 （ relationship instance ）是所建模的现实世界中命名实体间的一个关联 。

    + 例如， 一个教师 ID 为 45565 的 instructor 实体 Katz 和一个学生 ID 为 12345 的 student 实体 Shankar 参与到 advisor 的一个联系实例中 。 这一联系实例表示教师 Katz 指导学生 Shankar 。

+ 角色 （ role ）是实体在联系中扮演的功能 。 

+ 联系也可以具有描述性属性 （ descriptive attrihute ） 。

    + 考虑实体集 instructor 和 student 之间的联系集advisor 。 我们可以将属性 date 与该联系关联起来 ， 以表示教师成为学生的导 师的日期 。 教师 Katz 对应的实体和学生 Shankar 对应的实体之间的联系 αdvisor 的属性 date 的值为“ 10 June 2007 ”，表示 Katz 于2007 年 6 月 10 日成为 Shankar 的导师 。

+ 二元 （ binary）联系集是涉及两个实体集的联系集。

+ 参与联系集的实体集的数目称为联系集的度 （ degree ） 。

    +  二元联系集的度为 2 ； 三元联系集的度为 3 。    

## 属性

每个属性都有一个可取值的集合，称为该属性的**域 （ domain ）**， 或者**值集 （ value set ）** 。  

正规地说，实体集的属性是将实体集映射到域的函数。 由于一个实体集可能有多个属性，因此每个实体可以用一组 （属性，数据值）对来表示，实体集的每个属性对应一个这样的对。 例如，某个instructor 实体可以用集合 $\{ (ID, 76766) , ( name , Crick) , ( dept_name ，生 物） , ( salary，72000）\}$来描述。

E-R 模型中的属性可以按照如下的属性类型来进行划分：·

+ 简单 （ simple ）和复合 （ composite ）属性。 例子中 ， 迄今为止出现的属性都是简单属性 ，也就是说，它们不能划分为更小的部分。 另一方面，复合属性可以再划分为更小的部分（ 即其他属性） 。 例如，属性 name 可设计为一个包括 first_name 、 middle_initial 和 last_name 的复合属性 。

+ 单值 （ single- valued ）和多值 （ multivalude ）属性。 我们的例子中的属性对一个特定实体都只有单独的一个值。 例如，对某个特定的学生实体而言， student_ID 属性只对应于一个学生 ID。 这样的属性称作是单值 （ single valued ）的 。 而在某些情况下对某个特定实体而言，一个属性可能对应于一组值 。 假设我们往 instructor 实体集添加一个 phone_number 属性 ，每个教师可以有零个 、一个或多个电话号码，不同的教师可以有不同数量的电话。 这样的属性称作是多值 （ multivalued)的 。
+ 派生 （ derived ）属性。 这类属性的值可以从别的相关属性或实体派生出 来 。 例如，让我们假设instructor 实体集有一个属性 students_advised ，表示一个教师指导了多少个学生 。 我们可以通过统计与一个教师相关联的所有 student 实体的数目来得到这个属性的值。

当实体在某个属性上没有值时使用空 （ null ）值。 空值可以表示“ 不适用”，即该实体的这个属性不存在值。 例如， 一个人可能没有中间名字 。 空还可以用来表示属性值未知 。 未知的值可能是缺失的 （ 值存在，但我们没有该信息） ， 或不知道的（我们并不知道该值是否确实存在 ） 。  

## 映射基数

映射基数 （ mapping cardinality ）， 或基数比率 ，表示一个实体通过一个联系集能关联的实体的个数。

对于实体集 A 和 B 之间的二元联系集 R 来说，映射基数必然是以下情况之一 ：

+ 一对一 （ one to one ）：
    +  A 中的一个实体至多与 B 中的一个实体相关联 ，
    +  B 中的一个实体也至多与 A 中的一个实体相关联 。
+ 一对多（ one to many ） ：
    + A 中的一个实体可以与 B 中的任意数目（零个或多个）实体相关联 ，
    + B中的一个实体至多与 A 中的一个实体相关联 。
+ 多对一（ many to one ）：
    + A 中的一个实体至多与 B 中的一个实体相关联，
    + B 中的一个实体可 以与 A 中任意数目（零个或多个 ）实体相关联 。
+ 多对多（ many to many ）：
    + A 中的一个实体可以与 B 中任意数 目 （ 零个或多个）实体相关联，
    + B 中的一个实体也可以与 A 中任意数目（零个或多个）实体相关联。

![image-20210929162817291](https://gitee.com/TuTouPower/feng-image/raw/master/image-20210929162817291.png)

若实体集 E 中的每个实体都参与到联系集 R 的至少一个联系中，实体集 E 在联系集 R 中的参与称为**全部 （total ）** 的 。 如果 E 中只有部分实体参与到 R 的联系中，实体集 E 到联系集 R 的参与称为**部分(partial ）**的 。

## 实体 - 联系图

E-R 图包括如下几个主要构件： 

+ 分成两部分的矩形代表实体集 。
+ 菱形代表联系集 。
+ 未分割的矩形代表联系集的属性。 构成主码的属性以下划线标明 。
+ 线段将实体集连接到联系集。
+ 虚线将联系集属性连接到联系集 。
+ 双线显示实体在联系集中的参与度 。
+ 双菱形代表连接到弱实体集的标志性联系集。

考虑图中的 E-R 图，它由通过二元联系集 advisor 关联的两个实体集 instructor 和 student 组成。同 instructor 相关联的属性为 ID 、 name 和 salary。 同 student 相关联的属性为 ID 、 name 和 tot_cred 。 如图所示，实体集的属性中那些组成主码的属性以下划线标明 。

![image-20210907205652445](https://gitee.com/TuTouPower/feng-image/raw/master/image-20210907205652445.png)

<center>对应于教师和学生的 E-R 图  

如果一个联系集有关联的属性，那么我们将这些属性放入一个矩形中，并且用虚线将该矩形与代表联系集的菱形连接起来 。 例如 ，在图中 ，我们有描述性属性 date 附带到联系集 advisor 上．表示教师成为导师的日期 。

![image-20210907205705323](https://gitee.com/TuTouPower/feng-image/raw/master/image-20210907205705323.png)

<center>联系集上附带了一个属性的 E-R 图
## 实体的三种联系

包含一对一，一对多，多对多三种。

- 如果 A 到 B 是一对多关系，那么画个带箭头的线段指向 B；
- 如果是一对一，画两个带箭头的线段；
- 如果是多对多，画两个不带箭头的线段。

下图的 Course 和 Student 是一对多的关系。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/1d28ad05-39e5-49a2-a6a1-a6f496adba6a.png"/> </div><br>

## 表示出现多次的关系

一个实体在联系出现几次，就要用几条线连接。

下图表示一个课程的先修关系，先修关系出现两个 Course 实体，第一个是先修课程，后一个是后修课程，因此需要用两条线来表示这种关系。successor

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/ac929ea3-daca-40ec-9e95-4b2fa6678243.png"/> </div><br>

## 联系的多向性

虽然老师可以开设多门课，并且可以教授多名学生，但是对于特定的学生和课程，只有一个老师教授，这就构成了一个三元联系。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/5bb1b38a-527e-4802-a385-267dadbd30ba.png" width="350px"/> </div><br>

## 表示子类

用一个三角形和两条线来连接类和子类，与子类有关的属性和联系都连到子类上，而与父类和子类都有关的连到父类上。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/14389ea4-8d96-4e96-9f76-564ca3324c1e.png" width="450px"/> </div><br>
