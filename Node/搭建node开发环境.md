##安装node
> 卸载已安装到全局的 node/npm
>安装 nvm 之前最好先删除下已安装的 node 和全局 node 模块
```shell
    npm ls -g --depth=0 # 查看已经安装在全局的模块，以便删除这些全局模块后再按照不同的 node 版本重新进行全局安装
    sudo rm -rf /usr/local/lib/node_modules # 删除全局 node_modules 目录
    sudo rm /usr/local/bin/node # 删除 node
    cd  /usr/local/bin && ls -l | grep "../lib/node_modules/" | awk '{print $9}'| xargs rm # 删除全局 node 模块注册的软链
```

>安装nvm
```shell
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
```

>安装完 nvm 后，输入nvm，当看到有输出时，则 nvm 安装成功。 如果遇到关闭shell后遇到以下提示：

>>-bash: nvm: command not found

>MAC 打开.bash_profile 启动终端Terminal

>>输入cd ~
>>创建.bash_profile
>>输入touch .bash_profile
>>编辑.bash_profile文件
>>输入open -e .bash_profile

>>然后将以下代码复制进去，保存退出
```shell
    export NVM_DIR="$HOME/.nvm"
    [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
```

>>保存文件，关闭.bash_profile
>>更新刚配置的环境变量
>>输入source .bash_profile


>使用nvm
    ```shell
        nvm install stable # 安装最新稳定版 node，现在是 5.0.0
        nvm install 4.2.2 # 安装 4.2.2 版本
        nvm install 0.12.7 # 安装 0.12.7 版本

        # 特别说明：以下模块安装仅供演示说明，并非必须安装模块
        nvm use 4 # 切换至 4.2.2 版本
        npm install -g mz-fis # 安装 mz-fis 模块至全局目录，安装完成的路径是 /Users/<你的用户名>/.nvm/versions/node/v0.12.7/lib/mz-fis
        nvm use 0 # 切换至 0.12.7 版本
        npm install -g react-native-cli #安装 react-native-cli 模块至全局目录，安装完成的路径是 /Users/<你的用户名>/.nvm/versions/node/v4.2.2/lib/react-native-cli

        nvm alias default 0.12.7 #设置默认 node 版本为 0.12.7

        #https://blog.csdn.net/zjuwwj/article/details/72805671 
        #https://www.cnblogs.com/cllgeek/p/6076280.html

    ```
