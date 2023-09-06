

# Giới Thiệu: callbacks

```warn header="Chúng ta dùng Browser Methods ở những ví dụ này"
To demonstrate the use of callbacks, promises and other abstract concepts, we'll be using some browser methods: specifically, loading scripts and performing simple document manipulations.

If you're not familiar with these methods, and their usage in the examples is confusing, you may want to read a few chapters from the [next part](/document) of the tutorial.

Although, we'll try to make things clear anyway. There won't be anything really complex browser-wise.
```

Nhiều hàm được cung cấp bởi môi trường máy chủ JavaScript cho phép bạn lên lịch các hành động *không đồng bộ*. Nói cách khác, hành động bắt đầu bây giờ, nhưng được kết thúc sau đó.

Ví dụ, hàm `setTimeout`.

Có nhiều ví dụ thực tế khác về hành động bất đồng bộ, vd: Tải scripts và modules (Sẽ nói ở phần sau).

Nói một chút về hàm `loadScript(src)`, nó tải một script được cho bởi `src`:

```js
function loadScript(src) {
  // Tạo một thẻ <script> và gắn vào trang web
  // this causes the script with given src to start loading and run when complete
  let script = document.createElement('script');
  script.src = src;
  document.head.append(script);
}
```

It appends to the document the new, dynamically created, tag `<script src="…">` with given `src`. The browser automatically starts loading it and executes when complete.

Ta có thể sử dụng hàm này như sau:

```js
// Tải và thực thi script ở đường dẫn được cung cấp
loadScript('/my/script.js');
```

Đoạn script được thực thi "bất đồng bộ", nó bắt đầu tải lúc này, nhưng chạy sau đó, khi mà hàm đã kết thúc.

Nếu có bất kì đoạn code nào ở sau `loadScript(…)`, thì nó không cần chờ đoạn script kết thúc.

```js
loadScript('/my/script.js');
// the code below loadScript
// doesn't wait for the script loading to finish
// ...
```

Chúng ta muốn dùng đoạn script đó ngay sau khi nó được tải. Trong đoạn script, có một hàm tên là new functions, và ta muốn chạy nó.

nhưng nếu ta gọi hàm new functions sau khi gọi hàm `loadScript(…)`, nó không chạy đâu:

```js
loadScript('/my/script.js'); // the script has "function newFunction() {…}"

*!*
newFunction(); // đây là hàm nào vậy? tôi không biết!
*/!*
```

Một cách bình thường, Trình duyệt có lẽ không có thời gian để tải đoạn script. Hiện tại, hàm `loadScript` không hỗ trợ cách để kiểm tra xem quá trình tải đã hoàn thành chưa. Đoạn script sẽ được tải, tải xong thì chạy. Nhưng chúng ta muốn biết khi nào nó xong, để mà chúng ta có thể dùng hàm new funtion trong đó.

Hãy thêm một hàm `callback` như một tham số thứ hai của hàm `loadScript`, hàm này là hàm mà ta muốn thực thi sau khi script được tải xong:

```js
function loadScript(src, *!*callback*/!*) {
  let script = document.createElement('script');
  script.src = src;

*!*
  script.onload = () => callback(script);
*/!*

  document.head.append(script);
}
```

Now if we want to call new functions from the script, we should write that in the callback:

```js
loadScript('/my/script.js', function() {
  // the callback runs after the script is loaded
  newFunction(); // so now it works
  ...
});
```

That's the idea: the second argument is a function (usually anonymous) that runs when the action is completed.

Here's a runnable example with a real script:

```js run
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;
  script.onload = () => callback(script);
  document.head.append(script);
}

*!*
loadScript('https://cdnjs.cloudflare.com/ajax/libs/lodash.js/3.2.0/lodash.js', script => {
  alert(`Cool, the script ${script.src} is loaded`);
  alert( _ ); // function declared in the loaded script
});
*/!*
```

That's called a "callback-based" style of asynchronous programming. A function that does something asynchronously should provide a `callback` argument where we put the function to run after it's complete.

Here we did it in `loadScript`, but of course it's a general approach.

## Callback in callback

How can we load two scripts sequentially: the first one, and then the second one after it?

The natural solution would be to put the second `loadScript` call inside the callback, like this:

```js
loadScript('/my/script.js', function(script) {

  alert(`Cool, the ${script.src} is loaded, let's load one more`);

*!*
  loadScript('/my/script2.js', function(script) {
    alert(`Cool, the second script is loaded`);
  });
*/!*

});
```

After the outer `loadScript` is complete, the callback initiates the inner one.

What if we want one more script...?

```js
loadScript('/my/script.js', function(script) {

  loadScript('/my/script2.js', function(script) {

*!*
    loadScript('/my/script3.js', function(script) {
      // ...continue after all scripts are loaded
    });
*/!*

  });

});
```

So, every new action is inside a callback. That's fine for few actions, but not good for many, so we'll see other variants soon.

## Khiểm soát lỗi

Ở các ví dụ trên, chúng ta đã không hề xem xét đến lỗi. Chuyện gì xảy ra nếu quá trình tải script bị lỗi? Our callback should be able to react on that.

Đây là phiên bản cải tiến của`loadScript`, có bao gồm kiểm tra lỗi:

```js
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;

*!*
  script.onload = () => callback(null, script);
  script.onerror = () => callback(new Error(`Script load error for ${src}`));
*/!*

  document.head.append(script);
}
```

Gọi `callback(null, script)` khi tải thành công và ngược lại gọi `callback(error)` .

The usage:
```js
loadScript('/my/script.js', function(error, script) {
  if (error) {
    // handle error
  } else {
    // script loaded successfully
  }
});
```

Once again, the recipe that we used for `loadScript` is actually quite common. It's called the "error-first callback" style.

Điểm mạnh:
1. Tham số đầu tiên của `callback` is reserved for an error if it occurs. Then `callback(err)` is called.
2. The second argument (and the next ones if needed) are for the successful result. Then `callback(null, result1, result2…)` is called.

So the single `callback` function is used both for reporting errors and passing back results.

## Pyramid of Doom

From the first look, it's a viable way of asynchronous coding. And indeed it is. For one or maybe two nested calls it looks fine.

But for multiple asynchronous actions that follow one after another we'll have code like this:

```js
loadScript('1.js', function(error, script) {

  if (error) {
    handleError(error);
  } else {
    // ...
    loadScript('2.js', function(error, script) {
      if (error) {
        handleError(error);
      } else {
        // ...
        loadScript('3.js', function(error, script) {
          if (error) {
            handleError(error);
          } else {
  *!*
            // ...continue after all scripts are loaded (*)
  */!*
          }
        });

      }
    });
  }
});
```

In the code above:
1. We load `1.js`, then if there's no error.
2. We load `2.js`, then if there's no error.
3. We load `3.js`, then if there's no error -- do something else `(*)`.

As calls become more nested, the code becomes deeper and increasingly more difficult to manage, especially if we have real code instead of `...` that may include more loops, conditional statements and so on.

That's sometimes called "callback hell" or "pyramid of doom."

<!--
loadScript('1.js', function(error, script) {
  if (error) {
    handleError(error);
  } else {
    // ...
    loadScript('2.js', function(error, script) {
      if (error) {
        handleError(error);
      } else {
        // ...
        loadScript('3.js', function(error, script) {
          if (error) {
            handleError(error);
          } else {
            // ...
          }
        });
      }
    });
  }
});
-->

![](callback-hell.svg)

The "pyramid" of nested calls grows to the right with every asynchronous action. Soon it spirals out of control.

So this way of coding isn't very good.

We can try to alleviate the problem by making every action a standalone function, like this:

```js
loadScript('1.js', step1);

function step1(error, script) {
  if (error) {
    handleError(error);
  } else {
    // ...
    loadScript('2.js', step2);
  }
}

function step2(error, script) {
  if (error) {
    handleError(error);
  } else {
    // ...
    loadScript('3.js', step3);
  }
}

function step3(error, script) {
  if (error) {
    handleError(error);
  } else {
    // ...continue after all scripts are loaded (*)
  }
}
```

See? It does the same, and there's no deep nesting now because we made every action a separate top-level function.

It works, but the code looks like a torn apart spreadsheet. It's difficult to read, and you probably noticed that one needs to eye-jump between pieces while reading it. That's inconvenient, especially if the reader is not familiar with the code and doesn't know where to eye-jump.

Also, the functions named `step*` are all of single use, they are created only to avoid the "pyramid of doom." No one is going to reuse them outside of the action chain. So there's a bit of namespace cluttering here.

We'd like to have something better.

Luckily, there are other ways to avoid such pyramids. One of the best ways is to use "promises," described in the next chapter.
