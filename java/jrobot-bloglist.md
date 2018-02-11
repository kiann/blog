# Robot是java中一个非常好玩和有用的部分，通过它可以操作键盘鼠标，实现自动化测试，甚至游戏挂机等等功能

>这是头一次接触robot，我原本是想在公司OA上看一眼工号排到哪一位了，结果直接查看其他员工列表的页面提示我没有权限，而除此之外没想到还有哪儿能看全部工号。然后就想到虽然员工列表没有权限看，但是根据工号搜索员工是可以用的，果断开搞。

最初的设想是用fiddler抓包然后jsoup模拟一下手动搜索过程，把所有的员工都爬取一遍，结果脑细胞死了一大堆也没有完全理清楚OA的页面加载流程，前端加密了一遍又一遍，cookie和token到处都有，缺一个就返给我304，实在太恶心，还是算了吧。

但又不想放弃，~~说不定爬取过程中还能看到几个好看的小姐姐什么的~~，然后就想到了另一种爬取的办法，用代码模拟手动操作。

然后就有了JRobot。

>先说一下Robot类的几个常用方法，其实非常简单，先初始化一个robot。
```java
    Robot robot = new Robot();
```
首先是鼠标操作。
```java
    void mouseMove（int x，int y） //将鼠标移动到给定的屏幕坐标上
    void mousePress（int buttons） //按下一个或多个鼠标按键，参数InputEvent.BUTTON1_MASK
    void mouseRelease（int buttons） //释放一个活多个鼠标按键
    void mouseWheel（int wheelAmt） //滚动鼠标滑轮，参数：滚动距离，eg：5
```
鼠标按键对应：

* 左键 - InputEvent.BUTTON1_MASK
* 中键 - InputEvent.BUTTON2_MASK
* 右键 - InputEvent.BUTTON3_MASK

其次是键盘，类似
```java
    void keyPress（int keycode） //按下指定的键，参数KeyEvent.VK_A
    void keyRelease（int keycode） //释放指定的键
```
键盘功能键，
* alt   -   KeyEvent.VK_ALT
* ctrl  -   KeyEvent.VK_CONTROL
* shift -   KeyEvent.VK_SHIFT
* enter -   KeyEvent.VK_ENTER
* esc   -   KeyEvent.VK_ESCAPE
* space -   KeyEvent.VK_SPACE
* up    -   KeyEvent.VK_UP（上下左右）
* 1     -   KeyEvent.VK_0（0-9）
* A     -   KeyEvent.VK_A（A-Z）
* F1    -   KeyEvent.VK_F1（F1-F12）

再然后是截屏，robot提供全屏截屏和区域截屏的方法，
```java
    File save = new File("xxx/xxx/xxx.jpg");
    Rectangle screenRect = new Rectangle(20,20,100,100);//left,top,width,height
    BufferedImage image = createScreenCapture(screenRect);//截取指定区域的图像
    ImageIO.write(image,"jpg",save);
```
最后，在模仿手动操作的时候，一定要记得延时，因为代码运行的速度是非常快的，而我们需要GUI能够反应的过来。robot提供一个delay方法，相当于将当前线程sleep指定的毫秒值
```java
    void delay(int ms)
```

因为robot已经控制了我们的键盘和鼠标，所以在出错之后手动点击stop按钮是没办法了，我们还需要注册一个全局的快捷键，能够让我们迅速的停掉自己写的机器人。我使用了jintellitype。

因为java的跨平台特性，所以像注册系统热键钩子这种操作就得配合jni调用系统能力了，幸好有人已经封装好了，就是jintellitype。网上搜素可以下载到库文件，应该是一个jar包和两个dll文件，分别对应x86和x64系统，是的，只支持windows平台（linux平台可以使用[JXGrabKey](http://sourceforge.net/projects/jxgrabkey/),mac暂时没找到办法）,可以在代码中动态选择要使用哪个dll文件。

dll可以放在classpath能够加载到的目录下，这样jar包直接就能够找到dll文件，而如果想自己指定dll目录，那么将dll放在自定义的目录之后，需要在程序中第一时间调用
```java
JIntellitype.setLibraryLocation("xxx/libs/Jintellitype64.dll");
```
之后就可以注册快捷键
```java
    JIntellitype.getInstance().registerHotKey(11, JIntellitype.MOD_ALT, 'A');
```
以上的代码注册了ALT + A的快捷键，11是一个自定义的int值，用来在监听回调中区分快捷键来源

最后添加监听
```java
    JIntellitype.getInstance().addHotKeyListener(new HotKeyListener(){
        @override
        public void onHotKey(int i){
            if(i == 11){
                //TODO 您按下了ALT + A快捷键
            }
        }
    });
```

经过几个小时的折腾和一个来小时的抓取，我不仅知道了公司最大的工号，~~还顺便将在职小姐姐们的OA照片挨个截了一遍图~~，咳咳，当然，纯属技术研究。

因为API简单，代码也没什么复杂逻辑，更何况代码里的所有坐标和操作流程完全是写死的，毫无通用性可言，我就不上传代码了，看个热闹就行，如果你也想用，相信上面的介绍已经足够了。