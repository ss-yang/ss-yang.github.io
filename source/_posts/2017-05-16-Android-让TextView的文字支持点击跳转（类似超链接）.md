---
title: Android 让TextView的文字支持点击跳转（类似超链接）
date: 2017-05-16 18:13:48
tags: Android
categories: Android
---


TextView本身不支持onClick事件，不过借助ClickableSpan类的onClick和SpannableString类可以很方便地实现跳转：

    TextView login_quit = (TextView)headerView.findViewById(R.id.tv_login_quit);
    String login = "登陆";
    SpannableString spannableString=new SpannableString(login);
    spannableString.setSpan(new ClickableSpan() {
        @Override
        public void onClick(View view) {
            Intent intent=new Intent(MainActivity.this,LoginActivity.class);
            startActivity(intent);
        }
    }, 0, login.length(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
    login_quit.setText(spannableString);
    login_quit.setMovementMethod(LinkMovementMethod.getInstance());

这样就可以实现点击文字跳转的功能了。

附上setSpan函数的原型，理解各个参数：

    public void setSpan(Object what, int start, int end, int flags) {
        super.setSpan(what, start, end, flags);
    }


效果：

![效果图](2017-05-16-Android-让TextView的文字支持点击跳转（类似超链接）/headerView.png)