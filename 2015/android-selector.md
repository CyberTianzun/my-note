# Android中的Selector使用详解

## 废话

在进行布局的时候，为了更加方便地实现按钮点击状态或者某些状态的背景图切换、字体颜色切换等效果，Android提供了一种叫做 StateListDrawable 的东东，使用XML来进行定义不同状态下的图像。官方文档中举了这么一个例子。例如，一个按钮控件可以在几种不同的状态（按下，获取焦点，或者正常状态），使用 StateList ，你可以为每个状态绘制不同的背景图片。

## 重点

重点就是，Android 提供了这个 XML 定义里边，有9大9种状态，作为一个小学语文古诗词都从来背不到的记忆力障碍患者来说，太难记了，说不定哪天去面试就跪在这个这么基础的东西上了，所以这9个状态是本文的复习重点。

## 使用

#### 文件位置

  res/drawable/filename.xml
  
文件名就是资源id
  
#### 编译后的资源类型

在程序中获取到的这个资源对象的类型是 (StateListDrawable)[http://developer.android.com/reference/android/graphics/drawable/StateListDrawable.html]

#### 引用方法

在 Java 代码内： `R.drawable.filename`
在 XML 内：`@[package:]drawable/filename`

#### 语法

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android"
    android:constantSize=["true" | "false"]
    android:dither=["true" | "false"]
    android:variablePadding=["true" | "false"] >
    <item
        android:drawable="@[package:]drawable/drawable_resource"
        android:state_pressed=["true" | "false"]
        android:state_focused=["true" | "false"]
        android:state_hovered=["true" | "false"]
        android:state_selected=["true" | "false"]
        android:state_checkable=["true" | "false"]
        android:state_checked=["true" | "false"]
        android:state_enabled=["true" | "false"]
        android:state_activated=["true" | "false"]
        android:state_window_focused=["true" | "false"] />
</selector>
```

#### selector 标签

官方文档说，这个标签是必须的。。。好吧，你不说我还不知道呢，又学到了新姿势好高兴呢(\*^__^\*) 嘻嘻……。
这个标签需要包含一个或多个 item 标签，selector 标签有下列属性：

* `xmlns:android="http://schemas.android.com/apk/res/android"`

  这个我不想废话了

* `android:constantSize=["true" | "false"]`

  这个东西有什么用呢？这一东西是说我们在后面那个item里面设置drawable这个东西的大小是不是固定的。我们这个文件一般都是用作控件的Backgroup或者selector总之就是背景状态，一般背景都是把控件的后面全部覆盖，但有的时候我们要设置设固定的大小，比如一个Button有300\*200大，而设置这个Button的背景图片只有200\*100，而现在我们又不想图片被拉大把覆盖整个Button的底层，那么就可以把这个属性设置为true，这样图片就只显示在中间了，就像我们设置桌面背景一样，可以设置成居中、拉伸，如果这里设置成true就相当于居中，如果不设置或者设置为false就是拉伸.

* `android:dither=["true" | "false"]`

  这个东西是说是否让系统来帮我们处理颜色差异，一般android系统中使用的颜色是ARGB_8888，但很多显示设置是RGB_565，这个ARGB_8888与RGB_565有什么区别呢。这个ARGB_8888也就是说每一个像素点要拿4个字节来保存，依次每个字节是A8个字节，R8个字节，G8个字节，B8个字节，来保存，而RGB_565它只用了两个字节来保存颜色,两个字节总共16位，前5位保存R,中间6位保存G,后5位保存B.因此呀，如果android系统的点显示到屏幕上，还得转换一下,在这里这个dither就起作用了,

  如果我们把它设置为true的话，那显示的时候屏幕间断的取点，这样的结果，有的时候看上去就有那种分层的感觉，也就是前面一部分的颜色与后面一部分的颜色感觉断层了，就是很不平滑的感觉，如果我们这里设置为true的话，默认就是true，android系统，它会在取的点之间再经过一些计算，在其间补充一点相间的颜色使看起来比较平滑，但这样和真的图片还是有差异的，因些有的人想要得到很逼真的显示，这里就得自己来计算了，自己来计算，即占内存又占cpu，但颜色可以很逼真，如果有这样的需求那这里就要把这个属性设置为false

* `android:variablePadding=["true" | "false"]`

  这个是可变的填充，这个有什么用呢？这个就是，在当当前这个组件被selected的时候，比如某一个tab被selected,或者listView里面的个item被selected的时候，如果设置为true的话，那么被选的这个tab或item的填充就会变大，使得看上去与其它的tab或item不一样

#### item 标签

这个标签里边定义了每一个状态。这个标签有下列属性：

* `android:drawable="@[package:]drawable/drawable_resource"`

  这个就是在*当前这个状态*item的图片（drawable），这个是必须的。

  > **Note:** 事实上除了 `android:drawable` 标签，还有一个 `android:color` 标签，在不使用图片，而只使用颜色的时候使用。比如按钮的字体颜色选择器，字体颜色是没有办法用drawable来显示的。`android:drawable` 和 `android:color` 标签有且只能存在一个。

* `android:state_pressed=["true" | "false"]`

  这个是说当前这个组件是否被按下，如果要设置按下的那一刻的状态，那么这里就要设置为true，例如，一个Button当手按下去后，还没有离开的状态(就是touched住的时候，还没有放开，和Clicked，点击时的那一刻)。

* `android:state_focused=["true" | "false"]`

  这个是当获得焦点的时候的状态,就是当控件高亮的时候的状态，哪些情况可以造成此状态呢，比如说，轨迹球（有的手机上面有一个小球，可以用手指在上面向不同的方向滚动，滚动的时候，界面里面的焦点，就会转向滚动的方向的控件），还有就是d-pad之类的东西（比如果游戏手柄上面的上下左右键，还有键盘上面的上下左右键等）这些东西就可以控制组件上面的焦点。

* `android:state_hovered=["true" | "false"]`

  这个是api等组在14以上才有的,这个是当光标移动到某一个组件之上的时候的状态，到目前为止，还没有看见过哪个手机设备带有鼠标之类的东西，可能这个专门是为平板电脑设置的或者以后可能出现带有鼠标之类的设备而准备的吧，文档中说，一般这个值设置为与focused这个值一样。

* `android:state_selected=["true" | "false"]`

  这个是当一个tab被打开的状态。或者一个listView等里面一个item被选择的时候的状态,因此这个属性设置在一般的组件上面是没有用的，只有设置有作为tab或item的布局里面的项时，这个属才起作用.

* `android:state_checkable=["true" | "false"]`

  这个是当一个组件在可以 checked 或不可以 checked 的时候的状态，现在较常见的，能够 checkable 的组件有，单选项和多选项，所以这个属性只有设置在像这类组件上面才有作用的。

* `android:state_checked=["true" | "false"]`

  这个是当一个组件被 checked 或者没有c hecked 的时候的状态，也就是说只有在可checkable上面的组件才有作用的，一般常见的就是多选按钮组与单选按钮组里面的项，这个才有作用的。

* `android:state_enabled=["true" | "false"]`

  这个是当一个组件是否能处理touch或click事件的时候的状态，如果要对组件能否响应事件设置不同背景的时候，就要靠这个属性了.

* `android:state_activated=["true" | "false"]`

  这是一个 Android 3.0 加入的状态。一般这个东西用在当对象被持续选择的时候，（比如列表item的高亮状态）。与获取焦点不一点的是，如果用到多个对象被激活选中的时候，会适合使用这个状态，比如ListView多选等等。

* `android:state_window_focused=["true" | "false"]`

  这个是是否对当前界面是否得到焦点的两种状态的设置，比如说当我们打开一个界面，那么这个界面就获得了焦点，如果我们去把“通知”拉下来，那么这个界面就失去焦点，或者弹出了一个对话框，那么这个界面也失去焦点了。

> **Note:** 系统是从上往下匹配的，如果匹配到一个item那么它就将采用这个item，而不是采用的最佳匹配的规则，所以设置缺省的状态，一定要写在最后，很多人为了保险起见，一开始就把缺省的写好，那么这样后面所有的item就都不会起作用了，还会因此找不着哪里出了问题。

#### 样例

这个该死的文件在 `es/drawable/button.xml` :

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true"
          android:drawable="@drawable/button_pressed" /> <!-- pressed -->
    <item android:state_focused="true"
          android:drawable="@drawable/button_focused" /> <!-- focused -->
    <item android:state_hovered="true"
          android:drawable="@drawable/button_focused" /> <!-- hovered -->
    <item android:drawable="@drawable/button_normal" /> <!-- default -->
</selector>
```

然后在布局文件中的按钮上使用上边刚定义的状态：

```
<Button
    android:layout_height="wrap_content"
    android:layout_width="wrap_content"
    android:background="@drawable/button" />
```

---

## 扩展

#### Item标签内可以直接写Drawable对象 

 在XML的定义文件中，在每一个 Item 内可以直接使用 XML 定义的图形对象，比如 `<shape>` 或者 `<layer-list>`。例如下面这个例子
 
```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_focused="true">
        <layer-list>
            <item>
                <shape android:shape="rectangle">
                    <solid android:color="#FF0000"></solid>
                </shape>
            </item>
            <item android:bottom="1dip" android:left="1dip" android:right="1dip" android:top="1dip">
                <shape android:shape="rectangle">
                    <solid android:color="@color/white"></solid>
                </shape>
            </item>
            <item android:bottom="5dip">
                <shape android:shape="rectangle">
                    <solid android:color="@color/white"></solid>
                </shape>
            </item>
        </layer-list>
    </item>

    <item>
        <layer-list>
            <item>
                <shape android:shape="rectangle">
                    <solid android:color="#885500"></solid>
                </shape>
            </item>

            <item android:bottom="1dip" android:left="1dip" android:right="1dip" android:top="1dip">
                <shape android:shape="rectangle">
                    <solid android:color="@color/white"></solid>
                </shape>
            </item>
            <item android:bottom="5dip">
                <shape android:shape="rectangle">
                    <solid android:color="@color/white"></solid>
                </shape>
            </item>

        </layer-list>
    </item>
</selector>
```

## 参阅资料

1. (Android 开发文档 - State List)[http://developer.android.com/guide/topics/resources/drawable-resource.html#StateList]
2. (Android 开发文档 - StateListDrawable)[http://developer.android.com/reference/android/graphics/drawable/StateListDrawable.html]
3. (Selector中的各种状态详解)[http://blog.csdn.net/whyrjj3/article/details/7852761]
