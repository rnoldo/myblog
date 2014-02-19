Date:2012/04/19
#django——template中的context

django中一个网页的显示给用户的过程是这样的：用户访问一个url，比如”x.com/item/1“，django通过urls系统将”1“（item_id）传递给指定的view（比如item_detail(request, item_id)），view进行数据库查询等处理之后将变量和值存放在一个django.template.Context的实例中，如下面代码中黑体字所示：  

	def item_detail(request, item_id):
    	result = Item.objects.filter(pk=item_id)
    	template = loader.get_template('item/item_detail.html')
    	context = Context({ 'item': result })
    	response = template.render(context)
    	return HttpResponse(response)
最后将context传递到template中，这样在template中就可以以{{ item }}的方式获取item的值了。这种template渲染和response返回的方式比较繁琐，django的shortcuts提供了render_to_reponse函数来简化工作，它的调用方式通常如下：

	return render_to_response(template_name, { 'form': form }, context_instance=context)  

这里context则是将{ ‘form’: form }字典和context_instance指定的值合并起来的结果。这里的context_instance通常是RequestContext的一个实例。

RequestContext是Context的子类，它接受request作为参数，将settings.py中的 TEMPLATE_CONTEXT_PROCESSORS所指定的context processor函数产生的变量和值以及当前view产生的{{ ‘form’: form}}等值存放进context中。

context processer是一个函数，它接受request作为参数，返回一个dict。

	def media(request):
    	"""
   	 	Adds media-related context variables to the context.

    	"""
    	return {'MEDIA_URL': settings.MEDIA_URL}
settings.py中默认的TEMPLATE_CONTEXT_PROCESSORS 如下，它们将返回{ ‘user’: request.user }，{‘MEDIA_URL’: settings.MEDIA_URL}等dict。

	TEMPLATE_CONTEXT_PROCESSORS = (
	"django.core.context_processors.auth",
    "django.core.context_processors.debug",
    "django.core.context_processors.i18n",
    "django.core.context_processors.media",
    "django.core.context_processors.request",
    )
一个定义良好的view如下，它加入一个extra_context，以便在urls.py中向template中添加dict。

	def create_profile(request, form_class=None, success_url=None,
    	template_name='profiles/create_profile.html',extra_context=None)
上述的形式的view中context的处理方式如下：

	if extra_context is None:
    	extra_context = {}
	context = RequestContext(request)
	for key, value in extra_context.items():
		context[key] = callable(value) and value() or value