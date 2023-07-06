# arch linux 安装 anaconda3

anaconda3 的用户指南 <https://docs.conda.io/projects/conda/en/latest/user-guide/index.html>

## 一 安装教程

1. 下载

   下载网址 <https://www.anaconda.com/download#downloads>

   下载对应架构的linux安装包，比如arch linux是x86架构，就选择下载第一个

2. 安装

   官网安装教程<https://docs.anaconda.com/free/anaconda/install/linux/>

   1. 安装依赖——arch linux

      ```shell
      sudo pacman -Sy libxau libxi libxss libxtst libxcursor libxcomposite libxdamage libxfixes libxrandr libxrender mesa-libgl  alsa-lib libglvnd
      ```

   2. 验证完整性

      ```shell
      shasum -a 256 </PATH/FILENAME>
      ```

   3. 进入安装程序

        ```shell
        bash </PATH/FILENAME.sh>
        ```

   4. 查看协议

      1. `Enter`滚动
      2. 输入`yes`同意
   5. 配置安装位置
      1. 默认安装位置是用户目录下
      2. 使用`Enter`确定

      > 建议使用默认路径
   6. 是否初始化`conda`

      选择`yes`执行

3. 配置环境变量

    ```shell
    source <PATH_TO_CONDA>/bin/activate
    conda init
    ```

    > 如果仍然出现`conda not command`
    >
    > 就配置用户配置文件
    >
    > ```shell
    > # arch linux
    > 
    > vim ~/.bashrc
    > ```
    >
    > 添加语句
    >
    > ```vim
    > export PATH=$PATH:/home/leewenpeng/anaconda3/bin
    > ```

4. 验证安装

   ```shell
   conda -V
   ```

## 二 配置

1. 生成配置文件`.condarc`

   1. 使用 `shell` 命令自动生成
      首次运行 `conda config` 命令，会自动在用户目录下创建该文件

      ```shell
      conda config --set show_channel_urls yes
      ```

   2. 直接在用户目录下创建文件

2. 配置修改

   1. 使用命令
      > 该文件遵循YAML语法
      > 使用`conda config --help`查看配置命令完整列表

   2. 修改`.condarc`文件

3. 配置清华镜像

   清华镜像源配置到conda的手册 <https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/>

   1. 使用命令
   2. 修改`.condarc`文件：直接将页面内的代码复制粘贴到`.condarc`文件中

## 三 基础使用教程

1. conda 自身

    ```shell
    # 1. 查看conda版本号
    conda -V

    # 2.更新conda
    conda update conda
    ```

2. 软件包相关

    ```shell
    # 1. 查看包
    conda list
    # 可以使用该命令验证conda是否安装成功

    # 2. 下载包
    conda install <包名>
    # 或者
    pip uninstall <包名>

    # 3. 删除包
    conda uninstall <包名>
    # 或者
    pip uninstall <包名>
    ```

    > + 建议下载包时使用**conda**，因为**conda**会将包的所有相关依赖包给删除;
    >
    > + 删除包的时候使用**pip**,因为**conda**删除包时，会将包依赖的包也一同删除，而**pip**只删除当前包。

3. 虚拟环境相关

    ```shell
    # 1. 查看虚拟环境
    conda env list
    # 或者
    conda info -e

    # 2. 创建虚拟环境
    conda create -n <虚拟环境名> python=x.x

    # 3. 激活虚拟环境
    source activate your_env_nam

    # 4. 退出虚拟环境
    conda deactivate <虚拟环境名>
    
    # 5. 删除虚拟环境
    conda remove -n <虚拟环境名> --all

    # 6. 查看虚拟环境已经安装的包
    pip list
    ```

    > 进入虚拟环境后，对包的下载和删除操作都只影响当前虚拟环境

## 四 conda 安装 pytorch

pytorch 安装网址 <https://pytorch.org/>

pytorch 查找与 `cuda` 对应版本 <https://pytorch.org/get-started/previous-versions/>

conda安装代码

```shell
conda install pytorch torchvision torchaudio pytorch-cuda=<版本号> -c pytorch -c nvidia
```

> 如果电脑上没有安装cuda工具，那么执行命令时，可以忽略掉`pytorch-cuda=x.x`这个条件，因为 anaconda 安装 pytorch 时，会自动根据电脑上显卡驱动，安装对应版本的 `cuda`
>
> （如果按照上述，有时候，会安装成cpu版）
>
> 安装cuda和cudnn看下两节

## 五 conda 安装 cuda

1. 查看源上可用的 `cuda` 版本

   ```shell
   conda search cudatoolkit --info
   ```

2. 复制url到浏览器，下载到本地

3. 本地安装

   ```shell
   conda install --use-local <包路径>
   ```

## 六 conda 安装 cudnn

1. 查看源上与 `cuda` 对应的 `cudnn` 版本

   ```shell
   conda search cudnn --info
   ```

2. 复制url到浏览器，下载到本地

3. 本地安装
