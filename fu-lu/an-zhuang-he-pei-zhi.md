## 安装和配置<a id="orgheadline4"></a>

### ubuntu下

就是利用pip3 安装之：
```
pip3 install pyqt5
```


其中安装Qt5不需要我们多费心，请确保下面几个软件包安装上去了（参考了 [这个网页](http://askubuntu.com/questions/508503/whats-the-development-package-for-qt5-in-14-04) 和 [这个网页](http://askubuntu.com/questions/609238/error-pyqt5-requires-qt-v5-0-or-later) ）:

sudo apt-get install qt5-default
sudo apt-get install qtbase5-dev
sudo apt-get install qtdeclarative5-dev

上面的第三个可能并不需要安装，其中第一个 `qt5-default` 和qmake的v5版本有关，然后 `qtbase5-dev` 肯定是需要安装的。不管怎么说，确保这些都装上吧。

然后再在 [PyQt的官网](https://www.riverbankcomputing.com/news) 下载 `SIP` 的源码，运行 `python3 configure.py` 输出makefile，后面就是大家熟悉的 `make` , `sudo make install` 。

继续再下载 `PyQt5` 的源码，安装步骤同上，这里就不赘述了。

在后面我们会提到 `pyuic5` 和 `pyrcc5` 命令，其在下面这个软件包里面:

```sh
sudo apt-get install pyqt5-dev-tools
```

检查pyqt5安装情况执行以下脚本即可，显示的是当前安装的pyqt5的版本号:

>>> from PyQt5.QtCore import QT_VERSION_STR
>>> print(QT_VERSION_STR)
5.2.1

本文的代码都是PyQt版本号都是上面的，没有特别的理由，会一直维持在这个版本号里面了。
