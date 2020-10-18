# Flutter-路由管理

路由(Route) 在移动开发中通常指页面(Page),这跟web开发中单页应用的Route 概念意义是相同的，Route在Android中通常指一个Activity,在iOS 中指一个ViewController。所谓路由管理，就是管理页面之间如何跳转，通常也可被称为导航管理。Flutter中的路由管理和原生开发类似，无论是Android 还是 ios,导航管理都会维护一个路由栈，路由入栈(push)操作对应页面关闭操作，而路由管理主要是指如何来管理路由栈。

