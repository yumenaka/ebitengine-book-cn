# 第十九章 将函数绑定到类型（method）


次我们将学习与类型相关的函数的处理。尽管功能简单，但它有可能大幅改善程序的编写体验。

## 将函数与类型绑定在一起

函数声明时，如果在函数名之前添加 `(参数名 数据类型)` 这样的部分，就可以将该函数与类型关联。与类型关联的函数也称为方法/method，可以像 `变量名.方法名` 这样如同一个字段一样使用。

先试试看吧。

```go
// 型宣言
type user struct {
	name string
	age  int
}

// 普通の関数宣言
func userPrintln(u *user) {
	fmt.Println("名前:", u.name, "年齢:", u.age)
}

// メソッド宣言
// (u *user) という部分が、user型に紐づく関数であることを示している
func (u *user) println() {
	// 処理は普通の関数と全く同じ
	fmt.Println("名前:", u.name, "年齢:", u.age)
}

func main() {
	u := user{name: "Taro", age: 20}
	// 普通の関数の利用
	userPrintln(u)
	// メソッドの利用
	// 変数名.メソッド名 で利用できる
	u.println()
}
```


如您所见，仅通过在函数声明中更改 `(u *user)` 的位置，就可以将函数绑定到类型 `*user` ，并使其 `变量名.方法` 的语法可用。


函数名前的 `(u *user)` 这样的参数，有时被特别称为接收器/receiver。

### 函数的优点

函数的优点在于，只要类型不同，即使名称重复也没关系。

用专业术语来说，就是名称空间/namespace 被区分开了。

```go
func (u *user) println() {
	fmt.Println("名前:", u.name, "年齢:", u.age)
}

// 異なる型なら同じ名前のメソッドを宣言できる
func (p *position) println() {
	fmt.Println("X座標:", p.x, "Y座標:", p.y)
}

func main() {
	u := user{name: "Taro", age: 20}
	u.println() // 名前: Taro 年齢: 20
	p := position{x: 10, y: 20}
	p.println() // X座標: 10 Y座標: 20
}
```

由于不必担心重名，因此方法名可以相对简短，没问题。

方法也是更重要的功能“接口”的基础，不过这部分稍后再说。现在只需记住“命名更容易，好方便啊~”就可以了。

### 与指针的关系

调用方法时，无论是指针还是非指针，都可以完全相同地使用 `变量名.方法` 。

```go
func (u *user) println() {
	fmt.Println("名前:", u.name, "年齢:", u.age)
}

func main() {
	// 非ポインター変数を使う
	taro := user{name: "Taro", age: 20}
	taro.println()
	// (&taro).println() ←こう書かなくて済む

	// ポインター変数を使う
	jiro := &user{name: "Jiro", age: 30}
	jiro.println()
}
```

因此，在调用方法时，基本上不需要关心接收者是否是指针。

### 接收器相关的惯例

习惯上，接收器通常会取首字母作为 `u` 等名称。在其他语言中，通常会赋予 `this` 或 `self` 等特殊关键字，但 Go 则有所不同。此外，建议同一类型的方法统一接收器名称（避免在某个地方使用 `u` ，在另一个地方使用 `usr` 的情况）。


这只是习惯，所以并不是绝对的规则。

## 本章总结

这次我们学习了与类型相关的函数“方法”。这一回相对简单。
