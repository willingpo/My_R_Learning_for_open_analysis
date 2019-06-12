# R语言学习心得（3）：代码执行
包含定时执行.R文件或.Rmd文件
> 参考1：[R Markdown简单指南](http://blog.csdn.net/qq_36080693/article/details/52792031)

> 参考2：[windows下命令行调用R脚本](http://blog.csdn.net/diyiziran/article/details/21379181)

## 一、 命令行执行
### （1）Windows下的CMD命令行执行：
1. 将R.exe所在路径加到环境变量path下，路径一般为C:\Program Files\R\R-3.0.1\bin

2. 在windows 命令行中敲入 调用命令：r CMD BATCH D:\RWORKSPACE\CMD_TEST.R  （注意 CMD BATCH 都要大写）

    命令的普遍形式为R CMD command file，command是别的工具，比如前面用到的批处理工具BATCH
    这个功能相当实用，适合定时调度跑批。比Java的rserve简单太多。
    
3. 在windows 命令行可使用Rscript执行R文件且传参数：Rscript R文件路径 参数1 参数2 参数...  

```
#例子
Rscript D:\my_shinyserver\zhiran\data.R 2018-10-29
#后面的参数用空格隔开
#在R文件中使用args<-commandArgs(TRUE)获取参数值，空格隔开后第一个参数为args[1]，第二个参数为args[2]，以此类推
```


### （2）linux下的shell命令行执行：

1. 保存R脚本，在shell窗口执行Rscript /home/R/MyR/rserveTest.R 
2. php执行R脚本

```
string system ( string $command [, int &$return_var ] )
string exec ( string $command [, array &$output [, int &$return_var ]] )
void passthru ( string $command [, int &$return_var ] )
#注意，system()会将shell命令执行之后，立马显示结果，这一点会比较不方便，因为我们有时候不需要结果立马输出，甚至不需要输出，于是可以用到exec()
```

3. php代码示例

```
<?php
$shell = "Rscript /home/R/MyR/rserveTest.R";
exec($shell, $result, $status);
$shell = "<font color='red'>$shell</font>";
echo "<pre>";
if( $status ){
    echo "shell命令{$shell}执行失败";
} else {
    echo "shell命令{$shell}成功执行, 结果如下<hr>";
    print_r( $result );
}
echo "</pre>";
?>
```



## 二、 R代码执行.Rmd文件输出html

1. ### 执行普通的.Rmd文件
    ```
    library(knitr)
    setwd(<working directory>)
    knit2html("document.Rmd")
    browseURL("document.html")
    ```
2. ### 执行R Markdown v2 document
    ```
    library(knitr)
    setwd(<working directory>)
    rmarkdown::render("document.Rmd")
    browseURL("document.html")
    ```
3. ### 执行R Markdown v2 document 遇到的问题

- 首先是将R.exe所在路径加到环境变量path下，路径一般为C:\Program Files\R\R-3.0.1\bin；
- 其次是将代码保存为Rscript（.R文件），文件若含中文字符，需保存为GB2312的文本编码
- 最后执行中遇到 pandoc version 1.12.3 or higher is required and was not found ，需要在R窗口执行Sys.getenv("RSTUDIO_PANDOC")，获取pandoc的路径，然后在代码里面加上Sys.setenv(RSTUDIO_PANDOC="--- insert directory here ---")，如下所示：

    ```
    library(knitr)
    library(rmarkdown)
    setwd("E:/yuxiaoxin/工作/GitHub/My_R_Learning")
    Sys.setenv(RSTUDIO_PANDOC="C:/Program Files/RStudio/bin/pandoc")
    rmarkdown::render("11-dashboard.Rmd")
    browseURL("11-dashboard.html")
    ```

4. ### 命令行执行.R文件（.Rmd文件）提示：句法分析器24行里不能有多字节字符

    完整错误为：Error in parse(text = x, srcfile = src) : 句法分析器24行里不能有多字节字符
    
    原因分析：主要是.Rmd文件里面有中文字符，文本编码为UTF-8,因此提示错误
    
    解决方法：将.Rmd文件以gb2312编码保存即可
    
    
## 三、 plumber包将R代码转换为web API
> 参考1：[plumber包官网介绍](https://www.rplumber.io/)  
> 参考2：[Securing a dockerized plumber API with SSL and Basic Authentication](https://qunis.de/how-to-make-a-dockerized-plumber-api-secure-with-ssl-and-basic-authentication/)  
> 参考3：[R can API and So Can You!](https://medium.com/tmobile-tech/r-can-api-c184951a24a3)  

```
install.packages("plumber")
library(plumber)
r <- plumb("plumber.R")  # Where 'plumber.R' is the location of the file shown above
r$run(port=8000)
```

运行以上代码后会提示：Starting server to listen on port 8000
Running the swagger UI at http://127.0.0.1:8000/__swagger__/  
然后浏览器访问http://localhost:8000/echo?msg=hello或者http://localhost:8000/plot
- 访问链接的格式是：http://my_host:port/functionname?parameter1=...&parameter2=...
- 疑难点：刚开始一直报R – Plumber Error “404 – Resource Not Found” Solution，就算打开了端口也是不行，后来不知道为啥就可以了，不明白
- 解答上面的疑难点，用plumber官网的plumber.R代码不行，但是换成下面的代码就可以，然后其他也可以了，我也不知道为啥(访问地址：http://127.0.0.1:8000/predict_petal_length?petal_width=1)
```
# load the dataset
dataset <- iris
# create the model
model <- lm(Petal.Length ~ Petal.Width, data = dataset)
#* @get /predict_petal_length
get_predict_length <- function(petal_width){
    # convert the input to a number
    petal_width <- as.numeric(petal_width)
    # create the prediction data frame
    input_data <- data.frame(Petal.Width=as.numeric(petal_width))
    # create the prediction
    predict(model,input_data)
}
```
