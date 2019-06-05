# Go-Base
来源：http://c.biancheng.net/golang/
有关go的一些基础

## Go语言的函数类型实现接口（把函数作为接口来调用）
>Go语言中的interface是个很有意思的东西，把数据的传递分离开了，有点相互通讯的意思

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
    
## Go语言空接口类型（interface{}）
>go语言中的interface{}相当于c++中的std::any。空接口保存一个数据的过程会比直接用数据对应的类型保存数据稍慢  
>空接口不能将值直接赋值给相应的数据类型  
>比如：  
>>   var a int = 1  
>>    var i interface{} = a  
    var num int = i  
    上面的程序会报错，因为，i是interface{}类型的，不能直接赋值给int型的变量  
    可以使用 type assertion（断言）  
    var b int i.(int)  
interface{}类型的变量也是可以比较的：  
    map 不能比较， 会触发宕机  
    切片[]T 不能比较， 会触发宕机  
    channel 可比较 必须同一个make生成，同一个通道次啊会显示true  
    [num]T  可比较 编译期知道两个数组是否一致  
    结构体  可比较 可以逐个比较结构体的值  
    函数     可比较  

interface{}的一个很好的用法：  
```
type Dictionary struct {
	data map[interface{}]interface{} // set the type of KEY_VALUE as interface{} to accept any type
}

func (d *Dictionary) Get (key interface{}) interface{} {
	return d.data[key]
}

func (d *Dictionary) Set (key interface{},value interface{}) {
	d.data[key] = value
}

func (d *Dictionary) Visit (callback func(k, v interface{}) bool)  {
	if callback == nil{
		return
	}
	for k, v := range d.data{
		if !callback(k, v){
			return
		}
	}
}

func (d *Dictionary) Clear ()  {
	d.data = make(map[interface{}]interface{})
}

func NewDictionary() *Dictionary{
	d := &Dictionary{}
	d.Clear()
	return d
}


func main() {
}
```
##Go package  
>GOPATH
>>GOPATH 指定的是工程目录，代码会保存在$GOPATH/src下面，工程经过go build, go install, go get之后会产生二进制文件是放在$DOPATH/bin下面
>>中间的缓存文件会保存在$GOPATH/pkg下；
>>在利用终端去编写go的package的时候，如果不是在默认的目录下，每次都应该指定GOPATH的值，用的是
```
export GOPATH=`pwd`
```
>>关于设定package的GOPATH的时候，有global GOPATH and Project GOPATH两种，project GOPATH会将设定好的GOPATH保存在工作目录的.idea目录下
>>不会被设定到环境变量的GOPATH中，但是编译的时候会使用这个目录。
>>`建议在开发的时候不要使用global GOPATH这个`

##go并发  
	goroutine是由Go语言的运行时调度完成的，而线程是由操作系统调度完成的，其中channel是多个goroutine之间通讯的基础

    
    
    
    
