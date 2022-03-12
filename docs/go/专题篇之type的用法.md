# 专题篇之 type 的用法

## 一、结构体

```go
type Engine struct {
	router map[string]http.HandlerFunc
}
```

## 二、接口

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

## 三、JSON

```go
type T struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}
```

## 四、自定义函数类型

```go

type myFunc func(int) int

func (f myFunc) sum(a, b int) int {
	res := a + b
	return f(res)
}

func sum10(num int) int {
	return num * 10
}

func sum100(num int) int {
	return num * 100
}

func handlerSum(handler myFunc, a, b int) int {
	res := handler.sum(a, b)
	fmt.Println(res)
	return res
}

func main() {
	handlerSum(sum10,1,1)
	handlerSum(sum100,1,1)
}
```

## 自定义基本类型

```go
type myInt int
```
