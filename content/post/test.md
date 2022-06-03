---
title: "Test"
date: 2022-06-02T16:18:11+08:00
draft: false
toc: true
---
`[toc]`

## 引言

[`testify`](https://link.segmentfault.com/?enc=xtHQ4mgsuyRCjK%2FpPWyNRA%3D%3D.LO5M8ag51CJ5ibQMWbnYZ556X7UY4AsHxYMxb3KFE9lJKV93bDojIFXyjpnk%2F%2Fkl)可以说是最流行的Go 语言测试库了。提供了很多方便的函数帮助我们做 `assert`和错误信息输出。使用标准库 `testing`，我们需要自己编写各种条件判断，根据判断结果决定输出对应的信息。

`testify`核心有三部分内容：

* `assert`：断言；
* `mock`：测试替身；
* `suite`：测试套件。

## 引入

**安装** `testify`库：

```
$ go get -u github.com/stretchr/testify
```

## assert

`ssert`子库提供了便捷的**断言**函数，可以大大简化测试代码的编写。总的来说，它将之前需要 `判断 + 信息输出的模式`：

```go
if got != expected {
  t.Errorf("Xxx failed expect:%d got:%d", got, expected)
}
```

**简化为一行****断言**代码：

```
assert.Equal(t, got, expected, "they should be equal")
```

**结构更清晰，更可读。熟悉其他语言测试框架的开发者对** `assert`的相关用法应该不会陌生。此外，`assert`中的函数会自动生成比较清晰的错误描述信息：

```
func TestEqual(t *testing.T) {
  var a = 100
  var b = 200
  assert.Equal(t, a, b, "this is test")
}
```

**使用** `testify`编写测试代码与 `testing`一样，测试文件为 `_test.go`，测试函数为 `TestXxx`。使用 `go test`命令运行测试：

```
$ go test
--- FAIL: TestEqual (0.00s)
    assert_test.go:12:
                Error Trace:
                Error:          Not equal:
                                expected: 100
                                actual  : 200
                Test:           TestEqual
   Messages:      this is test
FAIL
exit status 1
FAIL    github.com/darjun/go-daily-lib/testify/assert   0.107s
```
