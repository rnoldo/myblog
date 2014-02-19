Date:2012-05-08
#django——signals和狼烟

静下心来从需求和应用场景来思考一个技术的时候，往往能对它有非常透彻的理解，而且不容易遗忘。拿django的signal来说：

需求和应用场景：当系统中的某个事件发生的时候，比如一个User创建的时候，系统执行一个操作，比如自动为其创建一个profile。

现实中，我们发信号的过程是怎样的呢？最典型最古老的信号机制莫过于“狼烟”，作为一个信号传递的古老手段，它通过以下几步来完成:  
1、定义信号（signal或event），解决信号是什么的问题，是吹号角还是发电报，这里则是“点燃一股狼烟”。  

2、约定接收者的反应（callback），解决看到信号后作何反应的问题，这里是“看到狼烟，做好防御”。  

3、将接收者的反应和信号绑定，解决如何建立二者之间联系的问题。  

4、发射时机，解决什么时候发射的问题，这里是“匈奴等入侵者来的时候发射”  


狼烟信号的使用过程示例代码如下：

	import django.dispatch
 
	"""
	1、定义信号
	"""
	enemy_detected = django.dispatch.Signal(providing_args=["enemy"])
 
	"""
	2、约定采取的行动：看到信号，准备防御
	"""
	def defense_ready(sender, instance, signal, created, **kwargs):
    	defense_ready()
 
	"""
	3、将信号和约定绑定在一起
	"""
	enemy_detected.connect(defense_ready, sender=Soldier)
 
	"""
	4、信号发送：当望到敌人的时候，发送信号
	"""
	if enemy_detected:
		enemy_detected.send(sender=Soldier, enemy=enemy)
django中的signal机制跟这个毫无二致，也要经过这四步，不过系统定义了一些内置信号。譬如需求和应用场景中需要用到的post_save信号，它将“定义信号”和“发射时机”两个问题一并解决了。我们只需要“约定接收者的反应”并“将信号和约定绑定在一起”，即定义callback函数并绑定在信号上即可。