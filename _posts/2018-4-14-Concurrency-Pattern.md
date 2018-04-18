---
layout: post
title:  "Concurrency pattern"
date:   2018-4-14 16:24:00
categories: C++,javascript,go,concurrency
---

In this paper, we disucss the concurrency pattern changed from callback, promise/future, to async/await and channel. Many program language supoort these patterns partially or fully. We will study the key idea across C++, Javascript, Go etc and focus on **WHAT** and **WHY**.

Parrtern | Language
-------- | --------
CallBack | C++
Promise/future | Javascript
Async/await | C#
Channel | Go

# Callback
Before talked about callback, give an example about push/pull. in some extent, callback is push.
```C++
// main thread
auto result = sendTasktoWorkThread(task); // execute in other thread
doOtherthing();
auto value = result.get(); // [pull the result]
doRemainedthing(value);
```

```C++
// when task done, call callback in worker thread directly
// or send event to main thread which execute call callback then [push the result]
sendTasktoWorkThread(task, callback); 
doOtherthing();
eventloop();
```
Let's see an example involved register many callbacks.
```c++
#include <iostream>

#define DEFINE_STEP_N(n) \
template <typename F> \
void step##n(F f) { \
    std::cout << n << std::endl; \
    f(); \
}

DEFINE_STEP_N(1)
DEFINE_STEP_N(2)
DEFINE_STEP_N(3)
DEFINE_STEP_N(4)
DEFINE_STEP_N(5)

void fun() {
    std::cout << "last" << std::endl;
}

int main() {
   step1([]{
        step2([]{
            step3([]{
                step4([] {
                    step5(fun);
                });
            });
        });
   });
}

// output
1
2
3
4
5
last 
```
## Prons and cons:
- callback hell as previous example show: 
  it's really counter-intuitive thing to register callback.
- lifecyle manage: 
  need keep the object involved in callback not destructed before call in lanugage not supporting garbage collect.
- executor/thread schedule:
  schedule the callback in original main thread, or in current worker thread. If run in current worker thread, should have additional mutex to avoid data race in the callback.
  
# Promise/future
```html
<html>
<head> 
<script>
function taskA() {
    console.log("Task A");
    throw new Error("throw Error Task A")
}
function taskB() {
    console.log("Task B");
}
function onRejected(error) {
    console.log("Catch Error: ", error);
}
function finalTask() {
    console.log("Final Task");
    throw new Error("throw Error Final task")
}

var promiseB = new Promise((resolve, reject) => reject(new Error('error msg')))
promiseB
.then(success => console.log('onfulfilled ', success), err => console.log('onRejected ', err))
.then(taskA)
.then(taskB)
.catch(onRejected)
.then(finalTask);
</script>
</head>
<body>
</body>
</html>
```
In addition, C++ has a standard implement, Facebook also has [folly/futures](https://github.com/facebook/folly/blob/master/folly/futures/Future.h). 
![folly/future](https://github.com/pzhp/pzhp.github.io/blob/master/images/promise_future.png)
[example](https://www.oschina.net/translate/futures-for-c-11-at-facebook)

## Pros and cons:
- Simplify the process on register then functions, easily chain then functions.
- Simplify the communication between main thread and worker thread. Promise set value/exception, future get value/exception.
- When then functions need multi arguments, it's hard to write, as follow example:
```javascript
function takeLongTime(n) {
    return new Promise(resolve => {
        // setTimeout mock time consuming task.
        // after n, will call the funtion
        setTimeout(() => resolve(n + 200), n);
    });
}

function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}

function step2(n) {
    console.log(`step2 with ${n}`);
    return takeLongTime(n);
}

function step3(n) {
    console.log(`step3 with ${n}`);
    return takeLongTime(n);
}

// serialize execute: step1 => step2 => step3 
function doIt() {
    console.time("doIt");
    const time1 = 600;
    step1(time1)
        .then(time2 => step2(time2))
        .then(time3 => step3(time3))
        .then(result => {
            console.log(`result is ${result}`);
            console.timeEnd("doIt");
        });
}

doIt();
```

# Async/await
async/await pattern comes from C#[Async and Await](https://blog.stephencleary.com/2012/02/async-and-await.html). Javascript ES7 support it [javascript promise and async/await](https://segmentfault.com/a/1190000007535316). There is a [proposal](https://isocpp.org/files/papers/N3858.pdf) about it in C++.


Javascript async/await features:
In some extent, async/await is wrapper for promise/future pattern.
Async function return a Promise.
Await only used in async function.

## Prons and cons
- Use sync logic to write async code
- The continuation is scheduled on specific context. Check it in [C# Async and await](https://blog.stephencleary.com/2012/02/async-and-await.html). Here is a short summary
>In the overview, I mentioned that when you await a built-in awaitable, then the awaitable will capture the current “context” and later >apply it to the remainder of the async method. What exactly is that “context”?
>Simple answer:
>
>If you’re on a UI thread, then it’s a UI context.
>If you’re responding to an ASP.NET request, then it’s an ASP.NET request context.
>Otherwise, it’s usually a thread pool context.
>Complex answer:
>
>If SynchronizationContext.Current is not null, then it’s the current SynchronizationContext. (UI and ASP.NET request contexts are >SynchronizationContext contexts).
>Otherwise, it’s the current TaskScheduler (TaskScheduler.Default is the thread pool context).
```C#
// WinForms example (it works exactly the same for WPF).
private async void DownloadFileButton_Click(object sender, EventArgs e)
{
  // Since we asynchronously wait, the UI thread is not blocked by the file download.
  await DownloadFileAsync(fileNameTextBox.Text);

  // Since we resume on the UI context, we can directly access UI elements.
  resultTextBox.Text = "File downloaded!";
}

private async Task DownloadFileAsync(string fileName)
{
  // Use HttpClient or whatever to download the file contents.
  var fileContents = await DownloadFileContentsAsync(fileName).ConfigureAwait(false);

  // Note that because of the ConfigureAwait(false), we are not on the original context here.
  // Instead, we're running on the thread pool.

  // Write the file contents out to a disk file.
  await WriteToDiskAsync(fileName, fileContents).ConfigureAwait(false);

  // The second call to ConfigureAwait(false) is not *required*, but it is Good Practice.
}
```

# Channel
``` Go
package main
import "fmt"
import "time"

func main() {
    c1 := make(chan string)
    go func() {
        c1 <- "ping from 1"
    }()

    c2 := make(chan int)
    go func() {
        c2 <- 2
    }()

    // time.Sleep(100 * time.Millisecond)
    // case is chosen randomly
    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-c1:
            fmt.Println(msg1)
        case msg2 := <-c2:
            fmt.Println(msg2)
        case <-time.After(time.Second * 30):
            fmt.Println("timeOut")
            // default: // if no default, block
            //  fmt.Println("non block")
        }
    }
}
```

``` C++
// consider use c++11 implement previous code
std::future<std::string> f = std::async(std::launch::async, []() { 
	return dotask();
	})
std::string ret = f.get(); // block
// f.wait_for(); // support time out
```
C++ future only support set value once, but channel support tranfer data like a stream.

# Summary
![Outline](https://github.com/pzhp/pzhp.github.io/blob/master/images/concurrency_pattern.png)

remained issues: executor/thread schedule, exception etc.


