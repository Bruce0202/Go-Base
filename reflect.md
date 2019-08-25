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
 >>就是将*i的指针强行的转成*emptyInterface，这个emptyInterface是一个这样的结构体
 ```cgo
type emptyInterface struct {
	typ  *rtype
	word unsafe.Pointer
}
```
>>最后就是的type是接受的*rtype类型的变量。这里就要看toType()这个方法的实现了 
我特意把toType的说明截取了下来， type是一个接口，它可以接受的这个*rtype类型的
变量后，可以转换成上面提到的接口里面各个字段
```cgo
// toType converts from a *rtype to a Type that can be returned
// to the client of package reflect. In gc, the only concern is that
// a nil *rtype must be replaced by a nil Type, but in gccgo this
// function takes care of ensuring that multiple *rtype for the same
// type are coalesced into a single Type.
func toType(t *rtype) Type {
	if t == nil {
		return nil
	}
	return t
}
```
>>我们在看看在value.go里面定义的`.type`的方法，v里面包含有`*rtype`信息，因此
`v.type()`实现的就是输出一个和`reflect.type`一样的值
```cgo
// Type returns v's type.
func (v Value) Type() Type {...}
```

##reflect，在复合类型上的使用  
>>其实在GO语言中`fmt,`下面的打印实现，大部分都是用reflect来实现的  
这里还需要引入一个数据结构`structField()`来一起讨论，我们一起来看一下
```cgo
type StructField struct {
	// Name is the field name.
	Name string
	// PkgPath is the package path that qualifies a lower case (unexported)
	// field name. It is empty for upper case (exported) field names.
	// See https://golang.org/ref/spec#Uniqueness_of_identifiers
	PkgPath string

	Type      Type      // field type
	Tag       StructTag // field tag string
	Offset    uintptr   // offset within struct, in bytes
	Index     []int     // index sequence for Type.FieldByIndex
	Anonymous bool      // is an embedded field
}
```
>>里面主要关注的有`name` & `type`这两个变量，其实name就是结构体字段，它会返回一个  
string类型的值，这个Type就是厉害了，也就是说，我们获得Type之后，就获得了这个字段的  
类型，方法等与这个字段有关的所有东西；下面我来看一段比较基础的代码：
```cgo
package main

import (
	"fmt"
	"reflect"
)

type People struct{
	name string
	age int
	children []*People
}

var Kobe = &People{
	name:"KOBE",
	age : 41,
	children:nil,
}

var Natalia = &People{
	name:"Natalia",
	age : 10,
	children:nil,
}

func main() {
	Kobe.children = append(Kobe.children, Natalia)
	typeOfKobe := reflect.TypeOf(Kobe)
	typeOfKobeAge := reflect.TypeOf(Kobe.age)
	valueOfKobe := reflect.ValueOf(Kobe)
	if typeOfKobe.Kind() == reflect.Ptr{
		typeOfKobe = typeOfKobe.Elem()
	}
	for i := 0; i < typeOfKobe.NumField(); i++{
		fmt.Println(typeOfKobe.Field(i))
	}
	for i := 0; i < typeOfKobe.NumField(); i++{
		fmt.Println(typeOfKobe.Field(i).Type.String())
	}
	fmt.Println(typeOfKobeAge)
	fmt.Println(valueOfKobe.Type().Elem().Kind())
}
the output:
{name main string  0 [0] false}
{age main int  16 [1] false}
{children main []*main.People  24 [2] false}
string
int
[]*main.People
int
struct
```
>>那么我们要遍历一个结构体改如何处理呢？其实遍历一个复杂类型的数据，和fmt的打印有很大的  
联系，我们可以参考`fmt`的源码：
```cgo
func (p *pp) printValue(value reflect.Value, verb rune, depth int) {
	// Handle values with special methods if not already handled by printArg (depth == 0).
	{...}
	switch f := value; value.Kind() {
	case reflect.Invalid:{...}
	case reflect.Bool:{...}
	case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:{...}
	case reflect.Uint, reflect.Uint8, reflect.Uint16, reflect.Uint32, reflect.Uint64, reflect.Uintptr:{...}
	case reflect.Float32:{...}
	case reflect.Float64:{...}
	case reflect.Complex64:{...}
	case reflect.Complex128:{...}
	case reflect.String:{...}
	case reflect.Map:{...}
	case reflect.Struct:{...}
	case reflect.Interface:{...}
	case reflect.Array, reflect.Slice:{...}
	case reflect.Ptr:{...}
	case reflect.Chan, reflect.Func, reflect.UnsafePointer:{...}
	default:{...}
	}
}
```
>>我们把里面单独拿出来：  
太细节的东西我们不去深究，我们看中间那个for循环，就是通过`name := f.Type().Field(i).Name; `  
将name打印出来了，他将值打印出来用的是`p.printValue(getField(f, i), verb, depth+1)`  
```cgo
	case reflect.Struct:
		if p.fmt.sharpV {
			p.buf.WriteString(f.Type().String())
		}
		p.buf.WriteByte('{')
		for i := 0; i < f.NumField(); i++ {
			if i > 0 {
				if p.fmt.sharpV {
					p.buf.WriteString(commaSpaceString)
				} else {
					p.buf.WriteByte(' ')
				}
			}
			if p.fmt.plusV || p.fmt.sharpV {
				if name := f.Type().Field(i).Name; name != "" {
					p.buf.WriteString(name)
					p.buf.WriteByte(':')
				}
			}
			p.printValue(getField(f, i), verb, depth+1)
		}
		p.buf.WriteByte('}')
```
>>fmt打印的方法，如果我们把结构体看成一棵树的话，那么fmt就是通过前序遍历来实现的  
由于只需要打印，所以不带返回值的前序遍历其实是很好处理的，我也写过一个带返回值的  
前序遍历打印结构体，当然是阉割版的，等有时间了再贴出来...

 
