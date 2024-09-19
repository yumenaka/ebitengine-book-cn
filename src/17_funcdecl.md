# 第十七章 把处理放在一起（函数声明）


前回是学习了如何使用结构体来汇总数据。这次，我们将学习如何使用函数来汇总处理。这次也有重构的环节哦。

## 函数


复习一下吧。

函数是某种处理的集合，可以传递参数（输入）并返回返回值（输出）。

此外，通过将长处理切分为函数，我们也改善了程序的可读性。这次我们将重新学习函数的声明方法。

##  参数的用法

使用参数时，函数声明为 `func 函数名(参数, 参数, ...) { ... }` 的形式。

引数可以像普通变量一样使用。

```go
func printMax(a int, b int) {
	fmt.Println("最大值是", max(a, b))
}
```

顺便提一下，在参数列表和返回值列表中（实际上变量声明和结构体字段也是如此），如果相同类型连续出现，可以省略最后一个参数类型。

```go
func printMax(a, b, c int) { // a int, b int, c int 的简略写法
```

##  返回值的用法与 return

返回值的函数同样以 `func 函数(参数, 参数, ...) (返回值, 返回值, ...) { ... }` 的形式声明。

```go
func minMax(a, b int) (minVal, maxVal int) {
	return min(a, b), max(a, b)
}
// ※ 为了避免与函数 min, max 重名，这里将参数命名为 minVal, maxVal 
```

新出现了 `return` （返回）关键字。 `return` 是结束函数处理并返回返回值的关键字。

 `return` 会在此处结束函数的处理，因此可以进行这样的操作。我们称这种在函数开头进行条件分支并提前结束处理的方式为早期返回/early return，这是一种常被推荐的技术，以保持程序的整洁。

```go
func minMax(a, b int) (minVal, maxVal int) {
	if a < b {
		return a, b // 满足条件的时候 return
	}
	// 只有上面↑的条件没有满足的时候、才会执行下面↓的语句
	return b, a
}
```

此外， `return` 也可以作为没有返回值的函数，仅用于结束处理。

```go
func fukubiki() {
	if rand.N(2) == 1 {
		fmt.Println("中奖了")
		return
	}
	fmt.Println("没中奖")
	// return ← 因为没有返回值，所以这一句可以省略
}
```


返回值为 None 的函数可以在不使用 `return` 的情况下结束。另一方面，返回值的函数必须使用 `return` 返回返回值并结束。

###  返回值的省略形式

函数声明的返回值部分有多种省略形式。

首先可以不为返回值命名，仅指定类型。

```go
func minMax(a, b int) (int, int) {
```

如果返回值没有起名，且只有一个返回值，可以省略包裹返回值的 `()` 。

```go
func add(a, b int) int {
```

最后，正如您所知，返回值为空时，无需写任何返回值相关内容。

```go
func fukubiki() {
```

!此外，还有方法的声明、带类型参数的函数、带类型参数的函数方法，这些将在后面介绍。


模式有很多，但一开始并不需要完全记住所有语法。相反，可以**先记个大概，然后写出来试试，如果出错误再修正**。这样编程会更加顺利。

##  函数的使用场合

正如已经使用过的那样，除了将长时间的处理切分以便于查看外，当同样的处理需要重复多次时，声明函数并重新使用也是很方便的。

```go
func drawComplexObjects() {
	// 已经写好的数百行描绘处理
}

func draw() {
	// 可以简单地再利用
	drawComplexObjects()
	drawComplexObjects()
	drawComplexObjects()
}
```

此外，通过合理设计函数的参数和返回值，可以明确程序的边界，并获得可扩展性、可维护性和可读性，但这感觉是相当高级的技术，留待以后再说。

##  重构吧

那么和上次一样，我们来重构跳跃的 gopher 游戏，练习将处理汇总到函数中的方法。例如，以下这些点似乎可以汇总到函数中。

- 上面的墙和下面的墙的绘制处理在游戏画面和游戏结束画面中是共通的，因此可以进行合并
- 四角形之间的碰撞检测计算公式在上壁和下壁是共通的，因此可以进行整合

当然，还有无数其他方法可以将其汇总为函数，但请您理解这是为了说明。

```diff-go
package main

import (
	"embed"
	"math/rand/v2"

	"github.com/eihigh/miniten"
)

//go:embed *.png
var fsys embed.FS

type wall struct {
	wallX int
	holeY int
}

var (
	// ...省略...
)

func main() {
	miniten.Run(draw)
}

func draw() {
	// ...省略...
}

func drawTitle() {
	// ...省略...
}

func drawGame() {
	// ...前略...

	for i := range walls {
		walls[i].wallX -= 2 // 少しずつ左へ
	}
	for _, wall := range walls {
		// 上の壁の描画
		miniten.DrawImageFS(fsys, "wall.png", wall.wallX, wall.holeY-wallHeight)

		// 下の壁の描画
		miniten.DrawImageFS(fsys, "wall.png", wall.wallX, wall.holeY+holeHeight)
		drawWalls(wall)

		// gopherくんを表す四角形を作る
		aLeft := int(x)
		aTop := int(y)
		aRight := int(x) + gopherWidth
		aBottom := int(y) + gopherHeight

		// 上の壁を表す四角形を作る
		bLeft := wall.wallX
		bTop := wall.holeY - wallHeight
		bRight := wall.wallX + wallWidth
		bBottom := wall.holeY

		// 上の壁との当たり判定
		if aLeft < bRight &&
			bLeft < aRight &&
			aTop < bBottom &&
			bTop < aBottom {
			scene = "gameover"
		}
		if hitTestRects(aLeft, aTop, aRight, aBottom, bLeft, bTop, bRight, bBottom) {
			scene = "gameover"
		}

		// 下の壁を表す四角形を作る
		bLeft = wall.wallX
		bTop = wall.holeY + holeHeight
		bRight = wall.wallX + wallWidth
		bBottom = wall.holeY + holeHeight + wallHeight

		// 下の壁との当たり判定
		if aLeft < bRight &&
			bLeft < aRight &&
			aTop < bBottom &&
			bTop < aBottom {
			scene = "gameover"
		}
		if hitTestRects(aLeft, aTop, aRight, aBottom, bLeft, bTop, bRight, bBottom) {
			scene = "gameover"
		}
	}

	if y < 0 {
		scene = "gameover"
	}
	if 360 < y {
		scene = "gameover"
	}
}

func drawGameover() {
	// 背景、gopher、壁の描画はdrawGame関数のコピペ
	miniten.DrawImageFS(fsys, "sky.png", 0, 0)
	miniten.DrawImageFS(fsys, "gopher.png", int(x), int(y))
	for _, wall := range walls {
		// 上の壁の描画
		miniten.DrawImageFS(fsys, "wall.png", wall.wallX, wall.holeY-wallHeight)

		// 下の壁の描画
		miniten.DrawImageFS(fsys, "wall.png", wall.wallX, wall.holeY+holeHeight)
		drawWalls(wall)
	}

	miniten.Println("Game Over")
	miniten.Println("Score", score)
	if isJustClicked {
		scene = "title"

		x = 200.0
		y = 150.0
		vy = 0.0
		frames = 0
		walls = []wall{}
		score = 0
	}
}

func drawWalls(w wall) {
	// 上の壁の描画
	miniten.DrawImageFS(fsys, "wall.png", w.wallX, w.holeY-wallHeight)

	// 下の壁の描画
	miniten.DrawImageFS(fsys, "wall.png", w.wallX, w.holeY+holeHeight)
}

func hitTestRects(aLeft, aTop, aRight, aBottom, bLeft, bTop, bRight, bBottom int) bool {
	return aLeft < bRight &&
		bLeft < aRight &&
		aTop < bBottom &&
		bTop < aBottom
}
```


怎么样，似乎又稍微看到了些希望呢？

##  什么时候、如何将其汇总为函数？

这次我们可以这样将其汇总为函数，但**何时、怎么抽取函数**是一个相当困难的问题。

例如这次，汇总了游戏画面和游戏结束画面的墙壁绘制处理，但如果出现了**“在游戏结束画面时想要改变碰到的墙壁外观”**这个需求，那该怎么办呢？在这种情况下，使用通用函数处理会有些困难，因此最好不要进行汇总。像这样，是否应该将处理汇总为函数，答案将会因时而异。

“多一点复制，总比多一点依赖更好。”这是一个 Go 谚语。这意味着，复制一些程序总比有依赖关系（A 需要 B 的关系，函数调用也是一种依赖关系）要好。“多一点”这个说法有些含混，不过这种规则本来就很难整齐划一，不过分拘泥于规则也很重要。

个人推荐“如果同样的处理出现了3次，就进行共通化”。出现 2次的时候，会有“不写在一起更好”的可能性，到了第3次时判断“这部分确实可以共通化”。我认为这样做可以保持相当的平衡。基于这一点，刚才的重构就显得不太合理了。但请您谅解，这么做是为了学习。

##  本章总结

我们学习了如何使用函数来汇总处理。函数可以接收参数并返回值，并且在 `return` 处结束处理。使用函数，可以提高程序的可读性。然而是否应该将共同部分整理成函数，会因情况而异。因此我们要记住，共通化并不总是正确的做法。
