# Events
> talk about the last kind of member a type can define: events.
A type that defines an event member allows the type (or instances of the type) to notify other objects that something special has happened.

  定义了一个Event，包含以下3个功能
  * 一个方法可以关注这个事件
  * 一个方法可以取消关注这个事件
  * 关注这个事件的所有方法，在事件发生时会被通知

  Event中有一个list用来存所有注册的方法，当event发生时，list中所有的方法会得到推送通知
  Event在CLR中基于delegate实现， delegate提供了callback function的功能
  作者使用一个EmailManager 接受邮件，Fax和Printer注册方法到EmailManger的event 这样的一个例子，用来详解Event

##  Designing a Type That Exposes an Event
**Step #1 定义Event中用来存放额外信息的类 EventArgs*  这个Type需要继承在System.EventArgs， 在EmailManager这个例子中，定义一个NewEmailEventArgs**
```C#
  // Step #1: Define a type that will hold any additional information that
  // should be sent to receivers of the event notification
  internal class NewMailEventArgs : EventArgs {
    private readonly String m_from, m_to, m_subject;
    public NewMailEventArgs(String from, String to, String subject) {
      m_from = from; m_to = to; m_subject = subject;
    }
    public String From { get { return m_from; } }
    public String To { get { return m_to; } }
    public String Subject { get { return m_subject; } }
  }
```
如果不需要额外的信息，可以使用EventArgs.Empty 而不要New一个EventArgs对象

**Step #2: Define the event member**
  event member 顾名思义是event类型的member，需要用event关键字来声明，同时需要用一个delegate type来声明event的method类型，下面是一个例子
 ```C#
  internal class MailManager {
    // Step #2: Define the event member
    public event EventHandler<NewMailEventArgs> NewMail;
    ...
  }
```
NewMail的类型是EventHandler<NewMailEventArgs>，也就是说注册到NewMail上的方法必须定义的跟EventHandler<NewMailEventArgs> 委托类型一样EventHandler的定义如下
``` C#
  public delegate void EventHandler<TEventArgs>(Object sender, TEventArgs e);
```
 下面是一个注册到event上的method的例子
 ```C#
  void MethodName(Object sender, NewMailEventArgs e);
 ```
 关于委托方法的第一个参数为何设计成Object，基于两方面考虑，一来是考虑到类的继承，反正都需要类型转换，干脆把参数定义为最基的类。二来是这样可以是这个方法重用性更高，而不限于父类与子类之间. 至于第二个参数 e， 第一步中提到了用于提供额外的一些信息，虽然有的时候不需要额外的信息，可能多加这个参数显得有些多余，但是这就是事件的模式，VS自动生成的一些event 方法中经常能看到这种样式的写法，遵循这一模式，可以编译开发者更容易的掌握event 
 
**Step#3  定义方法用来让event通知已经注册的方法event发生了**
  下面是实现的代码，注意这里考虑了Thread-Safe,防止NewMail在其他线程中被设置为null
  ```c#
  protected virtual void OnNewMail(NewMailEventArgs e) {
    EventHandler<NewMailEventArgs> temp = Volatile.Read(ref NewMail);
    if (temp != null) temp(this, e);
  }
  ```

**Step #4 定义一个方法，将信息传给event**
  需要定义一个方法来调用上一步中实现的方法
  ```C#
  // Step #4: Define a method that translates the
  // input into the desired event
  public void SimulateNewMail(String from, String to, String subject) {
    // Construct an object to hold the information we want
    // to pass to the receivers of our notification
    NewMailEventArgs e = new NewMailEventArgs(from, to, subject);
    // Call our virtual method notifying our object that the event
    // occurred. If no type overrides this method, our object will
    // notify all the objects that registered interest in the event
    OnNewMail(e);
  }
  ```


## How the Compiler implements an event

  编译器是如何解析Event的， 当编译器发现一个event的声明，会将这个event声明解析为3个部分
```C#
//Event
public event EventHandler<NewMailEventArgs> NewMail;

//Compiled code
// 1. A PRIVATE delegate field that is initialized to null
private EventHandler<NewMailEventArgs> NewMail = null;   // 始终声明为private ，防止外围修改委托列表

// 2. A PUBLIC add_Xxx method (where Xxx is the Event name)
// Allows methods to register interest in the event.
public void add_NewMail(EventHandler<NewMailEventArgs> value) {
  // The loop and the call to CompareExchange is all just a fancy way
  // of adding a delegate to the event in a thread-safe way
  EventHandler<NewMailEventArgs>prevHandler;
  EventHandler<NewMailEventArgs> newMail = this.NewMail;
  do {
    prevHandler = newMail;
    EventHandler<NewMailEventArgs> newHandler =
    (EventHandler<NewMailEventArgs>) Delegate.Combine(prevHandler, value);
    newMail = Interlocked.CompareExchange<EventHandler<NewMailEventArgs>>(
    ref this.NewMail, newHandler, prevHandler);
  } while (newMail != prevHandler);
}

// 3. A PUBLIC remove_Xxx method (where Xxx is the Event name)
// Allows methods to unregister interest in the event.
public void remove_NewMail(EventHandler<NewMailEventArgs> value) {
  // The loop and the call to CompareExchange is all just a fancy way
  // of removing a delegate from the event in a thread-safe way
  EventHandler<NewMailEventArgs> prevHandler;
  EventHandler<NewMailEventArgs> newMail = this.NewMail;
  do {
    prevHandler = newMail;
    EventHandler<NewMailEventArgs> newHandler =
    (EventHandler<NewMailEventArgs>) Delegate.Remove(prevHandler, value);
    newMail = Interlocked.CompareExchange<EventHandler<NewMailEventArgs>>(
    ref this.NewMail, newHandler, prevHandler);
  } while (newMail != prevHandler);
}
```
> Warning If you attempt to remove a method that was never added, then Delegate’s Remove method internally does nothing. That is, you get no exception or warning of any type; the event’s collection of methods remains unchanged.

  Remove委托方法的时候，如果remove的方法没有在列表中，不会有异常抛出。

  除了上面提到的，编译器还会在程序集的metadata中生成event definition entry，在反射中会应用到，但是在CLR中是不用的

## Designing a Type That Listens for an Event

下面的代码是注册到Event的委托所在的类的实现
```c#
    internal sealed class Fax {
      // Pass the MailManager object to the constructor
      public Fax(MailManager mm) {
        // Construct an instance of the EventHandler<NewMailEventArgs>
        // delegate that refers to our FaxMsg callback method.
        // Register our callback with MailManager's NewMail event
        mm.NewMail += FaxMsg;
      }
      // This is the method the MailManager will call
      // when a new email message arrives
      private void FaxMsg(Object sender, NewMailEventArgs e) {
        // 'sender' identifies the MailManager object in case
        // we want to communicate back to it.
        // 'e' identifies the additional event information
        // the MailManager wants to give us.
        // Normally, the code here would fax the email message.
        // This test implementation displays the info in the console
        Console.WriteLine("Faxing mail message:");
        Console.WriteLine(" From={0}, To={1}, Subject={2}", e.From, e.To, e.Subject);
      }
      // This method could be executed to have the Fax object unregister
      // itself with the NewMail event so that it no longer receives
      // notifications
      public void Unregister(MailManager mm) {
        // Unregister with MailManager's NewMail event
        mm.NewMail -= FaxMsg;
      }
    }
```
> C# requires your code to use the += and -= operators to add and remove delegates from the list. If you try to call the add or remove method explicitly, the C# compiler produces the CS0571 cannot explicitly call operator or accessor error message

C#中需要使用 += 和  -= 来添加和删除委托，不能显示的使用 add_Xxx 和 remove_xxx , 但是其他不支持Event的编程语言没有重载 += 和 -= 的话，可以直接调用 add 和 remove方法

## Explicitly Implementing an Event

  上面提到了C#在解析event的时候，要生成很多的代码包括一个私有的委托，两个accessor method，而在C# winform以及asp.net等技术中，每个类都会有很多events而且这些event有可能是程序员不会调用的，如果是为每个event都单独声明，那么编译器编译出来的代码会很多，造成不必要的损耗，为了解决这一问题，C#内部提供了 Explicitly Implementng Event的模式。
  其实这个显示实现就是为event显式的定义add和remove的accessor methods，而不是由编译器自动来生成，但是这样跟自动生成相比会少一个委托的定义，这时候就需要借助于其他的方式来实现，定义一个EventSet 类，将所有的事件都在一个dictionary<eventKey,eventDelegate>中保存，eventKey用来唯一标识event， eventDelegate对应的是event的委托方法，可以对此delegate进行删除添加方法，也可以根据给定的eventKey调用对应的delegate

  *显式的实现event的add/remove 方法可以减少编译器生成的中间代码量，尤其是需要多个event的时候效果尤其显著*
  使用这种方式重新定义NewMail
  ```C#
  //m_eventSet 是用于add/remove delegate的工具类
  private EventSet m_eventSet= new EventSet();   //使用EventSet 可以在这个Dictionary中保存多个event， 每一个event都可以按照newMail的实现方式，定义1.eventkey，2.event， 3.event触发方法
  #region New Mail event, if more events needed, repeat these steps
  private EventKey newMail_EventKey = new EventKey();
  public event EventHandler<NewMailEventArgs> NewMail
  {
    add { m_eventSet.Add(newMail_EventKey, value); }
    remove { m_eventSet.Remove(newMail_EventKey, value); }
  }
  public void RaiseNewMail(NewMailEventArgs args)  
  {
    m_eventSet.Raise(newMail_EventKey,args);
  }
  #endregion

  // EventSet定义如下，　其中考虑到了Thread-Safe的情况，是与微软原生的实现不同的地方
  // This class exists to provide a bit more type safety and
  // code maintainability when using EventSet
  public sealed class EventKey { }

  public sealed class EventSet {
    // The private dictionary used to maintain EventKey -> Delegate mappings
    private readonly Dictionary<EventKey, Delegate> m_events =   new Dictionary<EventKey, Delegate>();
    // Adds an EventKey -> Delegate mapping if it doesn't exist or
    // combines a delegate to an existing EventKey
    public void Add(EventKey eventKey, Delegate handler) {
      Monitor.Enter(m_events);
      Delegate d;
      m_events.TryGetValue(eventKey, out d);
      m_events[eventKey] = Delegate.Combine(d, handler);
      Monitor.Exit(m_events);
    }
    // Removes a delegate from an EventKey (if it exists) and
    // removes the EventKey -> Delegate mapping the last delegate is removed
    public void Remove(EventKey eventKey, Delegate handler) {
      Monitor.Enter(m_events);
      // Call TryGetValue to ensure that an exception is not thrown if
      // attempting to remove a delegate from an EventKey not in the set
      Delegate d;
      if (m_events.TryGetValue(eventKey, out d)) {
        d = Delegate.Remove(d, handler);
        // If a delegate remains, set the new head else remove the EventKey
        if (d != null) m_events[eventKey] = d;
        else m_events.Remove(eventKey);
      }
      Monitor.Exit(m_events);
    }
    // Raises the event for the indicated EventKey
    public void Raise(EventKey eventKey, Object sender, EventArgs e) {
      // Don't throw an exception if the EventKey is not in the set  Delegate d;
      Monitor.Enter(m_events);
      m_events.TryGetValue(eventKey, out d);
      Monitor.Exit(m_events);
      if (d != null) {
        // Because the dictionary can contain several different delegate types,
        // it is impossible to construct a type-safe call to the delegate at
        // compile time. So, I call the System.Delegate type's DynamicInvoke
        // method, passing it the callback method's parameters as an array of
        // objects. Internally, DynamicInvoke will check the type safety of the
        // parameters with the callback method being called and call the method.
        // If there is a type mismatch, then DynamicInvoke will throw an exception.
        d.DynamicInvoke(new Object[] { sender, e });
      }
    }
  }  
  ```

  对于其他要注册到NewMail事件上的类，它们是不知道event是隐式的通过编译器实现的或者是程序员自己显式的实现的。因为注册或者删除的方法还是使用 += / -=
