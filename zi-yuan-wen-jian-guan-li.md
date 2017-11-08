# 资源文件管理<a id="orgheadline26"></a>

## 资源管理<a id="orgheadline25"></a>

pyqt都用qrc文件来管理软件内部的资源文件（如图标文件，翻译文件等）。qrc文件的编写格式如下：

```xml
&lt;!DOCTYPE RCC&gt;&lt;RCC version="1.0"&gt;
&lt;qresource&gt;
&lt;file&gt;images/copy.png&lt;/file&gt;
&lt;/qresource&gt;
&lt;/RCC&gt;
```

qrc的编写还是很简单的，完全可以手工编写之。上面代码第三行的images/copy.png的意思就是qrc文件所在目录下的images文件夹，里面的copy.png文件。

qrc文件编写好了你需要运行如下命令

```sh
pyrcc5 wise.qrc -o wise_rc.py
```

这样将会输出一个wise\_rc.py文件，你如果要使用里面的资源，首先

```python
import wise_rc
```

然后引用路径如下 ' `:/images/copy.png` ' ，这样就可以使用该图标文件了。

上面是pyqt5的情况，对于pyqt4类似的有：

```sh
pyrcc4 wise.qrc -o wise_rc.py
```

值得一提的是pyrcc4还有一个额外的选项 `-py3` ，用于生成python3的代码。

推荐一个项目里面所有的资源文件都用一个qrc文件来管理。