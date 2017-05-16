---
title: Android Material Design 获取NavigationView的headerView的方法
date: 2017-05-16 18:00:27
tags:
- Android
- MaterialDesign
categories:
- Android
---

- ### headerView

NavigationView的headerView通常用来放用户信息。

![用户信息](headerView.png)


- ### 找不到headerView的原因


今天在这里实现[点击“登陆”跳转功能](http://syang.site/2017/05/16/Android-%E8%AE%A9TextView%E7%9A%84%E6%96%87%E5%AD%97%E6%94%AF%E6%8C%81%E7%82%B9%E5%87%BB%E8%B7%B3%E8%BD%AC%EF%BC%88%E7%B1%BB%E4%BC%BC%E8%B6%85%E9%93%BE%E6%8E%A5%EF%BC%89/)的时候踩了个小坑，一直闪退，debug后才发现在这一句出问题（原来一直怀疑是ClickableSpan的问题，从一开始就走错了方向，耗了不少时间）：

    TextView login = (TextView)findViewById(R.id.tv_login_quit);

这里login的值为null，终于找到****原因****：findViewById找不到布局文件里面的TextView控件。

仔细看就发现headerView是一个子布局，父布局是NavigationView，那么问题就解决了！

- ### 获取headerView的****正确姿势****

    - 先****获取到NavigationView****

            NavigationView navigationView = (NavigationView) findViewById(R.id.nav_view);

    - 再通过NavigationView获取headerView

            View headerView = navigationView.getHeaderView(0);

            
        其中getHeaderView(int index) 传入0就可一，index是nav中header的序号，一般只有一个header.

    - 最后，通过headerView获取headerView里面的控件

            TextView login = (TextView)headerView.findViewById(R.id.tv_login_quit);

        注意是****headerView.findViewById****