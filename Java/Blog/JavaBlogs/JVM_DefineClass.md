JVM_DefineClass 出现在类加载位置，这意味着当 ClassLoader 调用 defineClass 时，位置 URL 未包含在 ProtectionDomain 中的 CodeSource 中。这可能是因为类是动态生成的，但也可能是因为 ClassLoader 在定义类时根本没有提供信息。

`[Loaded sun.reflect.GeneratedMethodAccessor4035 from __JVM_DefineClass__]`

JVM 对待反射的两种方式：

- 使用 native 方法进行反射操作，这种方式第一次执行时会比较快，但是后面每次执行的速度都差不多
- 生成 bytecode 进行反射操作，所谓的 sun.reflect.GeneratedMethodAccessor，它是一个被反射调用方法的包装类，每次调用会有一个递增的序号。这种方式第一次调用速度较慢，但是多次调用后速度会提升 20 倍（jvm 代码文档说的，就不贴代码了，感兴趣的可以移步[这里](http://hg.openjdk.java.net/jdk6/jdk6/jdk/raw-file/ffa98eed5766/src/share/classes/sun/reflect/MethodAccessorGenerator.java)和[这里](http://hg.openjdk.java.net/jdk6/jdk6/jdk/file/tip/src/share/classes/sun/reflect/ReflectionFactory.java)的注释里观摩）

第二种方式的缺点就是会耗额外的内存，并且是在 permgen space 里。由于我真的不在乎那 20 倍的速度，所以决定把这个有一点点坑的特性关掉。在 ReflectionFactory 里有一种机制，就是当一个方法被反射调用的次数超过一定的阀值时（inflationThreshold），会使用第二种方式来提升速度。这个阀值的默认值是 15.那只要把这个值改大就好了，于是在启动参数里加上了

```
-Dsun.reflect.inflationThreshold=2147483647
```

再次观察的时候已经见不到类似这样的日志了

```
[Loaded sun.reflect.GeneratedMethodAccessor4035 from __JVM_DefineClass__]
```

整体：少用反射；

### 这行日志来源

这是 Sun 实现的 Java 标准库的一个细节。下面举例稍微讲解一下。
假如有这么一个类 A：

```java
public class A {
    public void foo(String name) {
        System.out.println("Hello, " + name);
    }
}
```

可以编写另外一个类来反射调用 A 上的方法：

```java
import java.lang.reflect.Method;

public class TestClassLoad {
    public static void main(String[] args) throws Exception {
        Class<?> clz = Class.forName("A");
        Object o = clz.newInstance();
        Method m = clz.getMethod("foo", String.class);
        for (int i = 0; i < 16; i++) {
            m.invoke(o, Integer.toString(i));
        }
    }
}
```


注意到 TestClassLoad 类上不会有对类 A 的符号依赖——也就是说在加载并初始化 TestClassLoad 类时不需要关心类 A 的存在与否，而是**等到 main() 方法执行到调用 Class.forName() 时才试图对类A做动态加载；**这里用的是一个参数版的 forName()，也就**是使用当前方法所在类的 ClassLoader 来加载，并且初始化新加载的类。**

回到主题。这次我的测试环境是 Sun 的 JDK 1.6.0 update 13 build 03。编译上述代码，并在执行 TestClassLoad 时加入 `-XX:+TraceClassLoading` 参数（或者 `-verbose:class` 或者直接 `-verbose` 都行），如下：

```java
java -XX:+TraceClassLoading TestClassLoad
```


可以看到输出了一大堆 log，把其中相关的部分截取出来如下：（完整的 log 可以从附件下载）

```java
[Loaded TestClassLoad from file:/D:/temp_code/test_java_classload/]
[Loaded A from file:/D:/temp_code/test_java_classload/]
[Loaded sun.reflect.NativeMethodAccessorImpl from shared objects file]
[Loaded sun.reflect.DelegatingMethodAccessorImpl from shared objects file]
Hello, 0
Hello, 1
Hello, 2
Hello, 3
Hello, 4
Hello, 5
Hello, 6
Hello, 7
Hello, 8
Hello, 9
Hello, 10
Hello, 11
Hello, 12
Hello, 13
Hello, 14
[Loaded sun.reflect.ClassFileConstants from shared objects file]
[Loaded sun.reflect.AccessorGenerator from shared objects file]
[Loaded sun.reflect.MethodAccessorGenerator from shared objects file]
[Loaded sun.reflect.ByteVectorFactory from shared objects file]
[Loaded sun.reflect.ByteVector from shared objects file]
[Loaded sun.reflect.ByteVectorImpl from shared objects file]
[Loaded sun.reflect.ClassFileAssembler from shared objects file]
[Loaded sun.reflect.UTF8 from shared objects file]
[Loaded java.lang.Void from shared objects file]
[Loaded sun.reflect.Label from shared objects file]
[Loaded sun.reflect.Label$PatchInfo from shared objects file]
[Loaded java.util.AbstractList$Itr from shared objects file]
[Loaded sun.reflect.MethodAccessorGenerator$1 from shared objects file]
[Loaded sun.reflect.ClassDefiner from shared objects file]
[Loaded sun.reflect.ClassDefiner$1 from shared objects file]
[Loaded sun.reflect.GeneratedMethodAccessor1 from __JVM_DefineClass__]
Hello, 15
```


可以看到前 15 次反射调用 `A.foo()` 方法并没有什么稀奇的地方，但在第 16 次反射调用时似乎有什么东西被触发了，导致 JVM 新加载了一堆类，其中就包括 `[Loaded sun.reflect.GeneratedMethodAccessor1 from __JVM_DefineClass__]`这么一行。这是哪里来的呢？

先来看看 JDK 里 `Method.invoke()` 是怎么实现的。
java.lang.reflect.Method：

```java
public final
    class Method extends AccessibleObject implements GenericDeclaration, 
                             Member {
    // ...

    private volatile MethodAccessor methodAccessor;
    // For sharing of MethodAccessors. This branching structure is
    // currently only two levels deep (i.e., one root Method and
    // potentially many Method objects pointing to it.)
    private Method              root;

    // ...

    public Object invoke(Object obj, Object... args)
            throws IllegalAccessException, IllegalArgumentException,
            InvocationTargetException
    {
        if (!override) {
            if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
                Class caller = Reflection.getCallerClass(1);
                Class targetClass = ((obj == null || !Modifier.isProtected(modifiers))
                                     ? clazz
                                     : obj.getClass());
                boolean cached;
                synchronized (this) {
                    cached = (securityCheckCache == caller)
                        && (securityCheckTargetClassCache == targetClass);
                }
                if (!cached) {
                    Reflection.ensureMemberAccess(caller, clazz, obj, modifiers);
                    synchronized (this) {
                    securityCheckCache = caller;
                    securityCheckTargetClassCache = targetClass;
                    }
                }
            }
        }
        if (methodAccessor == null) acquireMethodAccessor();
        return methodAccessor.invoke(obj, args);
    }

    // NOTE that there is no synchronization used here. It is correct
    // (though not efficient) to generate more than one MethodAccessor
    // for a given Method. However, avoiding synchronization will
    // probably make the implementation more scalable.
    private void acquireMethodAccessor() {
        // First check to see if one has been created yet, and take it
        // if so
        MethodAccessor tmp = null;
        if (root != null) tmp = root.getMethodAccessor();
        if (tmp != null) {
            methodAccessor = tmp;
            return;
        }
        // Otherwise fabricate one and propagate it up to the root
        tmp = reflectionFactory.newMethodAccessor(this);
        setMethodAccessor(tmp);
    }

    // ...
}
```

可以看到 `Method.invoke()` 实际上并不是自己实现的反射调用逻辑，而是委托给 `sun.reflect.MethodAccessor` 来处理。
每个实际的 Java 方法只有一个对应的 Method 对象作为 root，。这个 root 是不会暴露给用户的，而是每次在通过反射获取 `Method` 对象时新创建 `Method`对象把 root 包装起来再给用户。在第一次调用一个实际 Java 方法对应得 Method 对象的 `invoke()` 方法之前，实现调用逻辑的 MethodAccessor 对象还没创建；等第一次调用时才新创建 MethodAccessor 并更新给 root，然后调用`MethodAccessor.invoke()` 真正完成反射调用。

那么 MethodAccessor 是啥呢？
sun.reflect.MethodAccessor：

```java
public interface MethodAccessor {
    /** Matches specification in {@link java.lang.reflect.Method} */
    public Object invoke(Object obj, Object[] args)
        throws IllegalArgumentException, InvocationTargetException;
}
```


可以看到它只是一个单方法接口，其 `invoke()` 方法与 `Method.invoke()` 的对应。

创建 `MethodAccessor` 实例的是 `sun.reflect.ReflectionFactory`。

```java
public class ReflectionFactory {

    private static boolean initted = false;

    // ...

    //
    // "Inflation" mechanism. Loading bytecodes to implement
    // Method.invoke() and Constructor.newInstance() currently costs
    // 3-4x more than an invocation via native code for the first
    // invocation (though subsequent invocations have been benchmarked
    // to be over 20x faster). Unfortunately this cost increases
    // startup time for certain applications that use reflection
    // intensively (but only once per class) to bootstrap themselves.
    // To avoid this penalty we reuse the existing JVM entry points
    // for the first few invocations of Methods and Constructors and
    // then switch to the bytecode-based implementations.
    //
    // Package-private to be accessible to NativeMethodAccessorImpl
    // and NativeConstructorAccessorImpl
    private static boolean noInflation        = false;
    private static int     inflationThreshold = 15;

    // ...

    /** We have to defer full initialization of this class until after
        the static initializer is run since java.lang.reflect.Method's
        static initializer (more properly, that for
        java.lang.reflect.AccessibleObject) causes this class's to be
        run, before the system properties are set up. */
    private static void checkInitted() {
        if (initted) return;
        AccessController.doPrivileged(new PrivilegedAction() {
                public Object run() {
                    // Tests to ensure the system properties table is fully
                    // initialized. This is needed because reflection code is
                    // called very early in the initialization process (before
                    // command-line arguments have been parsed and therefore
                    // these user-settable properties installed.) We assume that
                    // if System.out is non-null then the System class has been
                    // fully initialized and that the bulk of the startup code
                    // has been run.

                    if (System.out == null) {
                        // java.lang.System not yet fully initialized
                        return null;
                    }

                    String val = System.getProperty("sun.reflect.noInflation");
                    if (val != null && val.equals("true")) {
                        noInflation = true;
                    }

                    val = System.getProperty("sun.reflect.inflationThreshold");
                    if (val != null) {
                        try {
                            inflationThreshold = Integer.parseInt(val);
                        } catch (NumberFormatException e) {
                            throw (RuntimeException) 
                                new RuntimeException("Unable to parse property sun.reflect.inflationThreshold").
                                    initCause(e);
                        }
                    }

                    initted = true;
                    return null;
                }
            });
    }

    // ...

    public MethodAccessor newMethodAccessor(Method method) {
        checkInitted();

        if (noInflation) {
            return new MethodAccessorGenerator().
                generateMethod(method.getDeclaringClass(),
                               method.getName(),
                               method.getParameterTypes(),
                               method.getReturnType(),
                               method.getExceptionTypes(),
                               method.getModifiers());
        } else {
            NativeMethodAccessorImpl acc =
                new NativeMethodAccessorImpl(method);
            DelegatingMethodAccessorImpl res =
                new DelegatingMethodAccessorImpl(acc);
            acc.setParent(res);
            return res;
        }
    }
}
```

如注释所述，实际的 MethodAccessor 实现有两个版本，一个是 Java 实现的，另一个是 native code 实现的。**Java 实现的版本在初始化时需要较多时间，但长久来说性能较好；native 版本正好相反，启动时相对较快，但运行时间长了之后速度就比不过 Java 版了**。这是 HotSpot 的优化方式带来的性能特性，同时也是许多虚拟机的共同点：跨越 native 边界会对优化有阻碍作用，它就像个黑箱一样让虚拟机难以分析也将其内联，于是运行时间长了之后反而是托管版本的代码更快些。

为了权衡两个版本的性能，Sun 的 JDK 使用了 `inflation` 的技巧：让 Java 方法在被反射调用时，开头若干次使用 native 版，等反射调用次数超过阈值时则生成一个专用的 `MethodAccessor` 实现类，生成其中的 `invoke()`方法的字节码，以后对该 Java 方法的反射调用就会使用 Java 版。

Sun 的 JDK 是从 1.4 系开始采用这种优化的，主要作者是 [Ken Russell](https://www.open-open.com/misc/goto?guid=4959676304937921919)

上面看到了 `ReflectionFactory.newMethodAccessor()`  生产 `MethodAccessor` 的逻辑，在「开头若干次」时用到的`DelegatingMethodAccessorImpl` 代码如下：
sun.reflect.DelegatingMethodAccessorImpl：

```java
/** Delegates its invocation to another MethodAccessorImpl and can
    change its delegate at run time. */

class DelegatingMethodAccessorImpl extends MethodAccessorImpl {
    private MethodAccessorImpl delegate;

    DelegatingMethodAccessorImpl(MethodAccessorImpl delegate) {
        setDelegate(delegate);
    }    

    public Object invoke(Object obj, Object[] args)
        throws IllegalArgumentException, InvocationTargetException
    {
        return delegate.invoke(obj, args);
    }

    void setDelegate(MethodAccessorImpl delegate) {
        this.delegate = delegate;
    }
}
```

这是一个间接层，方便在 native 与 Java 版的 MethodAccessor 之间实现切换。

然后下面就是 native 版 MethodAccessor 的 Java 一侧的声明：
sun.reflect.NativeMethodAccessorImpl：

```java
/** Used only for the first few invocations of a Method; afterward,
    switches to bytecode-based implementation */

class NativeMethodAccessorImpl extends MethodAccessorImpl {
    private Method method;
    private DelegatingMethodAccessorImpl parent;
    private int numInvocations;

    NativeMethodAccessorImpl(Method method) {
        this.method = method;
    }    

    public Object invoke(Object obj, Object[] args)
        throws IllegalArgumentException, InvocationTargetException
    {
        if (++numInvocations > ReflectionFactory.inflationThreshold()) {
            MethodAccessorImpl acc = (MethodAccessorImpl)
                new MethodAccessorGenerator().
                    generateMethod(method.getDeclaringClass(),
                                   method.getName(),
                                   method.getParameterTypes(),
                                   method.getReturnType(),
                                   method.getExceptionTypes(),
                                   method.getModifiers());
            parent.setDelegate(acc);
        }

        return invoke0(method, obj, args);
    }

    void setParent(DelegatingMethodAccessorImpl parent) {
        this.parent = parent;
    }

    private static native Object invoke0(Method m, Object obj, Object[] args);
}
```

每次 `NativeMethodAccessorImpl.invoke()` 方法被调用时，都会增加一个调用次数计数器，看超过阈值没有；一旦超过，则调用 `MethodAccessorGenerator.generateMethod()` 来生成 Java 版的 MethodAccessor 的实现类，并且改变 DelegatingMethodAccessorImpl 所引用的 MethodAccessor 为 Java 版。后续经由 `DelegatingMethodAccessorImpl.invoke()` 调用到的就是 Java 版的实现了。

注意到关键的 `invoke0()` 方法是个 native 方法。它在 HotSpot VM 里是由 `JVM_InvokeMethod()` 函数所支持的：

```java
JNIEXPORT jobject JNICALL Java_sun_reflect_NativeMethodAccessorImpl_invoke0
(JNIEnv *env, jclass unused, jobject m, jobject obj, jobjectArray args)
{
    return JVM_InvokeMethod(env, m, obj, args);
}
```

 

```java
JVM_ENTRY(jobject, JVM_InvokeMethod(JNIEnv *env, jobject method, jobject obj, jobjectArray args0))
  JVMWrapper("JVM_InvokeMethod");
  Handle method_handle;
  if (thread->stack_available((address) &method_handle) >= JVMInvokeMethodSlack) {
    method_handle = Handle(THREAD, JNIHandles::resolve(method));
    Handle receiver(THREAD, JNIHandles::resolve(obj));
    objArrayHandle args(THREAD, objArrayOop(JNIHandles::resolve(args0)));
    oop result = Reflection::invoke_method(method_handle(), receiver, args, CHECK_NULL);
    jobject res = JNIHandles::make_local(env, result);
    if (JvmtiExport::should_post_vm_object_alloc()) {
      oop ret_type = java_lang_reflect_Method::return_type(method_handle());
      assert(ret_type != NULL, "sanity check: ret_type oop must not be NULL!");
      if (java_lang_Class::is_primitive(ret_type)) {
        // Only for primitive type vm allocates memory for java object.
        // See box() method.
        JvmtiExport::post_vm_object_alloc(JavaThread::current(), result);
      }
    }
    return res;
  } else {
    THROW_0(vmSymbols::java_lang_StackOverflowError());
  }
JVM_END
```


其中的关键又是 Reflection::invoke_method()：

```java
// This would be nicer if, say, java.lang.reflect.Method was a subclass
// of java.lang.reflect.Constructor

oop Reflection::invoke_method(oop method_mirror, Handle receiver, objArrayHandle args, TRAPS) {
  oop mirror             = java_lang_reflect_Method::clazz(method_mirror);
  int slot               = java_lang_reflect_Method::slot(method_mirror);
  bool override          = java_lang_reflect_Method::override(method_mirror) != 0;
  objArrayHandle ptypes(THREAD, objArrayOop(java_lang_reflect_Method::parameter_types(method_mirror)));

  oop return_type_mirror = java_lang_reflect_Method::return_type(method_mirror);
  BasicType rtype;
  if (java_lang_Class::is_primitive(return_type_mirror)) {
    rtype = basic_type_mirror_to_basic_type(return_type_mirror, CHECK_NULL);
  } else {
    rtype = T_OBJECT;
  }

  instanceKlassHandle klass(THREAD, java_lang_Class::as_klassOop(mirror));
  methodOop m = klass->method_with_idnum(slot);
  if (m == NULL) {
    THROW_MSG_0(vmSymbols::java_lang_InternalError(), "invoke");
  }
  methodHandle method(THREAD, m);

  return invoke(klass, method, receiver, override, ptypes, rtype, args, true, THREAD);
}
```


再下去就深入到 HotSpot VM 的内部了，本文就在这里打住吧。有同学有兴趣深究的话以后可以再写一篇讨论 native 版的实现。

回到 Java 的一侧。MethodAccessorGenerator 长啥样呢？由于代码太长，这里就不完整贴了，有兴趣的可以到 OpenJDK 6 的 Mercurial 仓库看： [OpenJDK 6 build 17的MethodAccessorGenerator](https://www.open-open.com/misc/goto?guid=4959676305031596494) 。它的基本工作就是在内存里生成新的专用 Java 类，并将其加载。就贴这么一个方法：

```java
private static synchronized String generateName(boolean isConstructor,
                                                boolean forSerialization)
{
    if (isConstructor) {
        if (forSerialization) {
            int num = ++serializationConstructorSymnum;
            return "sun/reflect/GeneratedSerializationConstructorAccessor" + num;
        } else {
            int num = ++constructorSymnum;
            return "sun/reflect/GeneratedConstructorAccessor" + num;
        }
    } else {
        int num = ++methodSymnum;
        return "sun/reflect/GeneratedMethodAccessor" + num;
    }
}
```


去阅读源码的话，可以看到 MethodAccessorGenerator 是如何一点点把 Java 版的 MethodAccessor 实现类生产出来的。也可以看到 GeneratedMethodAccessor+数字这种名字是从哪里来的了，就在上面的 generateName()方法里。
对本文开头的例子的 A.foo()，生成的 Java 版 MethodAccessor 大致如下：

```java
package sun.reflect;

public class GeneratedMethodAccessor1 extends MethodAccessorImpl {    
    public GeneratedMethodAccessor1() {
        super();
    }

    public Object invoke(Object obj, Object[] args)   
        throws IllegalArgumentException, InvocationTargetException {
        // prepare the target and parameters
        if (obj == null) throw new NullPointerException();
        try {
            A target = (A) obj;
            if (args.length != 1) throw new IllegalArgumentException();
            String arg0 = (String) args[0];
        } catch (ClassCastException e) {
            throw new IllegalArgumentException(e.toString());
        } catch (NullPointerException e) {
            throw new IllegalArgumentException(e.toString());
        }
        // make the invocation
        try {
            target.foo(arg0);
        } catch (Throwable t) {
            throw new InvocationTargetException(t);
        }
    }
}
```

就反射调用而言，这个 invoke()方法非常干净（然而就“正常调用”而言这额外开销还是明显的）。注意到参数数组被拆开了，把每个参数都恢复到原本没有被 Object[]包装前的样子，然后对目标方法做正常的 invokevirtual 调用。由于在生成代码时已经循环遍历过参数类型的数组，生成出来的代码里就不再包含循环了。

当该反射调用成为热点时，它甚至可以被内联到靠近 Method.invoke()的一侧，大大降低了反射调用的开销。而 native 版的反射调用则无法被有效内联，因而调用开销无法随程序的运行而降低。

虽说 Sun 的 JDK 这种实现方式使得反射调用方法成本比以前降低了很多，但 Method.invoke()本身要用数组包装参数；而且每次调用都必须检查方法的可见性（在 Method.invoke()里），也必须检查每个实际参数与形式参数的类型匹配性（在 NativeMethodAccessorImpl.invoke0()里或者生成的 Java 版 MethodAccessor.invoke()里）；而且 Method.invoke()就像是个独木桥一样，各处的反射调用都要挤过去，在调用点上收集到的类型信息就会很乱，影响内联程序的判断，使得 Method.invoke()自身难以被内联到调用方。

相比之下 [JDK 7里新的MethodHandle](https://www.open-open.com/misc/goto?guid=4959676305110459284) 则更有潜力，在其功能完全实现后能达到比普通反射调用方法更高的性能。在使用 MethodHandle 来做反射调用时，MethodHandle.invoke()的形式参数与返回值类型都是准确的，所以只需要在链接方法的时候才需要检查类型的匹配性，而不必在每次调用时都检查。而且 MethodHandle 是不可变值，在创建后其内部状态就不会再改变了；JVM 可以利用这个知识而放心的对它做激进优化，例如将实际的调用目标内联到做反射调用的一侧。

到本来 Java 的安全机制使得不同类之间不是任意信息都可见，但 Sun 的 JDK 里开了个口，有一个标记类专门用于开后门：

```java
package sun.reflect;

/** <P> MagicAccessorImpl (named for parity with FieldAccessorImpl and
    others, not because it actually implements an interface) is a
    marker class in the hierarchy. All subclasses of this class are
    "magically" granted access by the VM to otherwise inaccessible
    fields and methods of other classes. It is used to hold the code
    for dynamicallfasy-generated FieldAccessorImpl and MethodAccessorImpl
    subclasses. (Use of the word "unsafe" was avoided in this class's
    name to avoid confusion with {@link sun.misc.Unsafe}.) </P>

    <P> The bug fix for 4486457 also necessitated disabling
    verification for this class and all subclasses, as opposed to just
    SerializationConstructorAccessorImpl and subclasses, to avoid
    having to indicate to the VM which of these dynamically-generated
    stub classes were known to be able to pass the verifier. </P>

    <P> Do not change the name of this class without also changing the
    VM's code. </P> */

class MagicAccessorImpl {
}
```

那个"__JVM_DefineClass__"的来源是这里：
src/share/vm/prims/jvm.cpp

```java
// common code for JVM_DefineClass() and JVM_DefineClassWithSource()
// and JVM_DefineClassWithSourceCond()
static jclass jvm_define_class_common(JNIEnv *env, const char *name,
                                      jobject loader, const jbyte *buf,
                                      jsize len, jobject pd, const char *source,
                                      jboolean verify, TRAPS) {
  if (source == NULL)  source = "__JVM_DefineClass__";
```