wxPython是Python语言的GUI工具包，作为Python的扩展模块实现，包装了wxWidgets

wxPython 安装
pip3 install -U wxPython
打包 mac 应用
# py2app 安装
pip3 install -U py2app

# 创建 setup.py 文件
py2applet --make-setup helloworld.py

# 创建开发版和测试版的应用
python3 setup.py py2app -A

# 构建发布版应用
rm -rf build dist
python3 setup.py py2app