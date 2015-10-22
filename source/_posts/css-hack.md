title: css hack
date: 2015-06-09 17:43:51
tags: [css, hack]
category: CSS
---

# CSS Hack #

1. *IE6*中float元素双倍margin问题
	解决方案: 在这个div里面加上 display:inline; 例如： <#div id=”imfloat”> 相应的css为 ` #IamFloat{ float:left; margin:5px;/*IE下理解为10px*/ display:inline;/*IE下再理解为5px*/} `
2. *IE*不支持min-属性
	解决方案： 如果只用宽度和高度，正常的浏览器里这两个值就不会变，如果只用min-width和min-height的话，IE下面根本等于没有设置宽度和高度。 比如要设置背景图片，这个宽度是比较重要的。要解决这个问题，可以这样：` #box{ width: 80px; height: 35px;}html>body #box{ width: auto; height: auto; min-width: 80px; min-height: 35px;} `

3. *IE*不支持页面最小宽度min-设置
	解决方案： 它可以指定元素最小也不能小于某个宽度，这样就能保证排版一直正确。但IE不认得这个，而它实际上把width当做最小宽度来使。为了让这一命令在IE上也能用，可以把一个<div> 放到 <body> 标签下，然后为div指定一个类, 然后CSS这样设计：` #container{ min-width: 600px; width:expression(document.body.clientWidth < 600? "600px": "auto" );} `第一个min-width是正常的；但第2行的width使用了Javascript，这只有IE才认得，这也会让你的HTML文档不太正规。它实际上通过Javascript的判断来实现最小宽度。 

4. *IE*左边浮动元素与右边元素产生3px边距问题
	DIV浮动在IE下产生3象素的bug 左边对象浮动，右边采用外补丁的左边距来定位，右边对象内的文本会离左边有3px的间距. ` #box{ float:left; width:800px;} #left{ float:left; width:50%;} #right{ width:50%;} *html #left{ margin-right:-3px; //这句是关键} ` ` <div id="box"> <div id="left"></div> <div id="right"></div> </div> `

5. *All*float的div闭合，清除浮动，浮动元素父级自适应高度

- <#div id=”floatA” ><#div id=”floatB” ><#div id=” NOTfloatC” >这里的NOTfloatC并不希望继续平移，而是希望往下排。(其中floatA、floatB的属性已经设置为 float:left;) 原因是NOTfloatC并非float标签，必须将float标签闭合。在 <#div class=”floatB”> <#div class=”NOTfloatC”>之间加上 ` < #div class=”clear”> `这个div一定要注意位置，而且必须与两个具有float属性的div同级，之间不能存在嵌套关系，否则会产生异常。并且将clear这种样式定义为为如下即可：` .clear{ clear:both;} `

- 作为外部 wrapper 的 div 不要定死高度,为了让高度能*自动适应*，要在wrapper里面加上overflow:hidden; 当包含float的 box的时候，高度自动适应在IE下无效，这时候应该触发IE的layout私有属性(万恶的IE啊！)用` zoom:1; `可以做到，这样就达到了兼容。 例如某一个wrapper如下定义：` .colwrapper{ overflow:hidden; zoom:1; margin:5px auto;} `

- 内部为浮动元素，父级wraper需要居中，但是又要自适应高度。对于排版,我们用得最多的css描述可能就是float:left.有的时候我们需要在n栏的float div后面做一个统一的背景,譬如: ` <div id=”page”> <div id=”left”></div> <div id=”center”></div> <div id=”right”></div> </div> `比如我们要将page的背景设置成蓝色,以达到所有三栏的背景颜色是蓝色的目的,但是我们会发现随着left center right的向下拉长,而 page居然保存高度不变,问题来了,原因在于page不是float属性,而我们的page由于要居中,不能设置成float,所以我们应该这样解决` <div id=”page”> <div id=”bg” style=”float:left;width:100%”> <div id=”left”></div> <div id=”center”></div> <div id=”right”></div> </div> </div> `再嵌入一个float left而宽度是100%的DIV解决之。*一个float的元素，其父级非float时，父级无法自适应高度；父级float时能只适应高度。当还需要居中时，考虑多嵌套一层wraper*
