## go1.18泛型的简单尝试

今天golang终于发布了1.18版本，这个版本最大的一个改变就是加入了泛型。虽然没有在beta版本的时候尝试泛型，但是由于在其他语言的泛型经验，入手泛型不是件难事~

#### 官方示例

[Tutorial: Getting started with generics - The Go Programming Language](https://go.dev/doc/tutorial/generics)

根据官方示例可以看出，在go中泛型声明使用中括号，大体用法也与其他语言差不多。下面就官方示例中出现的几个点做记录。

##### comparable

在泛型的约束中有 **comparable** 关键字，我们进到源码中看到解释：

```go
// comparable is an interface that is implemented by all comparable types
// (booleans, numbers, strings, pointers, channels, arrays of comparable types,
// structs whose fields are all comparable types).
// The comparable interface may only be used as a type parameter constraint,
// not as the type of a variable.
type comparable interface{ comparable }
```

看得出来这是官方定义的一个可比较的类型的一个泛型约束，它也只能存在于类型参数约束的时候。

#### 一些改变

我们尝试修改官方示例，体验一下其他的关键词及相关用法。

##### ~ 波浪号

我们会在一些泛型示例中看到这样的声明：

```go
type Number interface {
	~int64 | float64 | string
}
```

**~ 在这里应该可以理解为 泛类型 ，即所有以`int64`为基础类型的类型都能够被约束。**

我们来举个例子：现在我们声明一个以 `int64` 为基础类型，取名为`testInt`

```go
type testInt int64

type Number interface {
	~int64
}

func SumIntsOrFloats[K comparable, V Number](m map[K]V) V {
	var s V
	for _, v := range m {
		s += v
	}
	return s
}

func main() {
	ints := map[string]testInt{
		"first":  34,
		"second": 12,
	}

	fmt.Printf("Generic Sums: %v\n",
		SumIntsOrFloats(ints))
}
```

在这个示例中，可以看到我们将`testInt`作为自定义类型传入了泛型方法中，在这种情况下，如果不给`Number`中的`int64`加`~`，这里就会报错。**加上`~`之后代表以`int64`为基本类型的自定义类型也可以通过泛型约束**。

#### 未发布的内容

在泛型的测试阶段，有一个 `constraints` 包被加入到源码中，这个包里面声明了一些官方定义的约束，但是在正式的发布版中却被去掉了。

