Date:2012/04/18
#django，为QuerySet中的元素添加属性

在view中进行数据表查询之后，有时需要添加一个额外的属性，比如一个用户发表的文章数目。这时可以有两种方法实现：  


* 1、添加一个数组，user_article_num，对每个返回的QuerySet中的user，计算出他的发帖数量，存储在user_article_num中。然后将其存放在extra_context中返回。

* 2、利用django提供的一个非常方便的方案：annotate，在view返回之前，为每个QuerySet中的user添加一个额外的属性，article_num：

		User.objects.annotate(article_num=Count('article')).order_by('article_num')

这样，返回的QuerySet中的每个user都添加了一个属性：article_num