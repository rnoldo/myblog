#Martini中的Next方法解析
Martini是一个非常优雅的go语言web框架，它的中间件机制非常灵活，然而中间件中经常使用的Next方法却不怎么好理解。经过这段时间的思考，我清楚了它的用途，所以记下来，如果对你也有帮助，那也是件好事。

##Next的定义
	func (c *context) Next() {
		c.index += 1
		c.run()
	}
	
	func (c *context) run() {
		for c.index < len(c.handlers) {
			_, err := c.Invoke(c.handlers[c.index])
			if err != nil {
				panic(err)
			}
			c.index += 1

			if c.Written() {
				return
			}
		}
	}
##Next的使用
拿Martini提供的ClassicMartini来说，它使用了Recovery和Logger两个中间件。


	func Classic() *ClassicMartini {
		r := NewRouter()
		m := New()
		m.Use(Logger())
		m.Use(Recovery())
		m.Use(Static("public"))
		m.Action(r.Handle)
		return &ClassicMartini{m, r}
	}
	
	func Logger() Handler {
		return func(res http.ResponseWriter, req *http.Request, c Context, log *log.Logger) {
			start := time.Now()
			log.Printf("Started %s %s", req.Method, req.URL.Path)

			rw := res.(ResponseWriter)
			c.Next()

			log.Printf("Completed %v %s in %v\n", rw.Status(), http.StatusText(rw.Status()), time.Since(start))
		}
	}
	
	func Recovery() Handler {
		return func(res http.ResponseWriter, c Context, logger *log.Logger) {
			defer func() {
				if err := recover(); err != nil {
		      		res.WriteHeader(http.StatusInternalServerError)
					logger.Printf("PANIC: %s\n%s", err, debug.Stack())
				}
			}()

			c.Next()
		}
	}
	
## 分析
Logger的作用是，在requset进来的时候记下日志，request在其他中间件中被传递和处理之后，在response返回的时候也记下一个日志。
Recovery的作用是，request在其他中间件中传递和被处理的过程中出现panic的话则执行以下Recovery措施。
	
	res.WriteHeader(http.StatusInternalServerError)
	logger.Printf("PANIC: %s\n%s", err, debug.Stack())
	
想要达到这种效果，其他中间件必须被包裹（或者说嵌套）在Recovery和Logger中（两个log语句之间）。	
## 总结
那么，Next的作用就是：
> Next使其他中间件嵌套在当前中间件中执行。

从此我们也可以看出Martini中的中间件注册（m.Use）顺序也是有意义的。Logger和Recovery必须在其他中间件之前注册，它们才能在最外层，从而将其他中间件嵌套进来。
