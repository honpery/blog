---
title: 解决flutter tabbar抖动问题
categories: 跨段
tags: Flutter
date: 2020-05-06 01:28:32
---

<p></p>
<!-- more -->

## 问题

Tabbar切换时缩放是很常见的交互，但是在flutter中通过以下代码会出现抖动现象。
主要原因是缩放过程中baseline变动导致的。

```dart
Tabbar(
  labelStyle: TextStyle(fontSize: 18),
  unselectedLabelStyle: TextStyle(fontSize: 14),
);
```

抖动效果：

![](/images/flutter_tab_old.png)

## 解法

解法是不改变字体大小就不会涉及baseline，通过整体tab来实现。

```dart
import 'package:flutter/material.dart';

class TabviewDemo extends StatefulWidget {
  TabviewDemo({Key key}) : super(key: key);

  @override
  _TabviewDemoState createState() => _TabviewDemoState();
}

class _TabviewDemoState extends State<TabviewDemo> with TickerProviderStateMixin {
  TabController tabController;
  var tabProgress = 0.0;

  final tabs = ["首页", "次页"];

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();

    if (!mounted) return;

    setState(() {
      tabController = TabController(length: tabs.length, vsync: this)
        ..animation.addListener(() {
          setState(() {
            tabProgress = tabController.animation.value;
          });
        });
    });
  }

  @override
  void dispose() {
    super.dispose();
    tabController.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: <Widget>[
          // tabbar
          Container(
            padding: EdgeInsets.only(top: MediaQuery.of(context).padding.top),
            height: 100,
            color: Colors.blue,
            child: Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Expanded(
                  child: TabBar(
                    isScrollable: true,
                    controller: tabController,
                    indicatorColor: Colors.transparent,
                    tabs: tabs.map((item) {
                      final index = tabs.indexOf(item);
                      final nextIndex = tabProgress.ceil();
                      final prevIndex = tabProgress.floor();
                      final progress = tabProgress - prevIndex;

                      var opacity = 0.7;
                      var activeOpacity = 1.0;
                      if (index == nextIndex) opacity = opacity + ((activeOpacity - opacity) * progress);
                      if (index == prevIndex) opacity = activeOpacity - ((activeOpacity - opacity) * progress);
                      if (index == nextIndex && index == prevIndex) opacity = activeOpacity;

                      var scale = 0.9;
                      var activeScale = 1.0;
                      if (index == nextIndex) scale = scale + ((activeScale - scale) * progress);
                      if (index == prevIndex) scale = activeScale - ((activeScale - scale) * progress);
                      if (index == nextIndex && index == prevIndex) scale = activeScale;

                      return Opacity(
                        opacity: opacity,
                        child: Container(
                          child: Transform.scale(
                            scale: scale,
                            child: Text(item, style: TextStyle(fontSize: 18, color: Colors.white)),
                          ),
                        ),
                      );
                    }).toList(),
                  ),
                ),
              ],
            ),
          ),
          // tabview
          Expanded(
            child: TabBarView(
              controller: tabController,
              children: List<Widget>.generate(tabs.length, (index) {
                return Text(tabs[index]);
              }),
            ),
          )
        ],
      ),
    );
  }
}
```

效果：

![](/images/flutter_tab_new.png)