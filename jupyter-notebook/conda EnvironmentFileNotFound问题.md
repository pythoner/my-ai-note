##### 问题背景

我们想创建一个conda环境， 通过`conda env create -n env_name`报错

```
(base) C:\Users\lee>conda env create -n demo

EnvironmentFileNotFound: 'C:\Users\lee\environment.yml' file not found
```



##### 分析

在社区逛了一圈， 发现了类似的问题. 其实本质上还是*conda create* 和*conda env create*两条命令的区别：

* **conda create **  ： 全新创建一个。 

* **conda env create**： 适用于通过*conda env exporter*导出已存在的环境， 然后通过conda env create来导入



##### 解决

使用*conda create *创建环境即可



##### 参考

[conda_issue_13015](https://github.com/conda/conda/issues/13015)

[conda_issue_13600](https://github.com/conda/conda/issues/13600)