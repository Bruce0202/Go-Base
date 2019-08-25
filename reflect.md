##基础用法
>>reflect 主要有reflect.value && reflect.type两个方法。  
`t :=reflect.TypeOf(Obj); v :=reflect.ValueOf(Obj)`  
我们可以直接在源码里面看到t返回的是一个Type类型
的变量里面可以说包括了这个Obj的所以信息(大家可以在源码里面看到最全的解析，我这里就拿出一些最常用的
并且我后面会讲解的来讲)分别是:  t比较常用的一些方法：  
 `Method(int) Method`  
 `MethodByName(string)(Method, bool)`  
 `NumMethod() int`  
 `Name() string`  
 `Kind() string`  
 `Elem() type`
 `Field(i int) StrutField`
 `FieldByName(name string) (StructField, bool)`  
 `NumField() int`  
 相较与t，v的只有三个值:  
 `typ *rtype`  
 `ptr unsafe.Pointer`  
 `flag`  
 对于使用来说，v的typ这个字段很重要，我们可以看到typ的类型是*rtype，这里就要提起
 `reflect.TypeOF()`的底层的实现了，在go的源码里面我们可以看到typeOf的实现是这样的
 ``````
 func TypeOf(i interface{}) Type {
 	eface := *(*emptyInterface)(unsafe.Pointer(&i))
 	return toType(eface.typ)
 }
 func toType(t *rtype) Type {
 	if t == nil {
 		return nil
 	}
 	return t
 }
 ``````  
 

 
