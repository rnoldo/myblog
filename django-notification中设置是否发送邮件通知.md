Date:2012/04/02
#django-notification中设置是否发送邮件通知

最近用django-notification来实现提醒功能，作为django社区中比较有名气的一个app，它的文档实在不敢恭维。 为了搞清楚怎样设置才能只是站内提醒，而不发送邮件，我翻开了它的源代码，终于弄懂了。

因为是设置是否发送邮件，因此我从发送函数来找寻，发送邮件的函数send调用的是send_now，后者主要做了以下两方面工作：  

1、创建一个notice

	notice = Notice.objects.create(recipient=user, message=messages["notice.html"], notice_type=notice_type, on_site=on_site, sender=sender)  


2、检查should_send函数的设置，看是否要发送邮件  


	if should_send(user, notice_type, "1") and user.email and user.is_active:   
	    recipients.append(user.email)
	    send_mail(subject, body, settings.DEFAULT_FROM_EMAIL, recipients)

很明显，决定是否发送邮件的函数是should_send，它的定义如下：

	def should_send(user, notice_type, medium):
	    return get_notification_setting(user, notice_type, medium).send
它调用的get_notification_setting函数如下：  
	
	def get_notification_setting(user, notice_type, medium):
		try: 
			return NoticeSetting.objects.get(user=user, notice_type=notice_type, medium=medium)
		except NoticeSetting.DoesNotExist:
		default = (NOTICE_MEDIA_DEFAULTS[medium] <= notice_type.default)
		setting = NoticeSetting(user=user, notice_type=notice_type, medium=medium, send=default)
		setting.save()
		return setting  

也即是否发送邮件取决以下两行：

	default = （NOTICE_MEDIA_DEFAULTS[medium] <= notice_type.default）
	send=default

而models.py中有：

	NOTICE_MEDIA = ( ("1", _("Email")), )
	NOTICE_MEDIA_DEFAULTS = { "1": 2 }

即notice_type.default小于2的话则不发送。 这个参数是在app的management.py那里用create_notice_type创建NoticeType的时候确定的，函数如下：

	def create_notice_type(label, display, description, default=2, verbosity=1):
	"""
	Creates a new NoticeType.
	This is intended to be used by other apps as a post_syncdb manangement step.
	"""
	try:
		notice_type = NoticeType.objects.get(label=label)
		updated = False
		if display != notice_type.display:
			notice_type.display = display
			updated = True
		if description != notice_type.description:
			notice_type.description = description
			updated = True
		if default != notice_type.default:
			notice_type.default = default
			updated = True
		if updated:
			notice_type.save()
		if verbosity > 1:
			print "Updated %s NoticeType" % label
	except NoticeType.DoesNotExist:
		NoticeType(label=label, display=display, description=description, default=default).save()
		if verbosity > 1:
			print "Created %s NoticeType" % label

缺省default=2，也就是默认是发送邮件的

因此，如果要想定义一个不发送邮件的notice，则需要在management.py中调用create_notice_type的时候，将default设置为1：
例如：  `notification.create_notice_type("messages_sent", _("Message Sent"), _("you have sent a message"), default=1)`

如果不在management.py中定义NoticeType，直接使用send_now的话，由于没有default参数，因此默认是发送邮件的（avatar使用的就是这种方式）。
`def send_now(users, label, extra_context=None, on_site=True, sender=None)`