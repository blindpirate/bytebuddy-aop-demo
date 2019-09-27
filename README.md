### Byte Buddy AOP demo

This is a sample project demonstrating the power of [byte-buddy](https://github.com/raphw/byte-buddy).

The main code is quite short:

```java
public class Service {
    @LoggerAdvisor.Log
    public int foo(int value) {
        System.out.println("foo: " + value);
        return value;
    }

    public int bar(int value) {
        System.out.println("bar: " + value);
        return value;
    }

    public static void main(String[] args) throws Exception {
        Service service = new ByteBuddy()
                .subclass(Service.class)
                .method(ElementMatchers.any())
                .intercept(Advice.to(LoggerAdvisor.class))
                .make()
                .load(Service.class.getClassLoader())
                .getLoaded()
                .newInstance();
        service.bar(123);
        service.foo(456);
    }
}

class LoggerAdvisor {
    @Advice.OnMethodEnter
    public static void onMethodEnter(@Advice.Origin Method method, @Advice.AllArguments Object[] arguments) {
        if (method.getAnnotation(Log.class) != null) {
            System.out.println("Enter " + method.getName() + " with arguments: " + Arrays.toString(arguments));
        }
    }

    @Advice.OnMethodExit
    public static void onMethodExit(@Advice.Origin Method method, @Advice.AllArguments Object[] arguments, @Advice.Return Object ret) {
        if (method.getAnnotation(Log.class) != null) {
            System.out.println("Exit " + method.getName() + " with arguments: " + Arrays.toString(arguments) + " return: " + ret);
        }
    }

    @Retention(RetentionPolicy.RUNTIME)
    public @interface Log {
    }
}
```