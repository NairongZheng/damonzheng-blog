# gitbook and github

参考链接：[https://einverne.github.io/gitbook-tutorial/](https://einverne.github.io/gitbook-tutorial/)

## 安装

### 安装npm与nodejs

参考[链接](https://blog.csdn.net/HuangsTing/article/details/113857145)安装nodejs等，这个好用。可以进行版本控制。

nvm官方下载，建议直接默认路径，不然好像环境路径会有问题。

```bash
# 安装完后查看版本号
nvm version
# 查看可以安装版本号
nvm list available
# 选择需要的版本进行安装
nvm install 10.21.0
# 查看已经安装了哪些版本（可以切换的）
nvm lsit
# 选择安装好的版本使用
nvm use 10.21.0
# 查看nodejs版本
node -v
# 查看npm版本
npm -v
# 都有的话说明安装成功
```

用上面方法安装的npm跟nodejs可以进行版本控制，在安装gitbook的时候版本不行还可以切换，使用别的需要nodejs的应用也可以用这个，所以比较方便。

### 安装gitbook

```bash
# 安装gitbook
npm install gitbook-cli -g
# 查看安装的gitbook的版本（就是这一步安装gitbook很容易出错）
gitbook -V
# 安装成功显示如下
# C:\Users\Administrator>gitbook -V
# CLI version: 2.3.2
# GitBook version: 3.2.3
```

我在安装gitbook的时候就出现了上面提到的版本问题，可以参考[链接1](https://blog.csdn.net/weixin\_42349568/article/details/108414441)跟[链接2](https://blog.csdn.net/Lowerce/article/details/107579261)。





