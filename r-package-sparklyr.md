# R Package (sparklyr)

## Basic sparklyr

Package ‘sparklyr’: [https://cran.r-project.org/web/packages/sparklyr/sparklyr.pdf](https://cran.r-project.org/web/packages/sparklyr/sparklyr.pdf)

sparklyr是与Apache Spark的R接口，Apache Spark是用于大数据处理的快速通用引擎，该软件包支持连接到本地和远程Apache Spark集群，提供“ dplyr”兼容的后端，并提供Spark内置机器学习算法的接口。

### Installation and connecting

```
## install.packages("sparklyr")
## packageVersion("sparklyr")
library('sparklyr')

## returns a path with spaces
getwd()  
  ## reset path
  ## setwd("path")

## Spark是由Scala编程语言（由Java虚拟机（JVM）运行）构建的，因此您还需要在系统上安装Java 8
system("java -version") 

## 用Spark 2.3编写的，因此您还应该安装此版本，以确保可以遵循提供的所有示例，而不会出现任何意外
spark_install("2.3")
  ## display all of the versions of Spark that are available for installation
  ## spark_available_versions()

## 检查安装了哪些版本
spark_installed_versions()

##   spark  hadoop                                                             dir
## 1 2.3.3    2.7 C:\\Users\\zbai\\AppData\\Local/spark/spark-2.3.3-bin-hadoop2.7

  ## 要卸载特定版本的Spark，可以通过指定Spark和Hadoop版本来运行spark_uninstall（）
  ## spark_uninstall(version = "1.6.3", hadoop = "2.6")

## 连接到此本地集群
## 本地集群对于轻松入门，测试代码和排除故障确实很有帮助
sc <- spark_connect(master = "local", version = "2.3")
  ## 建立连接后，spark_connect（）检索活动的Spark连接，
  ## 大多数代码通常将其命名为sc； 然后，您将使用sc执行Spark命令。
## Disconnecting
## spark_disconnect(sc)

## Logs
spark_log(sc)
spark_log(sc, filter = "sparklyr")
```

### Using Spark&#x20;

```
## Using Spark
## 将mtcars数据集复制到Apache Spark中
cars <- copy_to(sc, mtcars)
## access data
cars
  ## copy_to()
  ## 第一个参数sc为该函数提供对之前使用spark_connect（）创建的活动Spark连接的引用
  ## 第二个参数指定要加载到Spark中的数据集
```

Most of the Spark commands are executed from the R console; however, monitoring and analyzing execution is done through Spark’s web interface

> 大多数Spark命令都是从R控制台执行的； 但是，监视和分析执行是通过Spark的Web界面完成的

Printing the cars dataset collected a few records to be displayed in the R console. You can see in the Spark web interface that a job was started to collect this information back from Spark. You can also select the Storage tab to see the mtcars dataset cached in memory in Spark

```
## Web Interface
## 大多数Spark命令都是从R控制台执行的； 但监视和分析执行是通过Spark的Web界面完成的
## access Web Interface
spark_web(sc)
```

### Analysis

在R中使用Spark来分析数据时，可以使用SQL（结构化查询语言）或dplyr（数据处理语法）。 您可以通过DBI包使用SQL。

```
## Analysis
## using Spark from R to analyze data, can use SQL (Structured Query Language) 
## or dplyr (a grammar of data manipulation)
## use SQL through the DBI package
library(DBI)
dbGetQuery(sc, "SELECT count(*) FROM mtcars")

library(dplyr)
select(cars, hp, mpg) %>%
  sample_n(100) %>%
  collect() %>%
  plot()



## Modeling
## ml_linear_regression {sparklyr}
model <- ml_linear_regression(cars, mpg ~ hp)
model %>%
  ml_predict(copy_to(sc, data.frame(hp = 250 + 10 * 1:10))) %>%
  transmute(hp = hp, mpg = prediction) %>%
  full_join(select(cars, hp, mpg)) %>%
  collect() %>%
  plot()
```

### Data and Streaming

```
## Data
## 通常不会将数据复制到Spark中
##  export our cars dataset
spark_write_csv(cars, "cars.csv")
## read back from the local file system
cars <- spark_read_csv(sc, "cars.csv")


## Streaming:  dynamic datasets
## 可以将流数据集视为静态数据源，不断有新数据到达
## 流数据通常是从Kafka（开源流处理软件平台）或从连续接收新数据的分布式存储中读取的。
## 流式传输，首先让我们创建一个Input /文件夹，其中包含一些数据，这些数据将用作该流的输入
## 定义一个流，该流处理来自input /文件夹的传入数据，在R中执行自定义转换，并将输出推送到output /文件夹：
dir.create("input")
write.csv(mtcars, "input/cars_1.csv", row.names = F)
stream <- stream_read_csv(sc, "input/") %>%
  select(mpg, cyl, disp) %>%
  stream_write_csv("output/")

## 输出文件夹还将包含一个通过应用自定义spark_apply（）转换产生的文件
## [1] "part-00000-40270e83-c741-476d-a531-244c668e55ba-c000.csv"
dir("output", pattern = ".csv")

## 可以继续将文件添加到输入/位置，Spark将自动并行化和处理数据
## Write more data into the stream source
write.csv(mtcars, "input/cars_2.csv", row.names = F)

## 检查流目的地的内容
dir("output", pattern = ".csv")
## [1] "part-00000-40270e83-c741-476d-a531-244c668e55ba-c000.csv"
## [2] "part-00000-92b0ab2f-ece0-4e33-8e27-f6168d123254-c000.csv"

## Stop the stream
stream_stop(stream)
```

## Analysis

Spark是（“““parallel execution”）并行计算引擎，可大规模运行，并提供SQL引擎和建模库。您可以使用它们执行R执行的大多数相同操作。这些操作包括数据选择，转换和建模。此外，Spark包含用于执行专门的计算工作的工具，例如图形分析，流处理等。

可以在Spark中执行数据导入，整理和建模。您还可以部分使用Spark做可视化，想法是使用R告诉Spark要运行哪些数据操作，然后仅将结果带入R。如图所示，理想的方法将计算推入Spark集群，然后将结果收集到R中。

![Spark computes while R collects results](broken-reference)

sparklyr软件包有助于使用“推算，收集结果”原理。 它的大多数功能是Spark API调用的包装器。 这使我们能够利用Spark的分析组件而不是R的组件。 例如，当您需要拟合线性回归模型时，可以使用Spark的ml\_linear\_regression（）函数来代替R熟悉的lm（）函数。 然后，此R函数调用Spark创建此模型。

### Data wrangling

```
## open a new local connection.
library(sparklyr)
library(dplyr)
sc <- spark_connect(master = "local", version = "2.3")

## 通常，导入意味着R将读取文件并将其加载到内存中； 
## 使用Spark时，数据将导入到Spark中，而不是R中。
## 可以请求Spark访问数据源而不导入数据，而不是将所有数据导入Spark
## 将所有数据导入到Spark会话中会产生一次性的前期加载成本
## using real clusters, you should use copy_to() to transfer only small tables from 
## large data transfers should be performed with specialized data transfer tools.

cars %>%
  mutate(transmission = ifelse(am == 0, "automatic", "manual")) %>%
  group_by(transmission) %>%
  summarise_all(mean)  %>%
  show_query()
  
  
# Source: spark<?> [?? x 12]
  transmission   mpg   cyl  disp    hp  drat    wt  qsec    vs    am  gear  carb
  <chr>        <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
1 manual        24.4  5.08  144.  127.  4.05  2.41  17.4 0.538     1  4.38  2.92
2 automatic     17.1  6.95  290.  160.  3.29  3.77  18.2 0.368     0  3.21  2.74
```

&#x20;SELECT `transmission`, AVG(`mpg`) AS `mpg`, AVG(`cyl`) AS `cyl`, AVG(`disp`) AS `disp`, AVG(`hp`) AS `hp`, AVG(`drat`) AS `drat`, AVG(`wt`) AS `wt`, AVG(`qsec`) AS `qsec`, AVG(`vs`) AS `vs`, AVG(`am`) AS `am`, AVG(`gear`) AS `gear`, AVG(`carb`) AS `carb` FROM (SELECT `mpg`, `cyl`, `disp`, `hp`, `drat`, `wt`, `qsec`, `vs`, `am`, `gear`, `carb`, IF(ISNULL(`am` = 0.0), NULL, IF(`am` = 0.0, "automatic", "manual")) AS `transmission` FROM `cars_7fbe961a_559c_45d7_81e2_be4cceaf0a2c`) `q01` GROUP BY `transmission`

Spark SQL基于Hive的SQL conventions and functions, and it is possible to call all these functions using dplyr as well. This means that we can use any Spark SQL functions to accomplish operations that might not be available via dplyr. We can access the functions by calling them as if they were R functions.&#x20;

> 也可以使用dplyr调用所有这些函数。 这意味着我们可以使用任何Spark SQL函数来完成dplyr可能无法使用的操作。 我们可以像调用R函数一样调用它们。

```
summarise(cars, mpg_percentile = percentile(mpg, 0.25)) %>%
  show_query()
```

要将多个值传递给percentile（），我们可以调用另一个名为array（）的Hive函数。 在这种情况下，array（）的作用类似于R的list（）函数。 我们可以传递多个用逗号分隔的值。 Spark的输出是一个数组变量，将其作为列表变量列导入R中：

```
summarise(cars, mpg_percentile = percentile(mpg, array(0.25, 0.5, 0.75)))

## Source: spark
  mpg_percentile
  <list>        
1 <dbl [3]>     
```

使用explode（）函数将Spark的数组值结果分成各自的记录。&#x20;

```
summarise(cars, mpg_percentile = percentile(mpg, array(0.25, 0.5, 0.75))) %>%
  mutate(mpg_percentile = explode(mpg_percentile))

# Source: 
  mpg_percentile
           <dbl>
1           15.4
2           19.2
3           22.8
```

### Visualize

R擅长数据可视化。众多专注于此分析步骤的R包扩展了其创建图的功能。不幸的是，创建图的绝大多数R函数取决于R中本地内存中已经存在的数据，因此在Spark中使用远程表时它们会失败。可以从Spark中存在的数据源在R中创建可视化。，可以在Spark内完成准备数据的繁重工作，例如按组或箱汇总数据，然后可以将更小的数据集收集到R中。

```
library(ggplot2)
car_group <- cars %>%
  group_by(cyl) %>%
  summarise(mpg = sum(mpg, na.rm = TRUE)) %>%
  collect() %>%
  print()
ggplot(aes(as.factor(cyl), mpg), data = car_group) + 
  geom_col(fill = "#999999") + coord_flip()
```

###

### Caching

在实际场景中，大量数据用于模型。 如果需要首先转换数据，则数据量可能会在Spark会话上造成沉重的负担。 在拟合模型之前，最好将所有转换的结果保存在Spark内存中加载的新表中。compute（）命令可以使用dplyr命令的末尾，并将结果保存到Spark内存中：

```
cached_cars <- cars %>% 
  mutate(cyl = paste0("cyl_", cyl)) %>%
  compute("cached_cars")
cached_cars %>%
  ml_linear_regression(mpg ~ .) %>%
  summary()
```

### Modeling

```
cars %>% 
  ml_linear_regression(mpg ~ hp + cyl) %>%
  summary() 

## generalized linear model:
cars %>% 
  ml_generalized_linear_regression(mpg ~ hp + cyl) %>%
  summary()
```

### Exploratory Data Analysis

```
## download this dataset as follows:
download.file(
  "https://github.com/r-spark/okcupid/raw/master/profiles.csv.zip",
  "okcupid.zip")
unzip("okcupid.zip", exdir = "data")
unlink("okcupid.zip")

library(sparklyr)
library(ggplot2)
library(dbplot)
library(dplyr)

## connect to Spark, load libraries, and read in the data:
sc <- spark_connect(master = "local", version = "2.3")
okc <- spark_read_csv(
  sc, 
  "data/profiles.csv", 
  escape = "\"", 
  memory = FALSE,
  options = list(multiline = TRUE)
) %>%
  mutate(
    height = as.numeric(height),
    income = ifelse(income == "-1", NA, as.numeric(income))
  ) %>%
  mutate(sex = ifelse(is.na(sex), "missing", sex)) %>%
  mutate(drinks = ifelse(is.na(drinks), "missing", drinks)) %>%
  mutate(drugs = ifelse(is.na(drugs), "missing", drugs)) %>%
  mutate(job = ifelse(is.na(job), "missing", job))

## glimpse（）快速查看数据
glimpse(okc)
```

Exploratory data analysis (EDA), in the context of predictive modeling, is the exercise of looking at excerpts and summaries of the data. The specific goals of the EDA stage are informed by the business problem, but here are some common objectives:

> 探索性数据分析（EDA）是查看数据摘录和摘要的练习。 EDA阶段的特定目标是由业务问题决定的，但以下是一些常见目标：

* Check for data quality; confirm meaning and prevalence of missing values and reconcile statistics against existing controls.检查数据质量； 确认缺失值的含义和普遍性，并使统计数据与现有控件保持一致。&#x20;
* Understand univariate relationships between variables.了解变量之间的单变量关系。&#x20;
* Perform an initial assessment on what variables to include and what transformations need to be done on them.对要包括哪些变量以及需要对它们进行哪些转换进行初步评估。

```
## add our response variable
okc <- okc %>%
  mutate(
    not_working = ifelse(job %in% c("student", "unemployed", "retired"), 1 , 0)
  )
okc %>% 
  group_by(not_working) %>% 
  tally()

##  split of our data into a training set and a testing set 
data_splits <- sdf_random_split(okc, training = 0.8, testing = 0.2, seed = 42)
okc_train <- data_splits$training
okc_test <- data_splits$testing

##  obtain numerical summaries of specific columns
sdf_describe(okc_train, cols = c("age", "income"))


# Source: spark<?> [?? x 3]
  summary age                income            
  <chr>   <chr>              <chr>             
1 count   48102              9230              
2 mean    32.336534863415245 105942.57854821235
3 stddev  9.43908920033797   203550.81474192906
4 min     18                 20000.0           
5 max     110                1000000.0   

library(dbplot)
dbplot_histogram(okc_train, age)

## 看一下两个预测因素之间的关系：alcohol use and drug use.
## 通过sdf_crosstab（）计算列联表：
contingency_tbl <- okc_train %>% 
  sdf_crosstab("drinks", "drugs") %>%
  collect()
```

###
