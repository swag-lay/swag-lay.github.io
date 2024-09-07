---
title: go-交替打印
date: 2024-09-07 18:24:31
categories:
- golang
tags:
---

```golang
//交替打印
func main() {
	letterC := make(chan bool)
	numberC := make(chan bool)

	wg.Add(2)

	go func() {
		defer wg.Done()

		letters := []rune{'a', 'b', 'c'}
		for _, letter := range letters {
			<-letterC
			fmt.Printf("%c ", letter)
			numberC <- true
			if letter == 'c' {
				break
			}
		}
		close(letterC)
	}()

	go func() {
		defer wg.Done()

		numbers := []int{1, 2, 3}
		for _, number := range numbers {
			<-numberC
			fmt.Printf("%d ", number)
			if number == 3 {
				break
			}
			letterC <- true
		}

		close(numberC)
	}()

	letterC <- true

	wg.Wait()
}
```

```golang
//我的方案
func main() {
	one := make(chan int)
	two := make(chan int)

	wg.Add(2)

	go func() {
		defer wg.Done()

		for i := 1; i <= 5; i += 2 {
			//把1从one中取出来
			<-one
			println(i)
			//把2放入two中
			two <- (i + 1)

			if i == 5 {
				break
			}
		}

		close(one)
	}()

	go func() {
		defer wg.Done()

		for i := 2; i <= 6; i += 2 {
			//把2从two中取出来
			<-two
			println(i)
			if i == 6 {
				break
			}
			//把1放入one中
			one <- (i + 1)
		}

		close(two)
	}()

	//把1放入one中
	one <- 1
	wg.Wait()
}
//https://dbwu.tech/posts/goroutine_alternate_print/
```

这里一定把关闭有阻塞的channel，不然会死锁