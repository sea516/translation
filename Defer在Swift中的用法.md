## Defer在Swift中的用法
虽然defer关键字在Swift2.0中已经被引入，但是在项目中我们用的还是很少。它的用法有时候很难理解，但是在特定的地方使用会使得你的代码变得优雅。

> *最常见的用法是用在开始或者结束一个上下文。*

### 它是如何工作的？
一个defer语句会发一个方法退出之前执行其中的代码。

```
func updateImage() {
    defer { print("Did update image") }

    print("Will update image")
    imageView.image = updatedImage
}
// Will update Image
// Did update image
```

### 多个defer语句的执行顺序
如果多个defer语句出现在同一个方法，它们的出现顺序与执行顺序恰恰相反。最后定义的语句反而是第一个要执行的语句，下面的例子将通过打印数字的逻辑顺序来证明
```
func printStringNumbers() {
    defer { print("1") }
    defer { print("2") }
    defer { print("3") }
    print("4")
}
/// Prints 4, 3, 2, 1
```

### 一个通用的场景

最常见的用法是开启和结束一个方法，例如在处理文件访问时。访问完成后，需要关闭文件句柄。你可以感觉到defer语句的好处，这样你就不会担心忘记这样做一件事情，它就像一个清道夫一样。

```
func writeFile() {
    let file: FileHandle? = FileHandle(forReadingAtPath: filepath)
    defer { file?.closeFile() }

    // Write changes to the file
}
```

### 确保回调
defer语句有一个更高级用法：在一个完成回调中确保有返回结果值。这会非常方便，因为我们时常会忘记触发这个回调
```
func getData(completion: (_ result: Result<String>) -> Void) {
    var result: Result<String>?
    defer {
        guard let result = result else {
            fatalError("We should always end with a result")
        }
        completion(result)
    }
    // Generate the result..
}
```

defer语句会永远确保执行完成处理并验证结果值。一旦结果值为空，就会触发fatalError，程序就会发生异常或者失败
