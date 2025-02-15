# VSCode 环境配置

## Python's venv

1、在虚拟环境的目录下创建多个不同的虚拟环境
2、在 vscode 进行配置：settings.json
左下角（齿轮）->设置(ctr1+,)->右上角打开 settings.json 文件
3、在最后添加一个虚拟路径的配置项：
"python.venvPath"D:/envs"
4、在 pythonl 的项目中，点击右下角进行 python 解析器的选择

## Docker

1. 下载 docker + dev container 扩展
2. 在 wsl 终端输入`sudo chmod 666 /var/run/docker.sock` 提升权限？
   [https://stackoverflow.com/questions/69530014/failed-to-connect-is-docker-running-vs-code/76817871#76817871?newreg=3557063922d04526b6f5c999b6a66217]
3. 右键选中容器->attach vscode 打开容器->转到 csapp 文件

## Clangd

问题:

1. 在 vscode 中使用 cmake+clangd 时，遇到无法解析新版本 c++支持的头文件（如 optional）, 则在 CMakeLists.txt 中添加如下代码：

   ```cmake
   set(CMAKE_CXX_STANDARD_REQUIRED ON)
   ```

来要求强制使用 `CMAKE_CXX_STANDARD` 中设置的 C++ 版本
