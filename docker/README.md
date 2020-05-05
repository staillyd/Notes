# docker使用
1. [安装docker](https://docs.docker.com/)
2. 安装[nvidia-docker](https://github.com/NVIDIA/nvidia-docker)
    - 作用:使docker可以使用nvidia显卡
    - 要求:宿主机安装nvidia驱动
3. vscode+docker插件+remote-containers插件
4. [dockerfile](dockerfile)
5. ```dokcer run -rm -it -v ~/Codes:/Codes -v ~/data:/data -p 1234:1001 --gpus all 镜像```