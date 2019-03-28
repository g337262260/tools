# Activity

四大组件中出现频率最高的的组件，与用户交互的第一接口，它提供了一个用户完成指令的窗口。当开发者创建Activity后，通过调用setContentView(View) 方法给该Activity指定一个显示的界面，并以此为基础提供给用户交互的接口。系统采用Activity栈的方式来管理Activity。

## Activity形态

|       形态       |           特征            |
| :------------: | :---------------------: |
| Active/Running | 处于Activity栈的最顶层，与用户进行交互 |
|     Paused     |   Activity失去焦点，保留多有信息   |
|    Stopped     |      被完全覆盖，保留所有信息       |
|     Killed     |     被系统回收或者是重来没有创建过     |

s