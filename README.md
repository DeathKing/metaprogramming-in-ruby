# Ruby中的元编程

## 简介

　　本系列翻译自[Ruby Metaprogramming](http://ruby-metaprogramming.rubylearning.com/html/ruby_metaprogramming_1.html)站点上的课程笔记，并加入了我（[DeathKing](https://github.com/DeathKing)）的一些个人演绎、资料补充等。希望对大家有所帮助。

　　该课程由[Satoshi Asakawa](http://www.workingwithrails.com/people/153056-satoshi)讲授，使用ruby 1.9.1p243 [i386-mingw32]。而我的测试环境则是ruby 1.9.2p180 [i386-mingw32]。

　　本系列教程在2011年9月间翻译完毕，最初发布在我的博客上。于2014年5月间，藉由GitBook整理，并发布在GitHub上。在此期间，Paolo Perrotta所著的《Ruby元编程》也被翻译并出版。同时，Ruby版本也从1.9飞跃到了2.1，尽管如此，本系列介绍的Ruby元编程技术仍然可用，读者可以放心阅读。

## 翻译的动机

　　Ruby是动态的、魔幻而有趣的。而元编程（Metaprogramming）技术在Ruby中的应用也让我大开眼界，虽然以前也有浅显地介绍反射机制（Reflection），但我仍然觉得才疏学浅，不能让大家完全感受到元编程的优美。借此机会，我想借Satoshi Asakawa的本系列讲座，为大家展示一个绚丽的Ruby元编程世界。
