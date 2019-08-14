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
 

 
