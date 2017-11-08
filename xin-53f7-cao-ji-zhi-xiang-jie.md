# 信号－槽详解<a id="orgheadline23"></a>

考虑到pyqt5只支持新式信号－槽机制了，这里将新式信号－槽机制详细说明，由于新式信号－槽机制在pyqt4上也能正常运行，所以新来的程序员推荐都用新式信号－槽机制。下面只介绍新式的信号－槽机制。

信号(singal)可以连接无数多个槽(slot)，或者没有连接槽也没有问题，信号也可以连接其他的信号。正如前面所述，连接的基本语句形式如下： `who.singal.connect(slot)` 。比如说按钮最常见的内置信号 `triggered` ，而槽实际上就是某个函数，比如主窗体的 `self.close` 方法。

信号就是 `QObject` 的一个属性，pyqt的窗体有很多内置信号，你也可以定义自己的信号，这个后面再提及。信号还没和槽连接起来就只是一个属性，只有通过 `connect` 方法连接起来，信号－槽机制就建立起来了。类似的信号还有 `disconnect` 方法和 `emit` 方法。disconnect就是断开信号－槽机制，而emit就是激活那个信号。

pyqt很多内置信号和内置槽将GUI的事件驱动细节给隐藏了，如果你自己定义自己的信号或者槽可能对who.singal.connect(slot)这样简洁的形式如何完成工作的感到困惑。这里先简要地介绍一下。

信号都是类的一个属性，新的信号必须继承自QObject，然后由 `PyQt5.QtCore.pyqtSingal` （在pyqt4下是PyQt4.QtCore.pyqtSingal。）方法创建，这个方法接受的参数中最重要的是types类型，比如int，bool之类的，你可以认为这是信号传递的参数类型，但实际传递这些参数值的是emit方法。然后槽实际上就是经过特殊封装的函数，这些函数当然需要接受一些参数或者不接受参数，而这些参数具体的值传进来的是由emit方法执行的，然后我们通过who.singal.connect(slot)这样的形式将某个信号和某个槽连接起来，who的信号，然后信号类自带的连接方法，然后连接到slot某个函数上，在这里隐藏的一个重要细节就是emit方法，比如说你定义一个新的信号，需要将点击屏幕的具体x,y坐标发送出去，内置的信号－槽将这一机制都完成了，如果你自己定义的信号和槽的话，比如 `pyqtSingal(int,int)` ，发送给func(x,y)，具体x和y的值你需要通过emit(x,y)来发送。至于什么时候发送，已经发送的x,y值的获取，这应该又是另外一个信号－槽机制的细节。

请看下面这个例子：

```python
import sys
from PyQt5.QtWidgets import QHBoxLayout,QSlider,QSpinBox,QApplication,QWidget
from PyQt5.QtCore import Qt

app = QApplication(sys.argv)
window = QWidget()
window.setWindowTitle("enter your age")
spinBox = QSpinBox()
slider = QSlider(Qt.Horizontal)
spinBox.setRange(0,130)
slider.setRange(0,130)

spinBox.valueChanged.connect(slider.setValue)
slider.valueChanged.connect(spinBox.setValue)

spinBox.setValue(35)

layout = QHBoxLayout()
layout.addWidget(spinBox)
layout.addWidget(slider)


window.setLayout(layout)
window.show()

sys.exit(app.exec_())
```

第16行将spinBox的 `valueChanged` 信号和slider的 `setValue` 槽连接起来了，其中QSpinBox内置的 `valueChanged` 信号发射自带的一个参数就是改变后的值，这个值传递给了QSlider的内置槽 `setValue` ，从而将slider的值设置为新值。第17行如果slider的值发生了改变，那么会发送valueChanged信号，然后又传递给了spinBox，并执行了内置槽setValue，由于此时的值即为原值，这样spinBox内的值就没有发生改变了，如此程序不会陷入死循环。

## 自定义信号<a id="orgheadline19"></a>

正如前所述及自定义信号由PyQt5.QtCore.pyqtSingal（在pyqt4下是PyQt4.QtCore.pyqtSingal。）方法创建，具体格式如下：

```python
from PyQt5.QtCore import QObject, pyqtSignal

class Foo(QObject):
closed = pyqtSignal()
range_changed = pyqtSignal(int, int, name='rangeChanged')
```

上面Foo类里面自定义了一个新的信号，它必须是GObject的子类。然后定义了一个closed信号，没有接受任何参数。下面是range\_changed信号，接受了一个int和一个int类型，然后这个信号的名字是rangeChanged，name选项是一个可选项，如果不填那么信号的名字就是range\_changed。

信号还可以overload，不过似乎不太适合python。

注意信号必须定义为类的属性，同时必须是GObject的子类。

## 自定义槽<a id="orgheadline20"></a>

按照python格式自己定义的函数就是所谓的自定义槽了。不过推荐用pyqt的槽装饰器来定义槽。

```python
from PyQt4.QtCore import pyqtSlot
#1
@pyqtSlot()
def foo(self):
pass
#2
@pyqtSlot(int, str)
def foo(self, arg1, arg2):
pass
#3
@pyqtSlot(int, name='bar')
def foo(self, arg1):
pass
#4
@pyqtSlot(int, result=int)
def foo(self, arg1):
pass
#5
@pyqtSlot(int, QObject)
def foo(self, arg1):
pass
```

上面的第一个例子定义了名叫foo的一个槽，然后不接受任何参数。第二个槽接受一个int类型的值和str类型的值。第三个槽名字叫做bar，接受一个int类型的值，第四个槽接受一个int类型的值，然后返回的是一个int类型的值，第五个操作接受一个int类型的值和一个GObject类型的值，此处应该暗指其他pyqt窗体类型都可以作为参数进行传递。

```python
@pyqtSlot(int)
@pyqtSlot('QString')
def valueChanged(self, value):
pass
```

这里定义了两个槽，名字都叫做valueChanged，一个接受int类型，一个接受QString类型，同前面信号的overload一样，在python中不推荐这么使用，还是明晰一点比较好。

## 发射信号<a id="orgheadline21"></a>

信号对象有emit方法用来发射信号，然后信号对象还有disconnect方法断开某个信号和槽的连接。

一个信号可以连接多个槽，多个信号可以连接同一个槽，一个信号可以与另外一个信号相连接。

下面通过一个例子详解自建信号还有自建槽并建立发射机制的情况。

```python
from PyQt5.QtWidgets import QDialog,QLabel,QLineEdit,QCheckBox,QPushButton,QHBoxLayout,QVBoxLayout,QApplication
from PyQt5.QtCore import Qt ,pyqtSignal,QObject,pyqtSlot


class FindDialog(QDialog):
findNext = pyqtSignal(str,Qt.CaseSensitivity)
findPrevious = pyqtSignal(str,Qt.CaseSensitivity)

def __init__(self,parent=None):
super().__init__(parent)
label = QLabel(self.tr("Find &amp;what:"))
self.lineEdit = QLineEdit()
label.setBuddy(self.lineEdit)

self.caseCheckBox=QCheckBox(self.tr("Match &amp;case"))
self.backwardCheckBox=QCheckBox(self.tr("Search &amp;backward"))
self.findButton = QPushButton(self.tr("&amp;Find"))
self.findButton.setDefault(True)
self.findButton.setEnabled(False)
closeButton=QPushButton(self.tr("Close"))

self.lineEdit.textChanged.connect(self.enableFindButton)
self.findButton.clicked.connect(self.findClicked)
closeButton.clicked.connect(self.close)

topLeftLayout=QHBoxLayout()
topLeftLayout.addWidget(label)
topLeftLayout.addWidget(self.lineEdit)
leftLayout=QVBoxLayout()
leftLayout.addLayout(topLeftLayout)
leftLayout.addWidget(self.caseCheckBox)
leftLayout.addWidget(self.backwardCheckBox)
rightLayout = QVBoxLayout()
rightLayout.addWidget(self.findButton)
rightLayout.addWidget(closeButton)
rightLayout.addStretch()
mainLayout=QHBoxLayout()
mainLayout.addLayout(leftLayout)
mainLayout.addLayout(rightLayout)
self.setLayout(mainLayout)

self.setWindowTitle(self.tr("Find"))
self.setFixedHeight(self.sizeHint().height())


def enableFindButton(self,text):
self.findButton.setEnabled(bool(text))
@pyqtSlot()
def findClicked(self):
text = self.lineEdit.text()
if self.caseCheckBox.isChecked():
cs=Qt.CaseSensitive
else:
cs=Qt.CaseInsensitive

if self.backwardCheckBox.isChecked():
self.findPrevious.emit(text,cs)
else:
self.findNext.emit(text,cs)



if __name__ == '__main__':
import sys
app=QApplication(sys.argv)
findDialog = FindDialog()
def find(text,cs):
print('find:',text,'cs',cs)
def findp(text,cs):
print('findp:',text,'cs',cs)

findDialog.findNext.connect(find)
findDialog.findPrevious.connect(findp)
findDialog.show()
sys.exit(app.exec_())
```

首先自建的信号必须是类的属性，然后这个类必须是QObject的子类，这里QDialog是继承自QObject的。请看到第9行和第10行，通过pyqtSignal函数来自建信号，此信号有两个参数，一个是str字符变量，一个是Qt.CaseSensitivity的枚举值。假设我们输入一些文字了，然后点击Find按钮，请看到第26行，点击之后将执行findClicked槽，按钮的clicked信号是不带参数的。所以后面定义的findClicked槽（简单的函数也可以）也没有任何参数。

findClicked槽的53-57行确定了当前的QLineEdit的text值和cs也就是大小写是否检查的状态。然后根据向前或者向后是否勾选来确定接下来要发送的信号。比如findNext信号调用emit方法，对应两个参数也传递过去了。而这个findNext正是我们前面自定义的信号，正是对应的两个参数类型。

我们再看到这里简单做了一个测试程序，70-73行定义了两个简单的函数，然后75，76行将findDialog的这两个信号和上面两个函数连接起来。于是当我们点击Find按钮，首先执行findClicked槽，然后假设这里发送了findNext信号（附带两个参数），然后信号又和find函数相连（参数传递给了find函数），然后执行find函数。整个过程就是这样的。

## 信号－槽机制的反思<a id="orgheadline22"></a>

在接下来Qt designer这一章也会详细讨论这个问题，我们使用Qt designer来设计和修改ui文件——对应程序中大部分的静态视图元素，主要的目的倒不是为了快速GUI程序编写，其实写代码也挺快的，主要的目的就是为了代码复用。当我们养成习惯，强迫自己程序中的静态视图元素都进入ui文件，这不仅增强了ui文件的复用性，而且也增强了剩下来的python代码的复用性。这其中很大一部分就是这里讨论的信号－槽机制的功劳。

当我们自定义的类加载好ui文件之后，该类里面的代码实际上就剩下两个工作：

1. 把本窗体的信号和槽都编写好
2. 把母窗体和子窗体和信号－槽接口写好。

一般程序的用户互动接口大多在最顶层，也就是用户一般喜欢在菜单栏找到所有可能对程序的控制，这些控制的实现函数如果放在都放在母窗体，那么整个程序的代码复用性会降到最低，而如果我们将这些实现函数分别移到和其视图窗体最紧密的窗体类中，那么不仅代码复用性会大大提高，而且这些槽或函数的编写也会简单很多。那么我们该如何组织这些信号和槽（实现函数）呢？我在这里提出组织学上的一些抽象原则：

1. 最小组织原则，凡是小组织能够自我实现的功能绝不上传到更大一级的组织中去。
2. 大组织对小组织元素的某些实现的引用，采用明文引用原则。比如说母窗体中有一个小窗体有一个编辑器，母窗体想要操控这个编辑器执行剪切操作，那么采用明文引用，也就是self.textEdit.cut。
3. 小组织对大组织属性的引用采用信号激活原则，比如说某个编辑器发生了内容修改，你可以自定义一个信号，该信号为标题修改信号，然后信号触发母窗体的某个方法，这样达到修改母窗体的标题的目的。而在母窗体中，只需要在声明是将小组织的信号和大组织的某个方法连接起来即可。

# 使用Qt designer<a id="orgheadline24"></a>

其实我们不一定要使用Qt designer，Qt desinger的目的主要不是为了快速绘制GUI，而是一种模块化编程思路。利用Qt designer在代码复用上ui文件只是很小的一部分，关键是将ui抽离之后，剩下的py文件里面定义的大多是信号和槽，其中槽就是函数，这些函数复用性是很高的。而对于不同的程序最大差异化的不分就是不同的信号和信号与槽之间的连接了。下面将通过一个timer计时器小程序简单演示下如何利用Qt designer快速Qt编程。


# 布局管理<a id="orgheadline38"></a>

布局管理是GUI设计中不可回避的一个话题，这里详细讨论下pyqt的布局管理。正如前所述及，pyqt5用于布局管理的类都移到了QtWidgets子模块那里了，首先是最基本的 `QHBoxLayout` 和 `QVBoxLayout` 。

## QBoxLayout<a id="orgheadline34"></a>

`QHBoxLayout` 和 `QVBoxLayout` 一个是横向排布，一个是竖向排布。它们的使用方法如下所示：

mainLayout=QHBoxLayout()
mainLayout.addWidget(button1)
mainLayout.addWidget(button2)
self.setLayout(mainLayout)

Layout对象就好像一个封装器，Layout里面还可以有Layout，当然还有其他一些窗体子单元，都通过 `addWidget` 方法来确立封装关系。最后主母窗口主要接受一个Layout对象，使用的是 `setLayout` 方法。

```python
from PyQt5.QtWidgets import QApplication,QWidget,QLabel,QVBoxLayout,QPushButton,QLineEdit, QMessageBox

class Form(QWidget):
def __init__(self):
super().__init__()
nameLabel = QLabel("Name:")
self.nameLine = QLineEdit()
self.submitButton = QPushButton("Submit")
bodyLayout = QVBoxLayout()
bodyLayout.addWidget(nameLabel)
bodyLayout.addWidget(self.nameLine)
bodyLayout.addWidget(self.submitButton)

self.submitButton.clicked.connect(self.submit)

self.setLayout(bodyLayout)
self.setWindowTitle("Hello Qt")
self.show()

def submit(self):
name = self.nameLine.text()

if name == "":
QMessageBox.information(self, "Empty Field",
"Please enter a name.")
return
else:
QMessageBox.information(self, "Success!",
"Hello %s!" % name)

if __name__ == '__main__':
import sys
app = QApplication(sys.argv)
screen = Form()
sys.exit(app.exec_())
```

pyqt4版本就是把头引入语句改成

from PyQt4.QtGui import ...

## addStretch方法<a id="orgheadline35"></a>

插入一个分隔符，也就是设计器里面的弹簧。

## QGridLayout<a id="orgheadline36"></a>

在tkinter中有个grid方法，也就是网格布局，同样pyqt中也有个网格布局对象QGridLayout。QGridLayout的用法和上面QBoxLayout类似，除了 **addWidget** 方法后面还可以接受两个额外的参数表示几行几列。

请看到下面的例子。这个例子很好地演示了QGridLayout的使用。其中 `(i-1)//3` 即该数对3取商，本来的1 2 3 4 5 6&#x2026;将变成0 0 0 1 1 1 2 2 2&#x2026;正好对应网格中的几行，而 `(i-1)%3` 即该数对3取余，本来的1 2 3 4 5 6&#x2026;将变成0 1 2 0 1 2 0 1 2&#x2026;正好对应网格中的几列的概念。

```python
from PyQt5.QtWidgets import QApplication,QWidget,QPushButton,QGridLayout

class Form(QWidget):
def __init__(self):
super().__init__()
bodyLayout = QGridLayout()
for i in range(1,10):
button = QPushButton(str(i))
bodyLayout.addWidget(button,(i-1)//3,(i-1)%3)
print(i,(i-1)//3,(i-1)%3)
self.setLayout(bodyLayout)
self.setWindowTitle("the grid layout")
self.show()


if __name__ == '__main__':
import sys
app = QApplication(sys.argv)
screen = Form()
sys.exit(app.exec_())
```

## QFormLayout<a id="orgheadline37"></a>

QFormLayout，表单布局，常用于提交某个配置信息的表单。

请看到下面的例子。这个例子来自pyqt5源码examples文件夹layouts文件夹里面的basiclayouts.py文件，做了简化主要用于演示表单布局。

```python
from PyQt5.QtWidgets import (QApplication, QDialog,QDialogButtonBox, QFormLayout, QGroupBox,QLabel, QLineEdit, QSpinBox,QVBoxLayout,QTextEdit)


class Dialog(QDialog):
def __init__(self):
super().__init__()
self.createFormGroupBox()
buttonBox = QDialogButtonBox(QDialogButtonBox.Ok | QDialogButtonBox.Cancel)
buttonBox.accepted.connect(self.accept)
buttonBox.rejected.connect(self.reject)
mainLayout = QVBoxLayout()
mainLayout.addWidget(self.formGroupBox)
mainLayout.addWidget(buttonBox)
self.setLayout(mainLayout)
self.setWindowTitle("user info")

def createFormGroupBox(self):
self.formGroupBox = QGroupBox("your infomation")
layout = QFormLayout()
layout.addRow(QLabel("name:"), QLineEdit())
layout.addRow("age:", QSpinBox())
layout.addRow(QLabel("other infomation:"), QTextEdit())
self.formGroupBox.setLayout(layout)


if __name__ == '__main__':
import sys
app = QApplication(sys.argv)
dialog = Dialog()
sys.exit(dialog.exec_())
```

这里 `QDialog` 类和 `QDialogButtonBox` 类我们且不去管他，QDialog类和下面的accept和reject方法有关，而QDialogButtonBox和最下面的两个按钮和绑定的喜好accepted和rejected有关。

然后我们看到下面创建表单的那个函数，其中 `QGroupBox` 也是一个窗体类型，带有标题。接下来就是QFormLayout表单布局的核心代码：

layout = QFormLayout()
layout.addRow(QLabel("name:"), QLineEdit())
layout.addRow(QLabel("age:"), QSpinBox())
layout.addRow(QLabel("other infomation:"), QTextEdit())
self.formGroupBox.setLayout(layout)

我们看到前面的layout的创建和后面母窗体使用本layout的 `setLayout` 方法和前面两个布局都是类似的，除了表单布局是一行行的，它的方法不是addWidget，而是 `addRow` ，然后addRow方法严格意义上可以接受两个窗体类型（包括layout类型）， <span class="underline">另外第一个参数还可以是字符串，即显示的文字</span> 。

# 快捷键和Tab键管理<a id="orgheadline42"></a>

## 什么是伙伴关系<a id="orgheadline39"></a>

一般是通过QLabel的setBuddy方法来关联某个输入窗体。然后QLabel有一个快捷键，当你按下这个快捷键，输入焦点就会转到这个QLabel对应的伙伴输入窗体上。

## 快捷键<a id="orgheadline40"></a>

QShortcut类

文本前用&会引入对应的Alt+w之类的快捷键。

然后QAction在初始化的时候有

然后QAction有方法

## QKeySequence<a id="orgheadline41"></a>

`QKeySequence` 类在pyqt4和pyqt5中来自QtGui子模块，是快捷键的解决方案。比如可以直接引用 `QKeySequence.Open` 来表示快捷键Ctrl+O。可用的构造函数如下所示：

QKeySequence(QKeySequence.Print)
QKeySequence(tr("Ctrl+P"))
QKeySequence(tr("Ctrl+p"))
QKeySequence(Qt.CTRL + Qt.Key_P)

我不太喜欢第一种表达方式，不是任何软件都有打印操作，况且打印和某个快捷键之间并没有逻辑联系，只有程序员的个人使用经验，这是不小的记忆负担。我比较喜欢第四种写法，看上去意义更加清晰，Qt来自QtCore子模块。

字母按键就是类似 `Qt.Key_W` 这样的形式，Shift按键是Qt.SHIFT，Meta按键是Qt.META，CTRL按键是 `Qt.CTRL` ，ALT按键是 `Qt.ALT` 。

# 国际化支持<a id="orgheadline45"></a>

本小节参考资料除了官方文档之外还有[这个网站](http://plashless.wordpress.com/2014/02/01/internationalizing-python-pyqt-apps/) 。

这里指的pyqt的软件国际化支持主要是指i18n，也就是两种语言，英语和本土语言。其中软件的字符串都是英语，然后用 `self.tr()` 封装。

然后在你的项目里新建一个translations文件夹，新建如下一个小文件 `wise.pro` ，这里的wise是你的模块具体的名字，随意修改之。这个文件的内容简要如下：

SOURCES += ../main.py ../__init__.py \
../Widgets/__init__.py
TRANSLATIONS += wise_zh_CN.ts

SOURCES 是你希望扫描的py文件，如果该文件有前面所说的self.tr()封装，那么里面的字符串pylupdate5工具就可以扫描出来。这里支持路径的相对表达。但是不支持glob语法。

第二个变量就是TRANSLATIONS就是你希望生成的目标翻译ts文件的文件名，一般是如下格式：

{PROJECT_NAME}_{QLocale.system().name()}.ts

其中PROJECT\_NAME是你项目的名字，而QLocale.system().name()是你当前机器所用的目标语言简写，你可以在python3的eval模式下查看一下：

>>> from PyQt5.QtCore import QLocale
>>> QLocale.system().name()
'zh_CN'

然后你需要用pylupdate5小工具处理该pro文件：

```sh
pylupdate5 wise.pro
```

这样你就可以看到生成的 `wise_zh_CH.ts` 文件了，然后请确保安装了qt4-dev-tools，

```sh
sudo apt-get install qt4-dev-tools
```

这样你就可以双击打开ts文件，操作很简单，看见对应的英文单词，然后填上相应的中文解释。操作玩了点击发布，即看到生成的qm文件，或者使用命令行工具 **lrelease** 。

## 使用翻译文件<a id="orgheadline43"></a>

样例如下：

```python
from PyQt5.QtCore import QTranslator,QLocale
myapp = QApplication(sys.argv)
translator = QTranslator()
if translator.load('wise_'+ QLocale.system().name()+'.qm',
":/translations/"):
myapp.installTranslator(translator)
```

首先你需要构建一个QTranslator对象，然后调用该对象的方法load，这里第一个参数是要load的qm文件名，第二个参数是qm文件的路径，可以使用前面谈及的qrc引用路径。

最后你的主母窗口myapp使用installTranslator方法把这个QTranslator对象加进去即可。

## 使用qt官方翻译文件<a id="orgheadline44"></a>

有些qt窗体内部文字可能不好DIY，这时需要如上一样加载qt的官方翻译文件。代码如下所示：

```python
translator_qt = QTranslator()
if translator_qt.load('qt_'+ QLocale.system().name()+'.qm',":/translations/"):
# print('i found qt')
myapp.installTranslator(translator_qt)
```

这样你的主母窗口myapp现在需要加载两个翻译文件了。

官方qt翻译文件在qt源码的translations文件夹里面，可以如下通过git clone获取。

```sh
git clone https://gitorious.org/qt/qttranslations.git
```

ts文件如前所述用 **lrelease** 命令处理以下，或者直接用语言工具打开然后发布即可。

# 附录<a id="orgheadline50"></a>

## PyQt4和PyQt5的区别整理<a id="orgheadline46"></a>

- 很多pyqt4原放在QtGui里面的一些QWidget在pyqt5里面都放入QtWidgets里面去了。一个简单的解决兼容性问题的方案就是在pyqt4里面引入的时候写上:

from PyQt4.QtGui import *

在pyqt5里面引入的时候写上:

from PyQt5.QtGui import *
from PyQt5.QtWidgets import *

## 引用信号发射对象<a id="orgheadline47"></a>

`sender` 方法来自GObject，所以一般Qt里的窗体对象都可以用。其用法主要在槽里面，调用 `self.sender()` ，即返回一个发射该信号从而调用该槽的对象。

## 菜单栏看不见？<a id="orgheadline48"></a>

不过可能你会遇到麻烦，我就折腾了好久，因为菜单栏总是显示不出来，然后才发现是系统环境的问题，我在GNOME下看不到pyqt5做的软件的菜单栏了，但是到Ubuntu默认的Unity环境下最上面的面板就是菜单栏了，这个值得说一下。

如果你在Unity环境（Ubuntu14.04）下，那么不需要做什么，如果你在gnome或者KDE上，那么qt5的菜单栏可能会显示不出来，你需要删除下面这个小东西。

```sh
sudo apt-get remove appmenu-qt5
```

把这个小软件删除，pyqt5上的菜单栏就能正常显示了，不过在unity环境下菜单栏不会显示在最上面的面板上了，而是常规的在图形GUI标题栏下面了。

## 参考资料<a id="orgheadline49"></a>

1. pyqt4教程，[PyQt4\_Tutorial](http://blog.cx125.com/books/PyQt4_Tutorial/)

2. Rapid GUI Programming with Python and Qt , 书籍的源码

3. C++-GUI-Programming-with-Qt-4-Second Edition 书籍的源码

4. [PyQt4各个类参考](http://pyqt.sourceforge.net/Docs/PyQt4/classes.html)