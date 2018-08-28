异常的继承结构
```java
Throwable
      -----Error
      -----Exception
            ---------RuntimeException
            ---------IOException
```

其中的Error和RuntimeException是非检查型的异常。其他的异常(主要指的是IOException和自定义的Exception的子类)必须异常检查

## Throwable分析
```java

public Throwable() {
        fillInStackTrace();
}

public Throwable(String message) {
        fillInStackTrace();
        detailMessage = message;
    }
    
public Throwable(String message, Throwable cause) {
        fillInStackTrace();
        detailMessage = message;
        this.cause = cause;
}

public Throwable(Throwable cause) {
        fillInStackTrace();
        detailMessage = (cause==null ? null : cause.toString());
        this.cause = cause;
}

public String getMessage() {
        return detailMessage;
}

public String getLocalizedMessage() {
        return getMessage();
}

public String toString() {
    String s = getClass().getName();
    String message = getLocalizedMessage();
    return (message != null) ? (s + ": " + message) : s;
}

private void printStackTrace(PrintStreamOrWriter s) {
        // Guard against malicious overrides of Throwable.equals by
        // using a Set with identity equality semantics.
        Set<Throwable> dejaVu =
            Collections.newSetFromMap(new IdentityHashMap<Throwable, Boolean>());
        dejaVu.add(this);

        synchronized (s.lock()) {
            // Print our stack trace
            s.println(this);
            StackTraceElement[] trace = getOurStackTrace();
            for (StackTraceElement traceElement : trace)
                s.println("\tat " + traceElement);

            // Print suppressed exceptions, if any
            for (Throwable se : getSuppressed())
                se.printEnclosedStackTrace(s, trace, SUPPRESSED_CAPTION, "\t", dejaVu);

            // Print cause, if any
            Throwable ourCause = getCause();
            if (ourCause != null)
                ourCause.printEnclosedStackTrace(s, trace, CAUSE_CAPTION, "", dejaVu);
        }
}    
```

Throwable有多种构造函数，可以设置detailMessge以及cause;  
detailMessge默认是空的。  
cause默认是this.

### 异常的链化
以一个异常对象为参数构造新的异常对象。新的异对象将包含先前异常的信息。这项技术主要是异常类的一个带Throwable参数的函数来实现的。这个当做参数的异常，我们叫他根源异常（cause）。   
就是通过构造函数传入cause。实际上就是常见了一个以新传进来的异常为基础的另外一个经过包装的异常。常常用于将一个异常捕获之后重新包装抛出一个新的异常

### getLocalizedMessage()
getLocalizedMessage()方法本质上还是返回的是detailMessage.因此在自定义的异常中常常需要重写这个方法。    
这样可以更加精细化的给出错误的描述信息。  
另外toString方法调用的也是getLocalizedMessage()方法  
toString()返回的就是className+getLocalizedMessage

### printStackTrace()
打印错误堆栈。首先打印this,然后打印错误堆栈，然后打印cause信息  
cause==this的时候不打印 
否则则证明cause是经过异常链化过来的，就需要打印cause信息

## Exception类
Exception类没有任何增强，只是对于Throwable进行了一层包装。方便他的子类进行一些操作