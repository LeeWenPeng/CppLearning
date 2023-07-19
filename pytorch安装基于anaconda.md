# conda 安装 pytorch

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

## 1 conda 安装 cuda

直接通过 conda 下载安装

```shell
conda install cudatoolkit=<版本号 x.x> -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/linux-64/
```

或者

1. 查看源上可用的 `cuda` 版本

   ```shell
   conda search cudatoolkit --info
   ```

2. 复制url到浏览器，下载到本地

3. 本地安装

   ```shell
   conda install --use-local <包路径>
   ```

## 2 conda 安装 cudnn

直接通过 conda 下载安装

```shell
conda install cudnn=<版本号> -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/linux-64/
```

或者

1. 查看源上与 `cuda` 对应的 `cudnn` 版本

   ```shell
   conda search cudnn --info
   ```

2. 复制url到浏览器，下载到本地

3. 本地安装
