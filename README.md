# Go-Base
来源：http://c.biancheng.net/golang/
有关go的一些基础

## Go语言的函数类型实现接口（把函数作为接口来调用）
Go语言中的interface是个很有意思的东西，把数据的传递分离开了，有点相互通讯的意思

    type FuncCaller func(interface{}) //这里讲匿名函数定义为类型，共之后实现接口用，因为函数声明不能直接作为接口    
    func (f FuncCaller) Call(p interface{}){
            //对接口传入的函数进行调用
            f(p)
    }
    在main函数里面有：
    var invoker Invoker
    invoker = FuncCaller(func(v interface{}){
            fmt.println("from func", v)
        }
    )
    invoker.Call("Hello")
    
    
    
