今天是<a href="https://developers.google.cn/">https://developers.google.cn</a>开放的日子。相信国内的应用开发环境会越来越好。<br>
来携程IBU有段日子了，因为是新开展的国际业务，业务庞杂，前期为了短平快的开发，但是到了后面随着需求业务的增长和团队的壮大新功能的开发变得越来越困难。主要是存在几个方面：

* 多产线公用组件，业务代码揉进了组件离。
* 多产线公用页面，页面代码需要做硬编码区分。
* module之间互相依赖关系，关系不清晰。
* 接口定义使用不当，实体多出被向上转型为接口在页面之间传递

在开发造成中造成的问题：
* 代码可读性差，硬编码比较多。
* 团队协作性差，多会因为互相的模块依赖而被block进度。
* 接口被烂用拓展，实体类冗余很多无用代码。
* module之间的互相依赖导致无法单独抽离，编译成本过高。
* 模块难以单独提测，测试难度大。

<p>以上问题可能伴随过很多公司的app发展过程，走到这一步一般不得不解耦才能更好的团队协作。有的人抱怨前人埋坑，其实前期某些关系的耦合反而有助于开发效率。个人觉得，面对产品需求庞杂多变的情况，综合人手情况能做到效率最高就好了，你知道未来业务会发展成什么样？</p>

解决方案：
<p>
1、针对视图公用组件<br>
组件里面只封装视图相关的控制，数据的是适配坚持组件中只有一个公用的ViewModel。在show公用组件的入口组装好viewModel的数据再传入<br>
这样组件里面就不需要对不同的数据传入做兼容了。<br>
建议：组装数据的代码最好能在一个地方收口。
</p>
<p>
2、针对公用页面<br>有些业务比较类似或者期初一样的页面，都放在一起处理。不同的产线或者统一产线的不同入口都会跳转同一个页面。例如，火车票和机票使用同一个常用联系人页面ContantListActivity。随着业务变化，页面差别越来越大不能共用了。此时每个业务都要自己的ContantListActivity，如FligthContanListActivity/TrainContanListActivity.对应的产线处理维护自己的ContantListAcvitity.

<p>
3、针对module关系的解耦。
</p>

先看一个起初的依赖关系<br>

![pre dependency](../pics/pre_dependency.png)

<p>
弊端：<br>
1、部分base揉进了app主程序里面，其他module可能需要使用就需要自己造轮子或者去倒置依赖app主模块。<br>
2、module之间可能会造成互相依赖，如其他module想引用使用pay功能，就需要在module里面去交叉 dependency pay module<br>
3、没有提取不常修改的moudle，module需要频繁修改。主程序只能通过project dependency ,编译效率低。<br>
4、module抽象差频繁修改上游模块，影响面积大。<br>
</p>

改造后的依赖关系<br>

![index page](../pics/now-dependency.png)


优势:<br>
1、抽离Base Module 提供所有module的依赖基础，其他module不用重复造轮子。<br>
2、抽出Business Module处理部分产线公用部分（尽量干净）。Business Module负责提供module依赖的接口。如：其他的上层module想调用pay，通过business module转发，不直接依赖pay module。所有mouldle只依赖business module的api。business module的api不更改，pay即使更新其他模块也不受影响。<br>
3、主程序只负责维护自己业务代码。<br>

<p>
4、针对接口问题
</p>
这个比较简单，编码习惯上保证页面传值的实体传递尽量不要传递接口。多使用组合+集成的方式拓展接口。
