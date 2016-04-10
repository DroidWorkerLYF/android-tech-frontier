使用RxBinding进行响应式Android UI开发
---

> * 原文链接 : [Reactive Android UI Programming with RxBinding](https://realm.io/news/donn-felker-reactive-android-ui-programming-with-rxbinding/)
* 原文作者 : [Donn Felker](https://www.linkedin.com/shareArticle?mini=true&url=https%3A%2F%2Frealm.io%2Fnews%2Fdonn-felker-reactive-android-ui-programming-with-rxbinding%2F&title=Reactive%20Android%20UI%20Programming%20with%20RxBinding%2C%20with%20Donn%20Felker)
* 译文出自 : [开发技术前线 www.devtf.cn](http://www.devtf.cn)
* 转载声明: 本译文已授权[开发者头条](http://toutiao.io/download)享有独家转载权，未经允许，不得转载!
* 译者 : [DroidWorkerLYF](https://github.com/DroidWorkerLYF) 
* 校对者: [这里校对者的github用户名](https://github.com/)  
* 状态 :  未完成

There’s an old saying that rings true in software…  
在软件开发中有一件俗话说的很对

> “The only thing that is constant is change.”
> "唯一不变的变化"

The same can easily be said of Android. For example, how many times have
you found yourself implementing a click listener, a text change listener
,or some other mundane callback that has a different signature? Android
Studio alleviates us from having to memorize the callbacks, listeners,
and their signatures, but it unfortunately does not provide any
uniformity. After a while, your code seems to resemble a mess of
anonymous classes that are storing state in a field in the fragment or
activity. This is further complicated if you need to achieve a more
reactive architecture where inputs of one widget/view are based off the
outputs of another view (or its actions). For most devs, implementing
this kind of reactive callback chaining yourself will prove a
time-consuming and error-prone mess. Thankfully, the easy-to-use
RxBinding libraries can help.  
Android开发也是如此。举个列子，你有多少次实现了不同签名的click listener，text change listener或者其他的listener？Android Studio缓解了我们需要记住回调，监听，但不幸的是没有提供任何的统一。

### What are the RxBinding Libraries?
### RxBinding是什么？

RxBinding is a set of libraries that allow you to react to user
interface events via the RxJava paradigm. Let’s take a look at a few
examples. This is how Android developers typically react to a button
click event:  
RxBinding是一系列允许你通过RxJava方式来响应用户操作的集合。让我们看些例子先，如下是Android开发者典型的处理button点击事件的方式：

```
Button b = (Button)findViewById(R.id.button);
b.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
              // do some work here     
            }
        });
```

Using `RxBinding`, you can accomplish the same thing
but with RxJava subscription:  
使用`RxBinding`，你可以使用RxJava的subscription达到同样的目的：


```
Button b = (Button)findViewById(R.id.button);
Subscription buttonSub =
                RxView.clicks(b).subscribe(new Action1<Void>() {
                    @Override
                    public void call(Void aVoid) {
                        // do some work here
                    }
                });
// make sure to unsubscribe the subscription
```

Let’s take a look at another example, this time with a text change
listener for an EditText:  
再看一个例子，这回事为EditText设置text change listener：

``` 
final EditText name = (EditText) v.findViewById(R.id.name);
name.addTextChangedListener(new TextWatcher() {
    @Override
    public void beforeTextChanged(CharSequence s, int start, int count, int after) {
        
    }

    @Override
    public void onTextChanged(CharSequence s, int start, int before, int count) {
        // do some work here with the updated text
    }

    @Override
    public void afterTextChanged(Editable s) {

    }
});
```

Now the same thing, but written with RxBinding support:  
同样的，用RxBinding来处理：

```
final EditText name = (EditText) v.findViewById(R.id.name);
Subscription editTextSub =
    RxTextView.textChanges(name)
            .subscribe(new Action1<String>() {
                @Override
                public void call(String value) {
                    // do some work with the updated text
                }
            });
// Make sure to unsubscribe the subscription
```

While this may seem like trading an apple for an orange, it actually
gives you something very different: consistency. These are merely two
examples of countless listeners and callbacks that are available via
Android’s various views and widgets that show how they differ in the
traditional sense of an Android implementation vs the RxBinding
implementation. When using RxBinding, you have the same consistent
implementation: an RxJava subscription. This offers less cognitive load
as well as all of the other benefits of RxJava.  
虽然这看起来可能是半斤八两，但实际上带给了你非常不一样的地方：统一性。以上只是Android各种视图和插件中无数监听和回调的两个例子，用来像你展示Android实现和RxBinding实现的区别。当你使用RxBinding时，你可以通过RxJava的subscription获得统一的实现方式。这提供了较小的认知负荷以及RxJava的其他好处。

### More Granular Control
### 更细粒度的控制

In the example above, I use the
`RxTextView.textChanges()` method to react only to
the text change event. In traditional Android, we have to implement a
full TextWatcher to accomplish this, wasting multiple lines of code
because the `beforeTextChanged` event and
`afterTextChanged` callbacks must also be
implemented. This dead code simply pollutes the class and offers no
additional value. With `RxBinding` I can achieve
more granular control over what I want to react to in my application
without the additional overhead in my codebase.  
在上面的例子中，我使用`RxTextView.textChanges()`方法来响应文字变换事件。在传统的Android开发中，我们需要实现完整的TextWatcher来做到这点，浪费了很多行代码就因为`beforeTextChanged`和`afterTextChanged`回调必须必实现。这些死代码污染了整个类并且不提供任何价值。使用`RxBinding`我可以在更细节的层面上控制我想响应什么操作，而不需要额外的负担。

It is important to note that in this case
`RxBinding` is simply implementing a
`TextWatcher` for you and only calling the
`onTextChanged` event for you. Here’s the
implementation of the `TextViewTextOnSubscribe`
class that `RxBinding` uses behind the scenes for
you in the `RxTextView.textChanges()` observable:  
需要注意的是这里`RxBinding`只是简单的实现了`TextWatcher`并且只调用`onTextChanged`事件。下面是`TextViewTextOnSubscribe`的实现：


```
final class TextViewTextOnSubscribe implements Observable.OnSubscribe<CharSequence> {
  final TextView view;

  TextViewTextOnSubscribe(TextView view) {
    this.view = view;
  }

  @Override public void call(final Subscriber<? super CharSequence> subscriber) {
    checkUiThread();

    final TextWatcher watcher = new TextWatcher() {
      @Override public void beforeTextChanged(CharSequence s, int start, int count, int after) {
      }

      @Override public void onTextChanged(CharSequence s, int start, int before, int count) {
        if (!subscriber.isUnsubscribed()) {
          subscriber.onNext(s);
        }
      }

      @Override public void afterTextChanged(Editable s) {
      }
    };
    view.addTextChangedListener(watcher);

    subscriber.add(new MainThreadSubscription() {
      @Override protected void onUnsubscribe() {
        view.removeTextChangedListener(watcher);
      }
    });

    // Emit initial value.
    subscriber.onNext(view.getText());
  }
}
```

The real benefit here is the syntactic sugar over the existing Android
API’s that make your code more readable and consistent. Regardless if
you’re observing a click event, a text change event, or even a dismissal
of a Snackbar, RxBinding provides a consistent implementation that you
can use to react accordingly.

### Transforming with Operators

Since `RxBinding` is applying RxJava paradigms to
existing Android Views and Widgets, you can use RxJava operators to
perform transformations on the stream of data that is emitted in real
time. Let’s take a look at an example:

Assume you have an EditText field that you want to watch for changes (as
the user types, etc). When the text changes, you want to take the text
string, reverse it, and output it to the screen in a different TextView.
Here’s how you could do that:

```
final TextView nameLabel = (TextView) findViewById(R.id.name_label);

final EditText name = (EditText) findViewById(R.id.name);
Subscription editTextSub =
    RxTextView.textChanges(name)
            .map(new Func1<CharSequence, String>() {
                @Override
                public String call(CharSequence charSequence) {
                    return new StringBuilder(charSequence).reverse().toString();
                }
            })
            .subscribe(new Action1<String>() {
                @Override
                public void call(String value) {
                    nameLabel.setText(value);
                }
            });
```

In the example above, the EditText text changed event is observed via
the `RxTextView.textChanges()` observable, which is
in turn mapped to a string via the `map()` operator.
The `map()` operator returns a reversed string and
then the subscription sets the `nameLabel` to that
text value. As you can imagine, you can do a number of things with built
in RxJava operators and any custom operators that you’ve built thus far
in your application.

I’d like to mention again the power of the syntactic sugar over the
Android views and widgets API. By conforming to the same RxJava
Observable paradigm, you can chain operations together that would
normally not be able to. This has great power as you start to compose
your application in a reactive nature.

### RxBinding Offers More

There have been a few occasions where I needed to support multiple click
listeners on a view (for various reasons). As you probably know, this is
not possible in Android unless you write some custom code to handle it.
Supporting multiple listeners with RxBinding is very simple. It’s
important to note that RxBinding does not enable this, but it is enabled
through RxJava operators like
[`publish()`](http://reactivex.io/documentation/operators/publish.html),
[`share()`](http://reactivex.io/RxJava/javadoc/rx/Observable.html#share()),
and
[`replay()`](http://reactivex.io/documentation/operators/replay.html).
It’s up to you to decide which one suits your needs. In this example,
I’m going to use the `share()` operator to enable
multicast listener support (multiple click listeners):

```
Button b = (Button) v.findViewById(R.id.do_magic);
Observable<Void> clickObservable = RxView.clicks(b).share();

Subscription buttonSub =
        clickObservable.subscribe(new Action1<Void>() {
            @Override
            public void call(Void aVoid) {
                // button was clicked. 
            }
        });
compositeSubscription.add(buttonSub);

Subscription loggingSub =
        clickObservable.subscribe(new Action1<Void>() {
            @Override
            public void call(Void aVoid) {
                // Button was clicked
            }
        });
compositeSubscription.add(loggingSub);
```

If you were to remove the `.share()` call from the
above code, only the last subscription would get called. As the
`share()` operator [docs
state](http://reactivex.io/RxJava/javadoc/rx/Observable.html#share()):

> “Returns a new Observable that multicasts (shares) the original.”

Using `share` in this context allows multiple click
listeners to be applied to one button, which is very powerful.

### RxBinding Idioms and Installation

There are a few things to be aware of when working with RxBinding.

First, weak references should not be used - as per the docs:

> “Weak references should not be used. RxJava’s subscription graph
> allows for proper garbage collections of reference-holding objects
> provided the caller unsubscribes.”

Secondly, in various parts of the Android framework ,the UI events emit
multiple values instead of a single argument like a click listener
(where view is the only argument). RxJava observables only emit a single
object, not an array of values. Therefore, a wrapper object is needed to
combine these values into a single object in these instances. For
example, the scroll change listener returns multiple values such as
`scrollX`, `scrollY`,
`oldScrollX`, `oldScrollY`. In
`RxBinding` these values are combined into a wrapper
object called `ViewScrollChangeEvent`
([source](https://github.com/JakeWharton/RxBinding/blob/master/rxbinding/src/main/java/com/jakewharton/rxbinding/view/ViewScrollChangeEvent.java)).
When the `RxView.scrollChangeEvents()` observable is
subscribed to, the `ViewScrollChangeEvent` will be
the value emitted from the `onNext` method
([source](https://github.com/JakeWharton/RxBinding/blob/master/rxbinding/src/main/java/com/jakewharton/rxbinding/view/ViewScrollChangeEventOnSubscribe.java)).
Therefore, you will get the `ViewScrollChangeEvent` 
which contains the values you’re interested in.

Third, each library is broken down based upon where it is in the Android
platform. For example, views and widgets, which are in the
`android.widget.*` package, will be found in the
`com.jakewharton.rxbinding.widget.*` package.

RxBinding is not limited to the platform classes either. There are
RxBinding libraries for the support libraries as well. For example the
basic RxBinding support for platform classes is found by using this
depdendency:


```
compile 'com.jakewharton.rxbinding:rxbinding:0.4.0'
```

Let’s assume that you are using the design support library and you want
the RxBinding goodness there as well. You can also include this
dependency to get those bits as well:

``` 
compile 'com.jakewharton.rxbinding:rxbinding-design:0.4.0'
```

Furthermore, let’s assume that you’re now using [Kotlin on
Android](https://realm.io/news/getting-started-with-kotlin-and-anko/),
and you would like RxBindings for Kotlin as well. Simply tack on
`-kotlin` to any of the dependencies to get the
Kotlin version. e.g. -

``` {.highlight}
compile 'com.jakewharton.rxbinding:rxbinding-kotlin:0.4.0'
```

### Expand or Introduce Your RxJava Toolbox

If you haven’t yet embarked on your RxJava journey, RxBinding may be the
gateway that hooks you. If you’re already hooked on RxJava, this is a
great supplement to the regular classes. RxBinding simple to use,
provides a consistent API for consumption, and makes your application
much more composable and reactive.

Happy programming!

[Download the source](https://github.com/donnfelker/RxBindingsIntro)