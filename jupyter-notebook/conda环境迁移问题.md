##### 问题背景

在windows平台上搭建了conda环境， 想把环境信息导出， 然后通过docker打包， 这样就可以避免每次从头搭建环境。

下面是通过*conda env export -n env_name -f filename*导出得环境信息:

```
# This file may be used to create an environment using:
# $ conda create --name <env> --file <this file>
# platform: win-64
affine=2.4.0=pypi_0
aiohttp=3.9.1=pypi_0
aiosignal=1.3.1=pypi_0
shapely=2.0.2=py310h839b4a8_0
vc14_runtime=14.36.32532=hdcecf7f_17
vs2015_runtime=14.36.32532=h05e6639_17
.....
zstd=1.5.5=h12be248_0

```



下面是Dockerfile文件

```
# 使用官方的Anaconda基础镜像
FROM continuumio/anaconda3

# 设置工作目录
WORKDIR /app

# 拷贝conda 环境信息requirements.txt文件到工作目录中
COPY requirements.txt /app/

# 复制Anaconda环境的配置文件
COPY activate.d /opt/conda/etc/conda/activate.d/
COPY deactivate.d /opt/conda/etc/conda/deactivate.d/

# 设置阿里云为conda的默认频道
RUN conda config --add channels https://mirrors.aliyun.com/anaconda/pkgs/free \
 && conda config --set show_channel_urls yes


# 使用requirements.txt文件创建一个新的Conda环境
RUN conda env create -f /app/requirements.txt -n demo

# 激活环境
RUN conda activate demo

# 其他可能需要的命令来设置环境或安装依赖
# ...

# 声明应用程序监听的端口
EXPOSE 8000

# 启动命令，这里需要根据你的项目实际情况来定
CMD ["conda", "run", "-n", "demo", "python", "/app/main.py"]


```



##### 分析

首先我们来看一下导出得环境信息文件:

```
......
vs2015_runtime=14.36.32532=h05e6639_17
shapely=2.0.2=py310h839b4a8_0
......
zstd=1.5.5=h12be248_0
```

导出得包(以zstd=1.5.5=h12be248_0)包含三个信息： 

* zstd： 包名
* 1.5.5： 包得版本号
* h12be248_0： builder信息

需要注意得是， 这里得builder信息是平台相关得；即通过这个builder信息是可以知道这个包是应用于那个平台得。 我们通过**conda search package==version --info**来验证一下.

```
(base) C:\Users\lee>conda search zstd=1.5.5 --info
Loading channels: done
zstd 1.5.5 h12be248_0
---------------------
file name   : zstd-1.5.5-h12be248_0.conda
name        : zstd
version     : 1.5.5
build       : h12be248_0
build number: 0
size        : 335 KB
license     : BSD-3-Clause
subdir      : win-64
url         : https://conda.anaconda.org/conda-forge/win-64/zstd-1.5.5-h12be248_0.conda
md5         : 792bb5da68bf0a6cac6a6072ecb8dbeb
timestamp   : 2023-08-27 15:53:35 UTC
dependencies:
  - libzlib >=1.2.13,<1.3.0a0
  - ucrt >=10.0.20348.0
  - vc >=14.2,<15
  - vc14_runtime >=14.29.30139


zstd 1.5.5 hd43e919_0
---------------------
file name   : zstd-1.5.5-hd43e919_0.conda
name        : zstd
version     : 1.5.5
build       : hd43e919_0
build number: 0
size        : 682 KB
license     : BSD-3-Clause AND GPL-2.0-or-later
subdir      : win-64
url         : https://repo.anaconda.com/pkgs/main/win-64/zstd-1.5.5-hd43e919_0.conda
md5         : 61b09dfb209667f7558f7c3cd92c1367
timestamp   : 2023-04-12 13:03:58 UTC
dependencies:
  - lz4-c >=1.9.4,<1.10.0a0
  - vc >=14.1,<15.0a0
  - vs2015_runtime >=14.16.27012,<15.0a0
  - xz >=5.2.10,<6.0a0
  - zlib >=1.2.13,<1.3.0a0


zstd 1.5.5 hd43e919_1
---------------------
file name   : zstd-1.5.5-hd43e919_1.conda
name        : zstd
version     : 1.5.5
build       : hd43e919_1
build number: 1
size        : 694 KB
license     : BSD-3-Clause AND GPL-2.0-or-later
subdir      : win-64
url         : https://repo.anaconda.com/pkgs/main/win-64/zstd-1.5.5-hd43e919_1.conda
md5         : 3c0ed4786dab9a9fa22eb8348a57d5ba
timestamp   : 2024-04-30 21:19:21 UTC
dependencies:
  - lz4-c >=1.9.4,<1.10.0a0
  - vc >=14.1,<15.0a0
  - vs2015_runtime >=14.16.27012,<15.0a0
  - xz >=5.4.6,<6.0a0
  - zlib >=1.2.13,<1.3.0a0


zstd 1.5.5 hd43e919_2
---------------------
file name   : zstd-1.5.5-hd43e919_2.conda
name        : zstd
version     : 1.5.5
build       : hd43e919_2
build number: 2
size        : 720 KB
license     : BSD-3-Clause AND GPL-2.0-or-later
subdir      : win-64
url         : https://repo.anaconda.com/pkgs/main/win-64/zstd-1.5.5-hd43e919_2.conda
md5         : d662b0ef9ecc4dfef56cfae98a2bb7db
timestamp   : 2024-05-02 19:46:37 UTC
dependencies:
  - lz4-c >=1.9.4,<1.10.0a0
  - vc >=14.1,<15.0a0
  - vs2015_runtime >=14.16.27012,<15.0a0
  - xz >=5.4.6,<6.0a0
  - zlib >=1.2.13,<1.3.0a0



(base) C:\Users\lee>
```

通过上面得sub-dir， 我们发现*zstd-1.5.5-hd43e919_0.conda*是windows平台的。 也就是说这个只是在windows平台下运行。 如果我们docker的基础镜像是linux的话，就会有异常。 

> 关于后面的builder信息， 这里多说两句： 
>
> 1. 包的信息可以通过conda-forge(https://conda-forge.org/packages/)进行查看， 这里也可以查到对应的包对应的平台信息
> 2. builder信息中的py310h839b4a8_0， 表示绑定的是python的3.10版本， 那么conda这里绑定的python就需要是3.10版本
> 3. builder信息中的pypi_0， 表示这是一个pypi包。 在目前我安装的conda环境信息中， 对于这一类的包是有专门的pip子级来提示。 后面解决部分看详细输出



问题2：创建环境的时候报 *PackagesNotFoundError:*

```
(base) [root@k8s ~]# conda env create -n demo -f ./requirements.txt
Channels:
 - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
 - defaults
Platform: linux-64
Collecting package metadata (repodata.json): done
Solving environment: failed

PackagesNotFoundError: The following packages are not available from current channels:

  - zstd==1.5.5=h12be248_0
  - zipp==3.17.0=pyhd8ed1ab_0
  - zict==3.0.0=pyhd8ed1ab_0
  - zeromq==4.3.5=h63175ca_0
  - yarl==1.9.4=pypi_0
  - yarg==0.1.9=pypi_0
  - yaml==0.2.5=h8ffe710_2
  - xz==5.2.6=h8d14728_0
  - xyzservices==2023.10.1=pyhd8ed1ab_0

  ......
  ......
  - aiohttp==3.9.1=pypi_0
  - affine==2.4.0=pypi_0

Current channels:

  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/linux-64
  - https://repo.anaconda.com/pkgs/main/linux-64
  - https://repo.anaconda.com/pkgs/r/linux-64

To search for alternate channels that may provide the conda package you're
looking for, navigate to

    https://anaconda.org

and use the search bar at the top of the page.

```

通过*conda config --show channels*查看目前的channels

```
base) [root@k8s ~]# conda config --show channels
channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
  - defaults
```

这里的channels只有default和清华的源。 而有一些包其实是在社区的 *conda-forge* channel中，  我们这里通过*conda config --add channels conda-forge* 来解决.

``` 
(base) [root@k8s ~]# conda config --add channels conda-forge
(base) [root@k8s ~]# 
(base) [root@k8s ~]# conda config --show channels
channels:
  - conda-forge
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
  - defaults

```



问题3： 在我们安装的conda-forge channel后， 依然会报*PackagesNotFoundError* 报错

```
(base) C:\Users\lee>conda env create -f ./requirements.txt -n demo
Collecting package metadata (repodata.json): done
Solving environment: failed

ResolvePackageNotFound:
  - jsonschema-specifications==2023.12.1=pypi_0
  - nbclient==0.10.0=pypi_0
  - blosc==1.11.1=pypi_0
  - pipreqs==0.5.0=pypi_0
  - pycryptodome==3.20.0=pypi_0
  - python-utils==3.8.1=pypi_0
  - httplib2==0.22.0=pypi_0
  - async-timeout==4.0.3=pypi_0
  - slicer==0.0.7=pypi_0
  - pandas==2.2.1=pypi_0
  - asyncio==3.4.3=pypi_0
  - crypto==1.4.1=pypi_0
  - charset-normalizer==3.3.2=pypi_0
  - cligj==0.7.2=pypi_0
  - itsdangerous==2.1.2=pypi_0
  - rpds-py==0.18.1=pypi_0
  - flask==3.0.0=pypi_0
  - requests==2.31.0=pypi_0
  - geopandas==0.14.0=pypi_0
  - threadpoolctl==3.2.0=pypi_0
  - tabulate==0.9.0=pypi_0
  - yarg==0.1.9=pypi_0
  - naked==0.1.32=pypi_0
  - joblib==1.3.2=pypi_0
  - regionmask==0.11.0=pypi_0
  - blinker==1.7.0=pypi_0
  - fastjsonschema==2.19.1=pypi_0
  - docopt==0.6.2=pypi_0
  - mysql-connector-python==8.3.0=pypi_0
  - pooch==1.8.0=pypi_0
  - attrs==23.1.0=pypi_0
  - jsonschema==4.22.0=pypi_0
  - statsmodels==0.14.0=pypi_0
  - backcall==0.2.0=pypi_0
  - dill==0.3.7=pypi_0
  - affine==2.4.0=pypi_0
  - jupyterlab-pygments==0.3.0=pypi_0
  - cmaps==2.0.1=pypi_0
  - progressbar2==4.2.0=pypi_0
  - fiona==1.9.5=pypi_0
  - et-xmlfile==1.1.0=pypi_0
  - pygam==0.9.0=pypi_0
  - meteva==1.7.5=pypi_0
  - aiohttp==3.9.1=pypi_0
  - numba==0.58.1=pypi_0
  - multidict==6.0.4=pypi_0
  - mistune==3.0.2=pypi_0
  - beautifulsoup4==4.12.3=pypi_0
  - ndindex==1.7=pypi_0
  - click-plugins==1.1.1=pypi_0
  - netcdf4==1.6.5=pypi_0
  - defusedxml==0.7.1=pypi_0
  - scipy==1.11.3=pypi_0
  - tzdata==2024.1=pypi_0
  - seaborn==0.13.0=pypi_0
  - yarl==1.9.4=pypi_0
  - numpy==1.26.4=pypi_0
  - pymannkendall==1.4.3=pypi_0
  - tqdm==4.66.1=pypi_0
  - nbconvert==7.16.4=pypi_0
  - tables==3.9.1=pypi_0
  - py-cpuinfo==9.0.0=pypi_0
  - descartes==1.1.0=pypi_0
  - metpy==1.5.1=pypi_0
  - gdal==3.4.3=pypi_0
  - patsy==0.5.3=pypi_0
  - shap==0.43.0=pypi_0
  - openpyxl==3.1.2=pypi_0
  - xarray==2023.10.1=pypi_0
  - shellescape==3.8.1=pypi_0
  - certifi==2023.11.17=pypi_0
  - webencodings==0.5.1=pypi_0
  - rasterio==1.3.9=pypi_0
  - tinycss2==1.3.0=pypi_0
  - panda==0.3.1=pypi_0
  - nbformat==5.10.4=pypi_0
  - pymysql==1.1.0=pypi_0
  - referencing==0.35.1=pypi_0
  - wheel==0.41.2=pypi_0
  - llvmlite==0.41.1=pypi_0
  - greenlet==3.0.3=pypi_0
  - numexpr==2.8.7=pypi_0
  - eli5==0.13.0=pypi_0
  - bleach==6.1.0=pypi_0
  - pytz==2024.1=pypi_0
  - pint==0.22=pypi_0
  - scikit-learn==1.3.2=pypi_0
  - blosc2==2.3.1=pypi_0
  - sqlalchemy==2.0.29=pypi_0
  - aiosignal==1.3.1=pypi_0
  - protobuf==3.19.6=pypi_0
  - soupsieve==2.5=pypi_0
  - werkzeug==3.0.1=pypi_0
  - frozenlist==1.4.0=pypi_0
  - cftime==1.6.3=pypi_0
  - idna==3.6=pypi_0
  - urllib3==2.1.0=pypi_0
  - miceforest==5.6.3=pypi_0
  - setuptools==68.1.2=pypi_0
  - pandocfilters==1.5.1=pypi_0
  - python-graphviz==0.20.1=pypi_0
  - lightgbm==4.1.0=pypi_0
  - snuggs==1.4.7=pypi_0

(base) C:\Users\lee>conda --version
conda 23.7.4
```

通过包的信息， 我们发现， 这里缺失的pypi的包， 这是推测我们使用较老的conda版本进行导出时没有进行pip包的区分。  所以我们需要把这部分包，单独摘出来， 后续通过pip进行安装。 。



##### 总结:

目前的环境迁移过程存在以下几个问题：

1. 导出的conda环境信息中包含builder的信息， 这可能与docker基础镜像的架构不一样。 如一个windows， 一个Linux， 导致构建环境失败。 
2. 缺少 *conda-forge* channel， 导致包找不到
3. 目前的版本导出的包有两类， conda包和pip包， 对于pip包(pypi_o后缀的)我们需要摘出， 在其他包装完后， 在通过*pip install*进行安装
4. 包跟python的版本绑定， 所以我们这里在构建环境的时候指定python版本



##### 解决

1. 对于包含builder信息， 我们在导出环境信息的时候， 通过*--no-builds*参数来指定不包含builder信息。

   ```
   (base) [root@k8s ~]# conda env export -n base -f conda_environment_base.yaml --no-builds
   (base) [root@k8s ~]# cat conda_environment_base.yaml
   name: base
   channels:
     - conda-forge
     - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
     - defaults
   dependencies:
     - _anaconda_depends=2024.02
     - _libgcc_mutex=0.1
     - _openmp_mutex=4.5
     - abseil-cpp=20211102.0
     - aiobotocore=2.7.0
      ......
     - zlib=1.2.13
     - zlib-ng=2.0.7
     - zope=1.0
     - zope.interface=5.4.0
     - zstandard=0.19.0
     - zstd=1.5.5
     - pip:
         - nvidia-cublas-cu12==12.1.3.1
         - nvidia-cuda-cupti-cu12==12.1.105
         - nvidia-cuda-nvrtc-cu12==12.1.105
         - nvidia-cuda-runtime-cu12==12.1.105
         - nvidia-cudnn-cu12==8.9.2.26
         - nvidia-cufft-cu12==11.0.2.54
         - nvidia-curand-cu12==10.3.2.106
         - nvidia-cusolver-cu12==11.4.5.107
         - nvidia-cusparse-cu12==12.1.0.106
         - nvidia-nccl-cu12==2.20.5
         - nvidia-nvjitlink-cu12==12.4.127
         - nvidia-nvtx-cu12==12.1.105
         - torch==2.3.0
         - triton==2.3.0
   prefix: /root/anaconda3
   ```

   > 这里的pypi的包，就使用在导出的环境信息中. 这样就不会存在上述的问题3,  对应的conda version是(base) 
   > conda 24.1.2

   2. 在docker文件中添加 *conda-forge* channel

   3. 创建环境的时候指定python的版本信息

      ```
      conda env create -f filename -n environment_name python=3.10
      ```

      
