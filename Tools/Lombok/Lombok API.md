# 一、val

### Finally! Hassle-free final local variables.

`val` was introduced in lombok 0.10.

NEW in Lombok 1.18.22: `val` gets replaced with `final var`.

### Overview

You can use `val` as the type of a local variable declaration instead of actually writing the type. When you do this, the type will be inferred from the initializer expression. The local variable will also be made final. This feature works on local variables and on foreach loops only, not on fields. The initializer expression is required.

您可以使用'val'作为局部变量声明的类型，而不是实际编写该类型。执行此操作时，将从初始值设定项表达式推断类型。局部变量也将成为最终变量。此功能仅适用于局部变量和foreach循环，而不适用于字段。初始值设定项表达式是必需的。

`val` is actually a 'type' of sorts, and exists as a real class in the `lombok` package. You must import it for val to work (or use `lombok.val` as the type). The existence of this type on a local variable declaration triggers both the adding of the `final` keyword as well as copying the type of the initializing expression which overwrites the 'fake' `val` type.

`val`实际上是一种排序类型，在'lombok'包中作为一个实类存在。必须导入它才能使val工作（或使用'lombok.val'作为类型）。局部变量声明中存在此类型会触发添加'final'关键字以及复制初始化表达式的类型，从而覆盖'fake''val'类型。

WARNING: This feature does not currently work in NetBeans.

### With Lombok

```java
import java.util.ArrayList;
import java.util.HashMap;
import lombok.val;

public class ValExample {
    public String example() {
        val example = new ArrayList<String>();
        example.add("Hello, World!");
        val foo = example.get(0);
        return foo.toLowerCase();
    }

    public void example2() {
        val map = new HashMap<Integer, String>();
        map.put(0, "zero");
        map.put(5, "five");
        for (val entry : map.entrySet()) {
            System.out.printf("%d: %s\n", entry.getKey(), entry.getValue());
        }
    }
}
```

### Vanilla Java

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;

public class ValExample {
    public String example() {
        final ArrayList<String> example = new ArrayList<String>();
        example.add("Hello, World!");
        final String foo = example.get(0);
        return foo.toLowerCase();
    }

    public void example2() {
        final HashMap<Integer, String> map = new HashMap<Integer, String>();
        map.put(0, "zero");
        map.put(5, "five");
        for (final Map.Entry<Integer, String> entry : map.entrySet()) {
            System.out.printf("%d: %s\n", entry.getKey(), entry.getValue());
        }
    }
}
```

### Supported configuration keys:

- `lombok.val.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `val` as a warning or error if configured.

### Small print

For compound types, the most common superclass is inferred, not any shared interfaces. For example, `bool ? new HashSet() : new ArrayList()` is an expression with a compound type: The result is both `AbstractCollection` as well as `Serializable`. The type inferred will be `AbstractCollection`, as that is a class, whereas `Serializable` is an interface.

In ambiguous cases, such as when the initializer expression is `null`, `java.lang.Object` is inferred.



# 二、var

### Mutably! Hassle-free local variables.

- `var` was promoted to the main package in lombok 1.16.20; given that [JEP 286](http://openjdk.java.net/jeps/286) establishes expectations, and lombok's take on `var` follows these, we've decided to promote `var` eventhough the feature remains controversial.
- `var` was introduced in lombok 1.16.12 as experimental feature.

### Overview

`var` works exactly like [`val`](https://projectlombok.org/features/val), except the local variable is *not* marked as `final`.

The type is still entirely derived from the mandatory initializer expression, and any further assignments, while now legal (because the variable is no longer `final`), aren't looked at to determine the appropriate type.
For example, `var x = "Hello"; x = Color.RED;` does *not* work; the type of x will be inferred to be `java.lang.String` and thus, the `x = Color.RED` assignment will fail. If the type of `x` was inferred to be `java.lang.Object` this code would have compiled, but that's not how`var` works.

### Supported configuration keys:

- `lombok.var.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `var` as a warning or error if configured.





# 三、@NonNull

### or: How I learned to stop worrying and love the NullPointerException.

或者：我是如何学会停止担忧，爱上NullPointerException的。

`@NonNull` was introduced in lombok v0.11.10

`@NonNull`是在 lombok v0.11.10 中引入的。

### Overview

You can use `@NonNull` on a record component, or a parameter of a method or constructor. This will cause to lombok generate a null-check statement for you.

Lombok has always treated various annotations generally named `@NonNull` on a field as a signal to generate a null-check if lombok generates an entire method or constructor for you, via for example [`@Data`](https://projectlombok.org/features/Data). However, using lombok's own `@lombok.NonNull` on a parameter or record component results in the insertion of the null-check at the top of that method.

The null-check looks like `if (param == null) throw new NullPointerException("param is marked @NonNull but is null");` and will be inserted at the very top of your method. For constructors, the null-check will be inserted immediately following any explicit `this()` or `super()` calls. For record components, the null-check will be inserted in the 'compact constructor' (the one that has no argument list at all), which will be generated if you have no constructor. If you have written out the record constructor in long form (with parameters matching your components exactly), then nothing happens - you'd have to annotate the parameters of this long-form constructor instead.

If a null-check is already present at the top, no additional null-check will be generated.

您可以对记录组件或方法或构造函数的参数使用“@NonNull”。这将使得 lombok 为您生成空检查语句。

Lombok 始终将字段上通常命名为“@NonNull”的各种注释视为信号，以便在 Lombok为您生成整个方法或构造函数时生成 null 检查，例如通过[`@Data`](https://projectlombok.org/features/Data).但是，使用 lombok 自己的`@lombok`。参数或记录组件上的 `NonNull`导致在该方法的顶部插入null检查。

空检查类似于“如果（param==null）抛出新的NullPointerException（“param标记为@NonNull，但为null”）；”**并将插入到方法的最顶部**。对于构造函数，空检查将在任何显式 `this()` 或 `super()` 调用之后立即插入。对于记录组件，空检查将插入到“紧凑构造函数”（完全没有参数列表的构造函数）中，如果没有构造函数，将生成该构造函数。如果您以长格式编写了记录构造函数（参数与您的组件完全匹配），那么什么也不会发生——您将不得不注释这个长格式构造函数的参数。

如果顶部已存在空检查，则不会生成额外的空检查。

### With Lombok

```java
import lombok.NonNull;

public class NonNullExample extends Something {
  private String name;
  
  public NonNullExample(@NonNull Person person) {
    super("Hello");
    this.name = person.getName();
  }
}
```

### Vanilla Java

```java
import lombok.NonNull;

public class NonNullExample extends Something {
  private String name;
  
  public NonNullExample(@NonNull Person person) {
    super("Hello");
    if (person == null) {
      throw new NullPointerException("person is marked @NonNull but is null");
    }
    this.name = person.getName();
  }
}
```

### Supported configuration keys:

- `lombok.nonNull.exceptionType` = [`NullPointerException` | `IllegalArgumentException` | `JDK` | `Guava` | `Assertion`] (default: `NullPointerException`).

    When lombok generates a null-check `if` statement, by default, a `java.lang.NullPointerException` will be thrown with '*field name* is marked non-null but is null' as the exception message. However, you can use `IllegalArgumentException` in this configuration key to have lombok throw that exception with this message instead. By using `Assertion`, an `assert` statement with the same message will be generated. The keys `JDK` or `Guava` result in an invocation to the standard nullcheck method of these two frameworks: `java.util.Objects.requireNonNull([field name here], "[field name here] is marked non-null but is null");` or `com.google.common.base.Preconditions.checkNotNull([field name here], "[field name here] is marked non-null but is null");` respectively.

- `lombok.nonNull.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@NonNull` as a warning or error if configured.
    
- `lombok.nonNull.exceptionType` = [`NullPointerException` | `IllegalArgumentException` | `JDK` | `Guava` | `Assertion`] ，默认为 `NullPointerException`

    当 lombok 生成一个 if 声明的空校验时，默认情况下，将抛出一个异常信息为「XXX属性名称 is marked non-null but is null」的 `java.lang.NullPointerException`，您可以在此配置键中使用非法ArgumentException，让lombok将该异常与此消息一起抛出。通过使用“断言”，将生成具有相同消息的“断言”语句。键'JDK'或'Guava'导致调用这两个框架的标准nullcheck方法：`java。util。物体！requirennull（[此处的字段名]，[此处的字段名]标记为非空，但为空）；或“com”。谷歌。普通的基地。先决条件！checkNotZero（[field name here]，[field name here]标记为非null，但为null；

- `lombok.nonNull.flagUsage` = [`warning` | `error`] ，默认没有配置

    Lombok 会将任何使用`@NonNull`的情况标记为警告或错误（如果已配置）

### Small print

Lombok's detection scheme for already existing null-checks consists of scanning for if statements or assert statements that look just like lombok's own. Any 'throws' statement as the 'then' part of the if statement, whether in braces or not, counts. Any invocation to any method named `requireNonNull` or `checkNotNull` counts. The conditional of the if statement *must* look exactly like `PARAMNAME == null`; the assert statement *must* look exactly like `PARAMNAME != null`. The invocation to a `requireNonNull`-style method must be on its own (a statement which just invokes that method), or must be the expression of an assignment or variable declaration statement. The first statement in your method that is not such a null-check stops the process of inspecting for null-checks.

While `@Data` and other method-generating lombok annotations will trigger on various well-known annotations that signify the field must never be `@NonNull`, this feature only triggers on lombok's own `@NonNull` annotation from the `lombok` package.

A `@NonNull` on a primitive parameter results in a warning. No null-check will be generated.

A `@NonNull` on a parameter of an abstract method used to generate a warning; starting with version 1.16.8, this is no longer the case, to acknowledge the notion that `@NonNull` also has a documentary role. For the same reason, you can annotate a method as `@NonNull`; this is allowed, generates no warning, and does not generate any code.



# 四、@Cleanup

### Automatic resource management: Call your `close()` methods safely with no hassle.

### Overview

You can use `@Cleanup` to ensure a given resource is automatically cleaned up before the code execution path exits your current scope. You do this by annotating any local variable declaration with the `@Cleanup` annotation like so:
`@Cleanup InputStream in = new FileInputStream("some/file");`
As a result, at the end of the scope you're in, `in.close()` is called. This call is guaranteed to run by way of a try/finally construct. Look at the example below to see how this works.

If the type of object you'd like to cleanup does not have a `close()` method, but some other no-argument method, you can specify the name of this method like so:
`@Cleanup("dispose") org.eclipse.swt.widgets.CoolBar bar = new CoolBar(parent, 0);`
By default, the cleanup method is presumed to be `close()`. A cleanup method that takes 1 or more arguments cannot be called via `@Cleanup`.

### With Lombok

```

import lombok.Cleanup;
import java.io.*;

public class CleanupExample {
  public static void main(String[] args) throws IOException {
    @Cleanup InputStream in = new FileInputStream(args[0]);
    @Cleanup OutputStream out = new FileOutputStream(args[1]);
    byte[] b = new byte[10000];
    while (true) {
      int r = in.read(b);
      if (r == -1) break;
      out.write(b, 0, r);
    }
  }
}
```

### Vanilla Java

```

import java.io.*;

public class CleanupExample {
  public static void main(String[] args) throws IOException {
    InputStream in = new FileInputStream(args[0]);
    try {
      OutputStream out = new FileOutputStream(args[1]);
      try {
        byte[] b = new byte[10000];
        while (true) {
          int r = in.read(b);
          if (r == -1) break;
          out.write(b, 0, r);
        }
      } finally {
        if (out != null) {
          out.close();
        }
      }
    } finally {
      if (in != null) {
        in.close();
      }
    }
  }
}
```

### Supported configuration keys:

- `lombok.cleanup.flagUsage` = [`warning` | `error`] (default: not set)

### Small print

In the finally block, the cleanup method is only called if the given resource is not `null`. However, if you use `delombok` on the code, a call to `lombok.Lombok.preventNullAnalysis(Object o)` is inserted to prevent warnings if static code analysis could determine that a null-check would not be needed. Compilation with `lombok.jar` on the classpath removes that method call, so there is no runtime dependency.

If your code throws an exception, and the cleanup method call that is then triggered also throws an exception, then the original exception is hidden by the exception thrown by the cleanup call. You should *not* rely on this 'feature'. Preferably, lombok would like to generate code so that, if the main body has thrown an exception, any exception thrown by the close call is silently swallowed (but if the main body exited in any other way, exceptions by the close call will not be swallowed). The authors of lombok do not currently know of a feasible way to implement this scheme, but if java updates allow it, or we find a way, we'll fix it.

You do still need to handle any exception that the cleanup method can generate!



# 五、@Getter and @Setter

Never write `public int getFoo() {return foo;}` again.

不在需要写 `public int getFoo() {return foo;}`;

### Overview

You can annotate any field with `@Getter` and/or `@Setter`, to let lombok generate the default getter/setter automatically.
A default getter simply returns the field, and is named `getFoo` if the field is called `foo` (or `isFoo` if the field's type is `boolean`). A default setter is named `setFoo` if the field is called `foo`, returns `void`, and takes 1 parameter of the same type as the field. It simply sets the field to this value.

The generated getter/setter method will be `public` unless you explicitly specify an `AccessLevel`, as shown in the example below. Legal access levels are `PUBLIC`, `PROTECTED`, `PACKAGE`, and `PRIVATE`.

You can also put a `@Getter` and/or `@Setter` annotation on a class. In that case, it's as if you annotate all the non-static fields in that class with the annotation.

You can always manually disable getter/setter generation for any field by using the special `AccessLevel.NONE` access level. This lets you override the behaviour of a `@Getter`, `@Setter` or `@Data` annotation on a class.

To put annotations on the generated method, you can use `onMethod=@__({@AnnotationsHere})`; to put annotations on the only parameter of a generated setter method, you can use `onParam=@__({@AnnotationsHere})`. Be careful though! This is an experimental feature. For more details see the documentation on the [onX](https://projectlombok.org/features/experimental/onX) feature.

*NEW in lombok v1.12.0:* javadoc on the field will now be copied to generated getters and setters. Normally, all text is copied, and `@return` is *moved* to the getter, whilst `@param` lines are *moved* to the setter. Moved means: Deleted from the field's javadoc. It is also possible to define unique text for each getter/setter. To do that, you create a 'section' named `GETTER` and/or `SETTER`. A section is a line in your javadoc containing 2 or more dashes, then the text 'GETTER' or 'SETTER', followed by 2 or more dashes, and nothing else on the line. If you use sections, `@return` and `@param` stripping for that section is no longer done (move the `@return` or `@param` line into the section).

### With Lombok

```java
import lombok.AccessLevel;
import lombok.Getter;
import lombok.Setter;

public class GetterSetterExample {
    /**
   * Age of the person. Water is wet.
   * 
   * @param age New value for this person's age. Sky is blue.
   * @return The current value of this person's age. Circles are round.
   */
    @Getter @Setter 
    private int age = 10;

    /**
   * Name of the person.
   * -- SETTER --
   * Changes the name of this person.
   * 
   * @param name The new value.
   */
    @Setter(AccessLevel.PROTECTED) 
    private String name;

    @Override 
    public String toString() {
        return String.format("%s (age: %d)", name, age);
    }
}
```

### Vanilla Java

```java
public class GetterSetterExample {
    /**
   * Age of the person. Water is wet.
   */
    private int age = 10;

    /**
   * Name of the person.
   */
    private String name;

    @Override 
    public String toString() {
        return String.format("%s (age: %d)", name, age);
    }

    /**
   * Age of the person. Water is wet.
   *
   * @return The current value of this person's age. Circles are round.
   */
    public int getAge() {
        return age;
    }

    /**
   * Age of the person. Water is wet.
   *
   * @param age New value for this person's age. Sky is blue.
   */
    public void setAge(int age) {
        this.age = age;
    }

    /**
   * Changes the name of this person.
   *
   * @param name The new value.
   */
    protected void setName(String name) {
        this.name = name;
    }
}
```

### Supported configuration keys:

- `lombok.accessors.chain` = [`true` | `false`] (default: false)

    If set to `true`, generated setters will return `this` (instead of `void`). An explicitly configured `chain` parameter of an [`@Accessors`](https://projectlombok.org/features/experimental/Accessors) annotation takes precedence over this setting.

- `lombok.accessors.fluent` = [`true` | `false`] (default: false)

    If set to `true`, generated getters and setters will not be prefixed with the bean-standard '`get`, `is` or `set`; instead, the methods will use the same name as the field (minus prefixes). An explicitly configured `chain` parameter of an [`@Accessors`](https://projectlombok.org/features/experimental/Accessors) annotation takes precedence over this setting.

- `lombok.accessors.prefix` += *a field prefix* (default: empty list)

    This is a list property; entries can be added with the `+=` operator. Inherited prefixes from parent config files can be removed with the `-=` operator. Lombok will strip any matching field prefix from the name of a field in order to determine the name of the getter/setter to generate. For example, if `m` is one of the prefixes listed in this setting, then a field named `mFoobar` will result in a getter named `getFoobar()`, not `getMFoobar()`. An explicitly configured `prefix` parameter of an [`@Accessors`](https://projectlombok.org/features/experimental/Accessors) annotation takes precedence over this setting.

- `lombok.getter.noIsPrefix` = [`true` | `false`] (default: false)

    If set to `true`, getters generated for `boolean` fields will use the `get` prefix instead of the default`is` prefix, and any generated code that calls getters, such as `@ToString`, will also use `get` instead of `is`

- `lombok.setter.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@Setter` as a warning or error if configured.

- `lombok.getter.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@Getter` as a warning or error if configured.

- `lombok.copyableAnnotations` = [*A list of fully qualified types*] (default: empty list)

    Lombok will copy any of these annotations from the field to the setter parameter, and to the getter method. Note that lombok ships with a bunch of annotations 'out of the box' which are known to be copyable: All popular nullable/nonnull annotations.

### Small print

For generating the method names, the first character of the field, if it is a lowercase character, is title-cased, otherwise, it is left unmodified. Then, get/set/is is prefixed.

No method is generated if any method already exists with the same name (case insensitive) and same parameter count. For example, `getFoo()` will not be generated if there's already a method `getFoo(String... x)` even though it is technically possible to make the method. This caveat exists to prevent confusion. If the generation of a method is skipped for this reason, a warning is emitted instead. Varargs count as 0 to N parameters. You can mark any method with `@lombok.experimental.Tolerate` to hide them from lombok.

For `boolean` fields that start with `is` immediately followed by a title-case letter, nothing is prefixed to generate the getter name.

Any variation on `boolean` will *not* result in using the `is` prefix instead of the `get` prefix; for example, returning `java.lang.Boolean` results in a `get` prefix, not an `is` prefix.

A number of annotations from popular libraries that indicate non-nullness, such as `javax.annotation.Nonnull`, if present on the field, result in an explicit null check in the generated setter.

Various well-known annotations about nullability, such as `org.eclipse.jdt.annotation.NonNull`, are automatically copied over to the right place (method for getters, parameter for setters). You can specify additional annotations that should always be copied via lombok [configuration key](https://projectlombok.org/features/configuration) `lombok.copyableAnnotations`.

You can annotate a class with a `@Getter` or `@Setter` annotation. Doing so is equivalent to annotating all non-static fields in that class with that annotation. `@Getter`/`@Setter` annotations on fields take precedence over the ones on classes.

Using the `AccessLevel.NONE` access level simply generates nothing. It's useful only in combination with [`@Data`](https://projectlombok.org/features/Data) or a class-wide `@Getter` or `@Setter`.

`@Getter` can also be used on enums. `@Setter` can't, not for a technical reason, but for a pragmatic one: Setters on enums are an extremely bad idea.



# 六、@ToString

No need to start a debugger to see your fields: Just let lombok generate a `toString` for you!

无需启动调试器就可以查看字段，只需要让 lombok 为您生成一个 `toString`。

### Overview

Any class definition may be annotated with `@ToString` to let lombok generate an implementation of the `toString()` method. By default, it'll print your class name, along with each field, in order, separated by commas.

任何类的定义都可以使用 `@ToString` 注释，使得 lombok 生成 `toString()` 方法的实现，**默认情况下，它将按照顺序打印您的类名和各个字段，相互之间使用逗号分隔**。

By setting the `includeFieldNames` parameter to true you can add some clarity (but also quite some length) to the output of the `toString()` method.

通过将 `includeFieldNames` 参数设置为 true，可以为 `toString()` 方法的输出增加一些清晰度（但是也有相当长的长度）。

By default, all non-static fields will be printed. If you want to skip some fields, you can annotate these fields with `@ToString.Exclude`. Alternatively, you can specify exactly which fields you wish to be used by using `@ToString(onlyExplicitlyIncluded = true)`, then marking each field you want to include with `@ToString.Include`.

**默认情况下，将打印所有非静态字段。如果要跳过某些字段，可以使用 `@ToString.Exclude` 注释这些字段。或者，您可以使用 `@ToString(onlyExplicitlyIncluded=true)`，然后用 `@ToString.Include` 标记要包含的每个字段，从而精确指定要使用的字段。**

By setting `callSuper` to *true*, you can include the output of the superclass implementation of `toString` to the output. Be aware that the default implementation of `toString()` in `java.lang.Object` is pretty much meaningless, so you probably don't want to do this unless you are extending another class.

**通过将 `callSuper` 设置为 true，可以将 `toString` 的超类实现的输出包含到输出中**。请注意，`java.lang.Object`中的`toString()`的默认实现几乎没有意义，因此除非扩展另一个类，否则您可能不想这样做。

You can also include the output of a method call in your `toString`. Only instance (non-static) methods that take no arguments can be included. To do so, mark the method with `@ToString.Include`.

您还**可以在 `toString` 中包含方法调用的输出。只能包含不带参数的实例（非静态）方法。**为此，请使用 `@ToString.Include` 标记该方法。

You can change the name used to identify the member with `@ToString.Include(name = "some other name")`, and you can change the order in which the members are printed via `@ToString.Include(rank = -1)`. Members without a rank are considered to have rank 0, members of a higher rank are printed first, and members of the same rank are printed in the same order they appear in the source file.

您可以使用 `@ToString.Include（name=“some other name”）`更改用于标识成员的名称，也可以通过 `@ToString.Include(rank=-1)`更改成员的打印顺序。没有级别的成员被视为级别为 0，级别较高的成员将首先打印，相同级别的成员将按照它们在源文件中出现的相同顺序打印。

### With Lombok

```java
import lombok.ToString;

@ToString
public class ToStringExample {
    private static final int STATIC_VAR = 10;
    private String name;
    private Shape shape = new Square(5, 10);
    private String[] tags;
    @ToString.Exclude 
        private int id;

    public String getName() {
        return this.name;
    }

    @ToString(callSuper=true, includeFieldNames=true)
    public static class Square extends Shape {
        private final int width, height;

        public Square(int width, int height) {
            this.width = width;
            this.height = height;
        }
    }
}

```

### Vanilla Java

```java
import java.util.Arrays;

public class ToStringExample {
  private static final int STATIC_VAR = 10;
  private String name;
  private Shape shape = new Square(5, 10);
  private String[] tags;
  private int id;
  
  public String getName() {
    return this.name;
  }
  
  public static class Square extends Shape {
    private final int width, height;
    
    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }
    
    @Override 
    public String toString() {
      return "Square(super=" + super.toString() + ", width=" + this.width + ", height=" + this.height + ")";
    }
  }
  
  @Override 
  public String toString() {
    return "ToStringExample(" + this.getName() + ", " + this.shape + ", " + Arrays.deepToString(this.tags) + ")";
  }
}
```

### Supported configuration keys:

- `lombok.toString.includeFieldNames` = [`true` | `false`] (default: true)

    Normally lombok generates a fragment of the toString response for each field in the form of `fieldName = fieldValue`. If this setting is set to `false`, lombok will omit the name of the field and simply deploy a comma-separated list of all the field values. The annotation parameter '`includeFieldNames`', if explicitly specified, takes precedence over this setting.

- `lombok.toString.doNotUseGetters` = [`true` | `false`] (default: false)

    If set to `true`, lombok will access fields directly instead of using getters (if available) when generating `toString` methods. The annotation parameter '`doNotUseGetters`', if explicitly specified, takes precedence over this setting.

- `lombok.toString.callSuper` = [`call` | `skip` | `warn`] (default: skip)

    If set to `call`, lombok will generate calls to the superclass implementation of `toString` if your class extends something. If set to `skip` no such call is generated. If set to `warn` no such call is generated either, but lombok does generate a warning to tell you about it.

- `lombok.toString.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@ToString` as a warning or error if configured.

### Small print

If there is *any* method named `toString` with no arguments, regardless of return type, no method will be generated, and instead a warning is emitted explaining that your `@ToString` annotation is doing nothing. You can mark any method with `@lombok.experimental.Tolerate` to hide them from lombok.

Arrays are printed via `Arrays.deepToString`, which means that arrays that contain themselves will result in `StackOverflowError`s. However, this behaviour is no different from e.g. `ArrayList`.

If a method is marked for inclusion and it has the same name as a field, it replaces the toString output for that field (the method is included, the field is excluded, and the method's output is printed in the place the field would be printed).

Prior to lombok 1.16.22, inclusion/exclusion could be done with the `of` and `exclude` parameters of the `@ToString` annotation. This old-style inclusion mechanism is still supported but will be deprecated in the future.

Having both `@ToString.Exclude` and `@ToString.Include` on a member generates a warning; the member will be excluded in this case.

We don't promise to keep the output of the generated `toString()` methods the same between lombok versions. You should never design your API so that other code is forced to parse your `toString()` output anyway!

By default, any variables that start with a $ symbol are excluded automatically. You can only include them by using the `@ToString.Include` annotation.

If a getter exists for a field to be included, it is called instead of using a direct field reference. This behaviour can be suppressed:
`@ToString(doNotUseGetters = true)`

`@ToString` can also be used on an enum definition.

If you have configured a nullity annotation flavour via [`lombok.config`](https://projectlombok.org/features/configuration) key `lombok.addNullAnnotations`, the method or return type (as appropriate for the chosen flavour) is annotated with a non-null annotation.



# 七、@EqualsAndHashCode

### Equality made easy: Generates `hashCode` and `equals` implementations from the fields of your object.

### Overview

Any class definition may be annotated with `@EqualsAndHashCode` to let lombok generate implementations of the `equals(Object other)` and `hashCode()` methods. By default, it'll use all non-static, non-transient fields, but you can modify which fields are used (and even specify that the output of various methods is to be used) by marking type members with `@EqualsAndHashCode.Include` or `@EqualsAndHashCode.Exclude`. Alternatively, you can specify exactly which fields or methods you wish to be used by marking them with `@EqualsAndHashCode.Include` and using `@EqualsAndHashCode(onlyExplicitlyIncluded = true)`.

If applying `@EqualsAndHashCode` to a class that extends another, this feature gets a bit trickier. Normally, auto-generating an `equals` and `hashCode` method for such classes is a bad idea, as the superclass also defines fields, which also need equals/hashCode code but this code will not be generated. By setting `callSuper` to *true*, you can include the `equals` and `hashCode` methods of your superclass in the generated methods. For `hashCode`, the result of `super.hashCode()` is included in the hash algorithm, and for`equals`, the generated method will return false if the super implementation thinks it is not equal to the passed in object. Be aware that not all `equals` implementations handle this situation properly. However, lombok-generated `equals` implementations **do** handle this situation properly, so you can safely call your superclass equals if it, too, has a lombok-generated `equals` method. If you have an explicit superclass you are forced to supply some value for `callSuper` to acknowledge that you've considered it; failure to do so results in a warning.

Setting `callSuper` to *true* when you don't extend anything (you extend `java.lang.Object`) is a compile-time error, because it would turn the generated `equals()` and `hashCode()` implementations into having the same behaviour as simply inheriting these methods from `java.lang.Object`: only the same object will be equal to each other and will have the same hashCode. Not setting `callSuper` to *true* when you extend another class generates a warning, because unless the superclass has no (equality-important) fields, lombok cannot generate an implementation for you that takes into account the fields declared by your superclasses. You'll need to write your own implementations, or rely on the `callSuper` chaining facility. You can also use the `lombok.equalsAndHashCode.callSuper` config key.

*NEW in Lombok 0.10:* Unless your class is `final` and extends `java.lang.Object`, lombok generates a `canEqual` method which means JPA proxies can still be equal to their base class, but subclasses that add new state don't break the equals contract. The complicated reasons for why such a method is necessary are explained in this paper: [How to Write an Equality Method in Java](https://www.artima.com/lejava/articles/equality.html). If all classes in a hierarchy are a mix of scala case classes and classes with lombok-generated equals methods, all equality will 'just work'. If you need to write your own equals methods, you should always override `canEqual` if you change `equals` and `hashCode`.

*NEW in Lombok 1.14.0:* To put annotations on the `other` parameter of the `equals` (and, if relevant, `canEqual`) method, you can use `onParam=@__({@AnnotationsHere})`. Be careful though! This is an experimental feature. For more details see the documentation on the onX feature.

*NEW in Lombok 1.18.16:* The result of the generated `hashCode()` can be cached by setting `cacheStrategy` to a value other than `CacheStrategy.NEVER`. *Do not* use this if objects of the annotated class can be modified in any way that would lead to the result of `hashCode()` changing.

### With Lombok

```
import lombok.EqualsAndHashCode;

@EqualsAndHashCode
public class EqualsAndHashCodeExample {
  private transient int transientVar = 10;
  private String name;
  private double score;
  @EqualsAndHashCode.Exclude private Shape shape = new Square(5, 10);
  private String[] tags;
  @EqualsAndHashCode.Exclude private int id;
  
  public String getName() {
    return this.name;
  }
  
  @EqualsAndHashCode(callSuper=true)
  public static class Square extends Shape {
    private final int width, height;
    
    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }
  }
}
```

### Vanilla Java

```
import java.util.Arrays;

public class EqualsAndHashCodeExample {
  private transient int transientVar = 10;
  private String name;
  private double score;
  private Shape shape = new Square(5, 10);
  private String[] tags;
  private int id;
  
  public String getName() {
    return this.name;
  }
  
  @Override public boolean equals(Object o) {
    if (o == this) return true;
    if (!(o instanceof EqualsAndHashCodeExample)) return false;
    EqualsAndHashCodeExample other = (EqualsAndHashCodeExample) o;
    if (!other.canEqual((Object)this)) return false;
    if (this.getName() == null ? other.getName() != null : !this.getName().equals(other.getName())) return false;
    if (Double.compare(this.score, other.score) != 0) return false;
    if (!Arrays.deepEquals(this.tags, other.tags)) return false;
    return true;
  }
  
  @Override public int hashCode() {
    final int PRIME = 59;
    int result = 1;
    final long temp1 = Double.doubleToLongBits(this.score);
    result = (result*PRIME) + (this.name == null ? 43 : this.name.hashCode());
    result = (result*PRIME) + (int)(temp1 ^ (temp1 >>> 32));
    result = (result*PRIME) + Arrays.deepHashCode(this.tags);
    return result;
  }
  
  protected boolean canEqual(Object other) {
    return other instanceof EqualsAndHashCodeExample;
  }
  
  public static class Square extends Shape {
    private final int width, height;
    
    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }
    
    @Override public boolean equals(Object o) {
      if (o == this) return true;
      if (!(o instanceof Square)) return false;
      Square other = (Square) o;
      if (!other.canEqual((Object)this)) return false;
      if (!super.equals(o)) return false;
      if (this.width != other.width) return false;
      if (this.height != other.height) return false;
      return true;
    }
    
    @Override public int hashCode() {
      final int PRIME = 59;
      int result = 1;
      result = (result*PRIME) + super.hashCode();
      result = (result*PRIME) + this.width;
      result = (result*PRIME) + this.height;
      return result;
    }
    
    protected boolean canEqual(Object other) {
      return other instanceof Square;
    }
  }
}
```

### Supported configuration keys:

- `lombok.equalsAndHashCode.doNotUseGetters` = [`true` | `false`] (default: false)

    If set to `true`, lombok will access fields directly instead of using getters (if available) when generating `equals` and `hashCode` methods. The annotation parameter '`doNotUseGetters`', if explicitly specified, takes precedence over this setting.

- `lombok.equalsAndHashCode.callSuper` = [`call` | `skip` | `warn`] (default: warn)

    If set to `call`, lombok will generate calls to the superclass implementation of `hashCode` and `equals` if your class extends something. If set to `skip` no such calls are generated. The default behaviour is like `skip`, with an additional warning.

- `lombok.equalsAndHashCode.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@EqualsAndHashCode` as a warning or error if configured.

### Small print

Arrays are 'deep' compared/hashCoded, which means that arrays that contain themselves will result in `StackOverflowError`s. However, this behaviour is no different from e.g. `ArrayList`.

You may safely presume that the hashCode implementation used will not change between versions of lombok, however this guarantee is not set in stone; if there's a significant performance improvement to be gained from using an alternate hash algorithm, that will be substituted in a future version.

For the purposes of equality, 2 `NaN` (not a number) values for floats and doubles are considered equal, eventhough 'NaN == NaN' would return false. This is analogous to `java.lang.Double`'s equals method, and is in fact required to ensure that comparing an object to an exact copy of itself returns `true` for equality.

If there is *any* method named either `hashCode` or `equals`, regardless of return type, no methods will be generated, and a warning is emitted instead. These 2 methods need to be in sync with each other, which lombok cannot guarantee unless it generates all the methods, hence you always get a warning if one *or* both of the methods already exist. You can mark any method with `@lombok.experimental.Tolerate` to hide them from lombok.

Attempting to exclude fields that don't exist or would have been excluded anyway (because they are static or transient) results in warnings on the named fields.

If a method is marked for inclusion and it has the same name as a field, it replaces the field (the method is included, the field is excluded).

Prior to lombok 1.16.22, inclusion/exclusion could be done with the `of` and `exclude` parameters of the `@EqualsAndHashCode` annotation. This old-style inclusion mechanism is still supported but will be deprecated in the future.

By default, any variables that start with a $ symbol are excluded automatically. You can only include them by marking them with `@EqualsAndHashCode.Include`.

If a getter exists for a field to be included, it is called instead of using a direct field reference. This behaviour can be suppressed:
`@EqualsAndHashCode(doNotUseGetters = true)`

If you have configured a nullity annotation flavour via [`lombok.config`](https://projectlombok.org/features/configuration) key `lombok.addNullAnnotations`, the parameter of both the generated `equals` method as well as any `canEqual` method is annotated with a nullable annotation. This is required if you use a `@NonNullByDefault` style annotation in combination with strict nullity checking.



# 八、@NoArgsConstructor, @RequiredArgsConstructor, @AllArgsConstructor

Constructors made to order: Generates constructors that take no arguments, one argument per final / non-null field, or one argument for every field.

### Overview

This set of 3 annotations generate a constructor that will accept 1 parameter for certain fields, and simply assigns this parameter to the field.

`@NoArgsConstructor` will generate a constructor with no parameters. If this is not possible (because of final fields), a compiler error will result instead, unless `@NoArgsConstructor(force = true)` is used, then all final fields are initialized with `0` / `false` / `null`. For fields with constraints, such as `@NonNull` fields, *no* check is generated,so be aware that these constraints will generally not be fulfilled until those fields are properly initialized later. Certain java constructs, such as hibernate and the Service Provider Interface require a no-args constructor. This annotation is useful primarily in combination with either `@Data` or one of the other constructor generating annotations.

`@RequiredArgsConstructor` generates a constructor with 1 parameter for each field that requires special handling. All non-initialized `final` fields get a parameter, as well as any fields that are marked as `@NonNull` that aren't initialized where they are declared. For those fields marked with `@NonNull`, an explicit null check is also generated. The constructor will throw a `NullPointerException` if any of the parameters intended for the fields marked with `@NonNull` contain `null`. The order of the parameters match the order in which the fields appear in your class.

`@AllArgsConstructor` generates a constructor with 1 parameter for each field in your class. Fields marked with `@NonNull` result in null checks on those parameters.

Each of these annotations allows an alternate form, where the generated constructor is always private, and an additional static factory method that wraps around the private constructor is generated. This mode is enabled by supplying the `staticName` value for the annotation, like so: `@RequiredArgsConstructor(staticName="of")`. Such a static factory method will infer generics, unlike a normal constructor. This means your API users get write `MapEntry.of("foo", 5)` instead of the much longer `new MapEntry<String, Integer>("foo", 5)`.

To put annotations on the generated constructor, you can use `onConstructor=@__({@AnnotationsHere})`, but be careful; this is an experimental feature. For more details see the documentation on the [onX](https://projectlombok.org/features/experimental/onX) feature.

Static fields are skipped by these annotations.

Unlike most other lombok annotations, the existence of an explicit constructor does not stop these annotations from generating their own constructor. This means you can write your own specialized constructor, and let lombok generate the boilerplate ones as well. If a conflict arises (one of your constructors ends up with the same signature as one that lombok generates), a compiler error will occur.

### With Lombok

```
import lombok.AccessLevel;
import lombok.RequiredArgsConstructor;
import lombok.AllArgsConstructor;
import lombok.NonNull;

@RequiredArgsConstructor(staticName = "of")
@AllArgsConstructor(access = AccessLevel.PROTECTED)
public class ConstructorExample<T> {
  private int x, y;
  @NonNull private T description;
  
  @NoArgsConstructor
  public static class NoArgsExample {
    @NonNull private String field;
  }
}
```

### Vanilla Java

```

public class ConstructorExample<T> {
  private int x, y;
  @NonNull private T description;
  
  private ConstructorExample(T description) {
    if (description == null) throw new NullPointerException("description");
    this.description = description;
  }
  
  public static <T> ConstructorExample<T> of(T description) {
    return new ConstructorExample<T>(description);
  }
  
  @java.beans.ConstructorProperties({"x", "y", "description"})
  protected ConstructorExample(int x, int y, T description) {
    if (description == null) throw new NullPointerException("description");
    this.x = x;
    this.y = y;
    this.description = description;
  }
  
  public static class NoArgsExample {
    @NonNull private String field;
    
    public NoArgsExample() {
    }
  }
}
```

### Supported configuration keys:

- `lombok.anyConstructor.addConstructorProperties` = [`true` | `false`] (default: `false`)

    If set to `true`, then lombok will add a `@java.beans.ConstructorProperties` to generated constructors.

- `lombok.`[`allArgsConstructor`|`requiredArgsConstructor`|`noArgsConstructor`]`.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of the relevant annotation (`@AllArgsConstructor`, `@RequiredArgsConstructor` or `@NoArgsConstructor`) as a warning or error if configured.

- `lombok.anyConstructor.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of any of the 3 constructor-generating annotations as a warning or error if configured.

- `lombok.copyableAnnotations` = [*A list of fully qualified types*] (default: empty list)

    Lombok will copy any of these annotations from the field to the constructor parameter, the setter parameter, and the getter method. Note that lombok ships with a bunch of annotations 'out of the box' which are known to be copyable: All popular nullable/nonnull annotations.

- `lombok.noArgsConstructor.extraPrivate` = [`true` | `false`] (default: false)

    If `true`, lombok will generate a private no-args constructor for any `@Value` or `@Data` annotated class, which sets all fields to default values (null / 0 / false).

### Small print

Even if a field is explicitly initialized with `null`, lombok will consider the requirement to avoid null as fulfilled, and will *NOT* consider the field as a 'required' argument. The assumption is that if you explicitly assign `null` to a field that you've also marked as `@NonNull` signals you must know what you're doing.

The `@java.beans.ConstructorProperties` annotation is never generated for a constructor with no arguments. This also explains why `@NoArgsConstructor` lacks the `suppressConstructorProperties` annotation method. The generated static factory methods also do not get `@ConstructorProperties`, as this annotation can only be added to real constructors.

`@XArgsConstructor` can also be used on an enum definition. The generated constructor will always be private, because non-private constructors aren't legal in enums. You don't have to specify `AccessLevel.PRIVATE`.

Various well known annotations about nullity cause null checks to be inserted and will be copied to the parameter. See [Getter/Setter](https://projectlombok.org/features/GetterSetter) documentation's small print for more information.

The `flagUsage` configuration keys do not trigger when a constructor is generated by `@Data`, `@Value` or any other lombok annotation.



# 九、@Data

All together now: A shortcut for `@ToString`, `@EqualsAndHashCode`, `@Getter` on all fields, `@Setter` on all non-final fields, and `@RequiredArgsConstructor`!

现在，所有字段上`@ToString`、`@EqualsAndHashCode`、`@Getter`，所有非最终字段上的 `@Setter` 和 `@RequiredArgsConstructor` 的快捷方式都在一起了！

### Overview

`@Data` is a convenient shortcut annotation that bundles the features of [`@ToString`](https://projectlombok.org/features/ToString), [`@EqualsAndHashCode`](https://projectlombok.org/features/EqualsAndHashCode), [`@Getter` / `@Setter`](https://projectlombok.org/features/GetterSetter) and [`@RequiredArgsConstructor`](https://projectlombok.org/features/constructor) together: In other words, `@Data` generates  all the boilerplate that is normally associated with simple POJOs (Plain Old Java Objects) and beans: getters for all fields, setters for all non-final fields, and appropriate `toString`, `equals` and `hashCode` implementations that involve the fields of the class, and a constructor that initializes all final fields, as well as all non-final fields with no initializer that have been marked with `@NonNull`, in order to ensure the field is never null.

`@Data`是一个方便的快捷注释，它捆绑了 `@ToString`，`@EqualsAndHashCode`，`@Getter`/`@Setter`和 `@RequiredArgsConstructor`一起的功能：换句话说，`@Data`生成所有通常与简单 POJO和bean关联的样板文件：所有字段的getter，所有非 final 修饰的字段的 setter，以及涉及类字段的适当的`toString`、`equals`和`hashCode`实现，以及一个构造函数，用于初始化所有最终字段，以及所有未使用已标记为 `@NonNull` 的初始值设定项的非 final 字段，以确保字段从不为 null。

`@Data` is like having implicit `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode` and `@RequiredArgsConstructor` annotations on the class (except that no constructor will be generated if any explicitly written constructors already exist). However, the parameters of these annotations (such as `callSuper`, `includeFieldNames` and `exclude`) cannot be set with `@Data`. If you need to set non-default values for any of these parameters, just add those annotations explicitly; `@Data` is smart enough to defer to those annotations.

`@Data`类似于在类上有隐式的`@Getter`、`@Setter`、`@ToString`、`@EqualsAndHashCode`和`@RequiredArgsConstructor`注释（**除非如果已经存在任何显式编写的构造函数，则不会生成任何构造函数**）。但是，这些声明的参数（如 `callSuper`、`includefeldnames` 和 `exclude`）不能用 `@Data` 设置。**如果需要为这些参数中的任何一个设置非默认值，只需显式添加这些注释**，`@Data`足够聪明，可以遵从这些注释。

All generated getters and setters will be `public`. To override the access level, annotate the field or class with an explicit `@Setter` and/or `@Getter` annotation. You can also use this annotation (by combining it with `AccessLevel.NONE`) to suppress generating a getter and/or setter altogether.

All fields marked as `transient` will not be considered for `hashCode` and `equals`. All static fields will be skipped entirely (not considered for any of the generated methods, and no setter/getter will be made for them).

所有生成的 getter 和 setter 都将是 `public`。要覆盖访问级别，请使用显式的 `@Setter`和/或 `@Getter`注释对字段或类进行注释。**您还可以使用此注释（通过将其与 `AccessLevel.NONE` 结合使用）来完全禁止生成 getter 和/或 setter。**

**对于 `hashCode` 和 `equals` ，将不考虑标记为 `transient` 的所有字段。将完全跳过所有 static 修饰字段**（不考虑任何生成的方法，并且不会为它们生成 setter/getter）。

If the class already contains a method with the same name and parameter count as any method that would normally be generated, that method is not generated, and no warning or error is emitted. For example, if you already have a method with signature `equals(AnyType param)`, no `equals` method will be generated, even though technically it might be an entirely different method due to having different parameter types. The same rule applies to the constructor (any explicit constructor will prevent `@Data` from generating one), as well as `toString`, `equals`, and all getters and setters. You can mark any constructor or method with `@lombok.experimental.Tolerate` to hide them from lombok.

`@Data` can handle generics parameters for fields just fine. In order to reduce the boilerplate when constructing objects for classes with generics, you can use the `staticConstructor` parameter to generate a private constructor, as well as a static method that returns a new instance. This way, javac will infer the variable name. Thus, by declaring like so: `@Data(staticConstructor="of") class Foo<T> { private T x;}` you can create new instances of `Foo` by writing: `Foo.of(5);` instead of having to write: `new Foo<Integer>(5);`.

**如果类已经包含一个与通常生成的任何方法具有相同名称和参数数目的方法，则不会生成该方法，也不会发出警告或错误。例如，如果您已经有一个签名为 `equals（AnyType param）`的方法，则不会生成 `equals`方法，即使从技术上讲，由于参数类型不同，它可能是一个完全不同的方法。同样的规则也适用于构造函数（任何显式构造函数都会阻止 `@Data` 生成构造函数），以及 `toString`、`equals` 和所有 `getter` 和 `setter`。您可以使用 `@lombok.experimental.Tolerate`标记任何构造函数或方法，以对 lombok 隐藏它们。**

`@Data`可以很好地处理字段的泛型参数。为了减少使用泛型为类构造对象时的样板文件，可以使用 `staticConstructor`参数生成私有构造函数，以及返回新实例的静态方法。这样，javac 将推断变量名。因此，通过这样声明：`@Data（staticConstructor=“of”）class Foo<T>{private tx；}`您可以通过编写：`Foo.of(5)`来创建`Foo`的新实例；不必写：`newfoo<Integer>（5）`。

### With Lombok

```java
import lombok.AccessLevel;
import lombok.Setter;
import lombok.Data;
import lombok.ToString;

@Data 
public class DataExample {
    private final String name;
    @Setter(AccessLevel.PACKAGE) 
    private int age;
    private double score;
    private String[] tags;

    @ToString(includeFieldNames=true)
    @Data(staticConstructor="of")
    public static class Exercise<T> {
        private final String name;
        private final T value;
    }
}
```

### Vanilla Java

```java
import java.util.Arrays;

public class DataExample {
    private final String name;
    private int age;
    private double score;
    private String[] tags;

    public DataExample(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }

    void setAge(int age) {
        this.age = age;
    }

    public int getAge() {
        return this.age;
    }

    public void setScore(double score) {
        this.score = score;
    }

    public double getScore() {
        return this.score;
    }

    public String[] getTags() {
        return this.tags;
    }

    public void setTags(String[] tags) {
        this.tags = tags;
    }

    @Override 
    public String toString() {
        return "DataExample(" + this.getName() + ", " + this.getAge() + ", " + this.getScore() + ", " + Arrays.deepToString(this.getTags()) + ")";
    }

    protected boolean canEqual(Object other) {
        return other instanceof DataExample;
    }

    @Override 
    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof DataExample)) return false;
        DataExample other = (DataExample) o;
        if (!other.canEqual((Object)this)) return false;
        if (this.getName() == null ? other.getName() != null : !this.getName().equals(other.getName())) return false;
        if (this.getAge() != other.getAge()) return false;
        if (Double.compare(this.getScore(), other.getScore()) != 0) return false;
        if (!Arrays.deepEquals(this.getTags(), other.getTags())) return false;
        return true;
    }

    @Override 
    public int hashCode() {
        final int PRIME = 59;
        int result = 1;
        final long temp1 = Double.doubleToLongBits(this.getScore());
        result = (result*PRIME) + (this.getName() == null ? 43 : this.getName().hashCode());
        result = (result*PRIME) + this.getAge();
        result = (result*PRIME) + (int)(temp1 ^ (temp1 >>> 32));
        result = (result*PRIME) + Arrays.deepHashCode(this.getTags());
        return result;
    }

    public static class Exercise<T> {
        private final String name;
        private final T value;

        private Exercise(String name, T value) {
            this.name = name;
            this.value = value;
        }

        public static <T> Exercise<T> of(String name, T value) {
            return new Exercise<T>(name, value);
        }

        public String getName() {
            return this.name;
        }

        public T getValue() {
            return this.value;
        }

        @Override 
        public String toString() {
            return "Exercise(name=" + this.getName() + ", value=" + this.getValue() + ")";
        }

        protected boolean canEqual(Object other) {
            return other instanceof Exercise;
        }

        @Override 
        public boolean equals(Object o) {
            if (o == this) return true;
            if (!(o instanceof Exercise)) return false;
            Exercise<?> other = (Exercise<?>) o;
            if (!other.canEqual((Object)this)) return false;
            if (this.getName() == null ? other.getValue() != null : !this.getName().equals(other.getName())) return false;
            if (this.getValue() == null ? other.getValue() != null : !this.getValue().equals(other.getValue())) return false;
            return true;
        }

        @Override 
        public int hashCode() {
            final int PRIME = 59;
            int result = 1;
            result = (result*PRIME) + (this.getName() == null ? 43 : this.getName().hashCode());
            result = (result*PRIME) + (this.getValue() == null ? 43 : this.getValue().hashCode());
            return result;
        }
    }
}
```

### Supported configuration keys:

- `lombok.data.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@Data` as a warning or error if configured.

    Lombok 会将任何使用`@Data`的情况标记为警告或错误（如果已配置）。

- `lombok.noArgsConstructor.extraPrivate` = [`true` | `false`] (default: false)

    If `true`, lombok will generate a private no-args constructor for any `@Data` annotated class, which sets all fields to default values (null / 0 / false).
    
    如果 `true`，lombok 将为任何带 `@Data` 注释的类生成一个私有的无参数构造函数，该构造函数将所有字段设置为默认值（null/0/false）。

### Small print 附属细则

See the small print of `@ToString`, `@EqualsAndHashCode`, `@Getter / @Setter` and `@RequiredArgsConstructor`。

Various well known annotations about nullity cause null checks to be inserted and will be copied to the relevant places (such as the method for getters, and the parameter for the constructor and setters). See [Getter/Setter](https://projectlombok.org/features/GetterSetter) documentation's small print for more information.

By default, any variables that start with a $ symbol are excluded automatically. You can include them by specifying an explicit annotation (`@Getter` or `@ToString`, for example) and using the 'of' parameter.

请参阅 `@ToString`, `@EqualsAndHashCode`, `@Getter / @Setter` and `@RequiredArgsConstructor`的附属细则。

关于null性的各种众所周知的注释会导致插入null检查并将其复制到相关位置（例如getter的方法以及构造函数和setter的参数）。参见[Getter/Setter](https://projectlombok.org/features/GetterSetter)有关更多信息，请参阅文档的小附属细则。

默认情况下，将自动排除以$符号开头的任何变量。您可以通过指定显式注释（`Getter`或`ToString`，例如）并使用'of'参数来包含它们。

# 十、@Value

### Immutable classes made very easy.

`@Value` was introduced as experimental feature in lombok v0.11.4.

`@Value` no longer implies `@With` since lombok v0.11.8.

`@Value` promoted to the main `lombok` package since lombok v0.12.0.

### Overview

`@Value` is the immutable variant of [`@Data`](https://projectlombok.org/features/Data); all fields are made `private` and `final` by default, and setters are not generated. The class itself is also made `final` by default, because immutability is not something that can be forced onto a subclass. Like `@Data`, useful `toString()`, `equals()` and `hashCode()` methods are also generated, each field gets a getter method, and a constructor that covers every argument (except `final` fields that are initialized in the field declaration) is also generated.

In practice, `@Value` is shorthand for: `final @ToString @EqualsAndHashCode @AllArgsConstructor @FieldDefaults(makeFinal = true, level = AccessLevel.PRIVATE) @Getter`, except that explicitly including an implementation of any of the relevant methods simply means that part won't be generated and no warning will be emitted. For example, if you write your own `toString`, no error occurs, and lombok will not generate a `toString`. Also, *any* explicit constructor, no matter the arguments list, implies lombok will not generate a constructor. If you do want lombok to generate the all-args constructor, add `@AllArgsConstructor` to the class. Note that if both `@Builder` and `@Value` are on a class, the package private allargs constructor that `@Builder` wants to make 'wins' over the public one that `@Value` wants to make. You can mark any constructor or method with `@lombok.experimental.Tolerate` to hide them from lombok.

It is possible to override the final-by-default and private-by-default behavior using either an explicit access level on a field, or by using the `@NonFinal` or `@PackagePrivate` annotations. `@NonFinal` can also be used on a class to remove the final keyword.
It is possible to override any default behavior for any of the 'parts' that make up `@Value` by explicitly using that annotation.

### With Lombok

```

import lombok.AccessLevel;
import lombok.experimental.NonFinal;
import lombok.experimental.Value;
import lombok.experimental.With;
import lombok.ToString;

@Value public class ValueExample {
  String name;
  @With(AccessLevel.PACKAGE) @NonFinal int age;
  double score;
  protected String[] tags;
  
  @ToString(includeFieldNames=true)
  @Value(staticConstructor="of")
  public static class Exercise<T> {
    String name;
    T value;
  }
}
```

### Vanilla Java

```java
import java.util.Arrays;

public final class ValueExample {
    private final String name;
    private int age;
    private final double score;
    protected final String[] tags;

    @java.beans.ConstructorProperties({"name", "age", "score", "tags"})
        public ValueExample(String name, int age, double score, String[] tags) {
        this.name = name;
        this.age = age;
        this.score = score;
        this.tags = tags;
    }

    public String getName() {
        return this.name;
    }

    public int getAge() {
        return this.age;
    }

    public double getScore() {
        return this.score;
    }

    public String[] getTags() {
        return this.tags;
    }

    @java.lang.Override
        public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof ValueExample)) return false;
        final ValueExample other = (ValueExample)o;
        final Object this$name = this.getName();
        final Object other$name = other.getName();
        if (this$name == null ? other$name != null : !this$name.equals(other$name)) return false;
        if (this.getAge() != other.getAge()) return false;
        if (Double.compare(this.getScore(), other.getScore()) != 0) return false;
        if (!Arrays.deepEquals(this.getTags(), other.getTags())) return false;
        return true;
    }

    @java.lang.Override
        public int hashCode() {
        final int PRIME = 59;
        int result = 1;
        final Object $name = this.getName();
        result = result * PRIME + ($name == null ? 43 : $name.hashCode());
        result = result * PRIME + this.getAge();
        final long $score = Double.doubleToLongBits(this.getScore());
        result = result * PRIME + (int)($score >>> 32 ^ $score);
        result = result * PRIME + Arrays.deepHashCode(this.getTags());
        return result;
    }

    @java.lang.Override
        public String toString() {
        return "ValueExample(name=" + getName() + ", age=" + getAge() + ", score=" + getScore() + ", tags=" + Arrays.deepToString(getTags()) + ")";
    }

    ValueExample withAge(int age) {
        return this.age == age ? this : new ValueExample(name, age, score, tags);
    }

    public static final class Exercise<T> {
        private final String name;
        private final T value;

        private Exercise(String name, T value) {
            this.name = name;
            this.value = value;
        }

        public static <T> Exercise<T> of(String name, T value) {
            return new Exercise<T>(name, value);
        }

        public String getName() {
            return this.name;
        }

        public T getValue() {
            return this.value;
        }

        @java.lang.Override
            public boolean equals(Object o) {
            if (o == this) return true;
            if (!(o instanceof ValueExample.Exercise)) return false;
            final Exercise<?> other = (Exercise<?>)o;
            final Object this$name = this.getName();
            final Object other$name = other.getName();
            if (this$name == null ? other$name != null : !this$name.equals(other$name)) return false;
            final Object this$value = this.getValue();
            final Object other$value = other.getValue();
            if (this$value == null ? other$value != null : !this$value.equals(other$value)) return false;
            return true;
        }

        @java.lang.Override
            public int hashCode() {
            final int PRIME = 59;
            int result = 1;
            final Object $name = this.getName();
            result = result * PRIME + ($name == null ? 43 : $name.hashCode());
            final Object $value = this.getValue();
            result = result * PRIME + ($value == null ? 43 : $value.hashCode());
            return result;
        }

        @java.lang.Override
            public String toString() {
            return "ValueExample.Exercise(name=" + getName() + ", value=" + getValue() + ")";
        }
    }
}
```

### Supported configuration keys:

- `lombok.value.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@Value` as a warning or error if configured.

- `lombok.noArgsConstructor.extraPrivate` = [`true` | `false`] (default: false)

    If `true`, lombok will generate a private no-args constructor for any `@Value` annotated class, which sets all fields to default values (null / 0 / false).

### Small print

Look for the documentation on the 'parts' of `@Value`: [`@ToString`](https://projectlombok.org/features/ToString), [`@EqualsAndHashCode`](https://projectlombok.org/features/EqualsAndHashCode), [`@AllArgsConstructor`](https://projectlombok.org/features/Constructor), [`@FieldDefaults`](https://projectlombok.org/features/experimental/FieldDefaults), and [`@Getter`](https://projectlombok.org/features/GetterSetter).

For classes with generics, it's useful to have a static method which serves as a constructor, because inference of generic parameters via static methods works in java6 and avoids having to use the diamond operator. While you can force this by applying an explicit `@AllArgsConstructor(staticConstructor="of")` annotation, there's also the `@Value(staticConstructor="of")` feature, which will make the generated all-arguments constructor private, and generates a public static method named `of` which is a wrapper around this private constructor.

Various well known annotations about nullity cause null checks to be inserted and will be copied to the relevant places (such as the method for getters, and the parameter for the constructor and setters). See [Getter/Setter](https://projectlombok.org/features/GetterSetter) documentation's small print for more information.

`@Value` was an experimental feature from v0.11.4 to v0.11.9 (as `@lombok.experimental.Value`). It has since been moved into the core package. The old annotation is still around (and is an alias). It will eventually be removed in a future version, though.

It is not possible to use `@FieldDefaults` to 'undo' the private-by-default and final-by-default aspect of fields in the annotated class. Use `@NonFinal` and `@PackagePrivate` on the fields in the class to override this behaviour.



# 十一、@Builder

### ... and Bob's your uncle: No-hassle fancy-pants APIs for object creation!

`@Builder` was introduced as experimental feature in lombok v0.12.0.

`@Builder` gained `@Singular` support and was promoted to the main `lombok` package since lombok v1.16.0.

`@Builder` with `@Singular` adds a clear method since lombok v1.16.8.

`@Builder.Default` functionality was added in lombok v1.16.16.

`@Builder(builderMethodName = "")` is legal (and will suppress generation of the builder method) starting with lombok v1.18.8.

`@Builder(access = AccessLevel.PACKAGE)` is legal (and will generate the builder class, the builder method, etc with the indicated access level) starting with lombok v1.18.8.

### Overview

The `@Builder` annotation produces complex builder APIs for your classes.

`@Builder` lets you automatically produce the code required to have your class be instantiable with code such as:

```java
Person.builder()
.name("Adam Savage")
.city("San Francisco")
.job("Mythbusters")
.job("Unchained Reaction")
.build();
```

`@Builder` can be placed on a class, or on a constructor, or on a method. While the "on a class" and "on a constructor" mode are the most common use-case, `@Builder` is most easily explained with the "method" use-case.

A method annotated with `@Builder` (from now on called the *target*) causes the following 7 things to be generated:

- An inner static class named `*Foo*Builder`, with the same type arguments as the static method (called the *builder*).
- In the *builder*: One private non-static non-final field for each parameter of the *target*.
- In the *builder*: A package private no-args empty constructor.
- In the *builder*: A 'setter'-like method for each parameter of the *target*: It has the same type as that parameter and the same name. It returns the builder itself, so that the setter calls can be chained, as in the above example.
- In the *builder*: A `build()` method which calls the method, passing in each field. It returns the same type that the *target* returns.
- In the *builder*: A sensible `toString()` implementation.
- In the class containing the *target*: A `builder()` method, which creates a new instance of the *builder*.

Each listed generated element will be silently skipped if that element already exists (disregarding parameter counts and looking only at names). This includes the *builder* itself: If that class already exists, lombok will simply start injecting fields and methods inside this already existing class, unless of course the fields / methods to be injected already exist. You may not put any other method (or constructor) generating lombok annotation on a builder class though; for example, you can not put `@EqualsAndHashCode` on the builder class.



`@Builder` can generate so-called 'singular' methods for collection parameters/fields. These take 1 element instead of an entire list, and add the element to the list. For example:

```java
Person.builder()
.job("Mythbusters")
.job("Unchained Reaction")
.build();
```

would result in the `List<String> jobs` field to have 2 strings in it. To get this behavior, the field/parameter needs to be annotated with `@Singular`. The feature has [its own documentation](https://projectlombok.org/features/Builder#singular).

Now that the "method" mode is clear, putting a `@Builder` annotation on a constructor functions similarly; effectively, constructors are just static methods that have a special syntax to invoke them: Their 'return type' is the class they construct, and their type parameters are the same as the type parameters of the class itself.

Finally, applying `@Builder` to a class is as if you added `@AllArgsConstructor(access = AccessLevel.PACKAGE)` to the class and applied the `@Builder` annotation to this all-args-constructor. This only works if you haven't written any explicit constructors yourself. If you do have an explicit constructor, put the `@Builder` annotation on the constructor instead of on the class. Note that if you put both `@Value` and `@Builder` on a class, the package-private constructor that `@Builder` wants to generate 'wins' and suppresses the constructor that `@Value` wants to make.

If using `@Builder` to generate builders to produce instances of your own class (this is always the case unless adding `@Builder` to a method that doesn't return your own type), you can use `@Builder(toBuilder = true)` to also generate an instance method in your class called `toBuilder()`; it creates a new builder that starts out with all the values of this instance. You can put the `@Builder.ObtainVia` annotation on the parameters (in case of a constructor or method) or fields (in case of `@Builder` on a type) to indicate alternative means by which the value for that field/parameter is obtained from this instance. For example, you can specify a method to be invoked: `@Builder.ObtainVia(method = "calculateFoo")`.

The name of the builder class is `*Foobar*Builder`, where *Foobar* is the simplified, title-cased form of the return type of the *target* - that is, the name of your type for `@Builder` on constructors and types, and the name of the return type for `@Builder` on methods. For example, if `@Builder` is applied to a class named `com.yoyodyne.FancyList<T>`, then the builder name will be `FancyListBuilder<T>`. If `@Builder` is applied to a method that returns `void`, the builder will be named `VoidBuilder`.

The configurable aspects of builder are:

- The *builder's class name* (default: return type + 'Builder')
- The *build()* method's name (default: `"build"`)
- The *builder()* method's name (default: `"builder"`)
- If you want `toBuilder()` (default: no)
- The access level of all generated elements (default: `public`).
- (discouraged) If you want your builder's 'set' methods to have a prefix, i.e. `Person.builder().setName("Jane").build()` instead of `Person.builder().name("Jane").build()` and what it should be.

Example usage where all options are changed from their defaults:
`@Builder(builderClassName = "HelloWorldBuilder", buildMethodName = "execute", builderMethodName = "helloWorld", toBuilder = true, access = AccessLevel.PRIVATE, setterPrefix = "set")`

Looking to use your builder with [Jackson](https://github.com/FasterXML/jackson), the JSON/XML tool? We have you covered: Check out the [@Jacksonized](https://projectlombok.org/features/experimental/Jacksonized) feature.

### @Builder.Default

If a certain field/parameter is never set during a build session, then it always gets 0 / `null` / false. If you've put `@Builder` on a class (and not a method or constructor) you can instead specify the default directly on the field, and annotate the field with `@Builder.Default`:
`@Builder.Default private final long created = System.currentTimeMillis();`

### @Singular

By annotating one of the parameters (if annotating a method or constructor with `@Builder`) or fields (if annotating a class with `@Builder`) with the `@Singular` annotation, lombok will treat that builder node as a collection, and it generates 2 'adder' methods instead of a 'setter' method. One which adds a single element to the collection, and one which adds all elements of another collection to the collection. No setter to just set the collection (replacing whatever was already added) will be generated. A 'clear' method is also generated. These 'singular' builders are very complicated in order to guarantee the following properties:

- When invoking `build()`, the produced collection will be immutable.
- Calling one of the 'adder' methods, or the 'clear' method, after invoking `build()` does not modify any already generated objects, and, if `build()` is later called again, another collection with all the elements added since the creation of the builder is generated.
- The produced collection will be compacted to the smallest feasible format while remaining efficient.



`@Singular` can only be applied to collection types known to lombok. Currently, the supported types are:

- `java.util`:
    - `Iterable`, `Collection`, and `List` (backed by a compacted unmodifiable `ArrayList` in the general case).
    - `Set`, `SortedSet`, and `NavigableSet` (backed by a smartly sized unmodifiable `HashSet` or `TreeSet` in the general case).
    - `Map`, `SortedMap`, and `NavigableMap` (backed by a smartly sized unmodifiable `HashMap` or `TreeMap` in the general case).
- Guava's com.google.common.collect:
    - `ImmutableCollection` and `ImmutableList` (backed by the builder feature of `ImmutableList`).
    - `ImmutableSet` and `ImmutableSortedSet` (backed by the builder feature of those types).
    - `ImmutableMap`, `ImmutableBiMap`, and `ImmutableSortedMap` (backed by the builder feature of those types).
    - `ImmutableTable` (backed by the builder feature of `ImmutableTable`).

If your identifiers are written in common english, lombok assumes that the name of any collection with `@Singular` on it is an english plural and will attempt to automatically singularize that name. If this is possible, the add-one method will use this name. For example, if your collection is called `statuses`, then the add-one method will automatically be called `status`. You can also specify the singular form of your identifier explicitly by passing the singular form as argument to the annotation like so: `@Singular("axis") List<Line> axes;`.
If lombok cannot singularize your identifier, or it is ambiguous, lombok will generate an error and force you to explicitly specify the singular name.

The snippet below does not show what lombok generates for a `@Singular` field/parameter because it is rather complicated. You can view a snippet [here](https://projectlombok.org/features/builderSingular).

If also using `setterPrefix = "with"`, the generated names are, for example, `withName` (add 1 name), `withNames` (add many names), and `clearNames` (reset all names).

Ordinarily, the generated 'plural form' method (which takes in a collection, and adds each element in this collection) will check if a `null` is passed the same way [`@NonNull`](https://projectlombok.org/features/NonNull) does (by default, throws a `NullPointerException` with an appropriate message). However, you can also tell lombok to ignore such collection (so, add nothing, return immediately): `@Singular(ignoreNullCollections = true`.

### With Jackson

You can customize parts of your builder, for example adding another method to the builder class, or annotating a method in the builder class, by making the builder class yourself. Lombok will generate everything that you do not manually add, and put it into this builder class. For example, if you are trying to configure [jackson](https://github.com/FasterXML/jackson) to use a specific subtype for a collection, you can write something like:

```
@Value @Builder
@JsonDeserialize(builder = JacksonExample.JacksonExampleBuilder.class)
public class JacksonExample {
	@Singular(nullBehavior = NullCollectionBehavior.IGNORE) private List<Foo> foos;
	
	@JsonPOJOBuilder(withPrefix = "")
	public static class JacksonExampleBuilder implements JacksonExampleBuilderMeta {
	}
	
	private interface JacksonExampleBuilderMeta {
		@JsonDeserialize(contentAs = FooImpl.class) JacksonExampleBuilder foos(List<? extends Foo> foos)
	}
}
```

### With Lombok

```
import lombok.Builder;
import lombok.Singular;
import java.util.Set;

@Builder
public class BuilderExample {
  @Builder.Default private long created = System.currentTimeMillis();
  private String name;
  private int age;
  @Singular private Set<String> occupations;
}
```

### Vanilla Java

```
import java.util.Set;

public class BuilderExample {
  private long created;
  private String name;
  private int age;
  private Set<String> occupations;
  
  BuilderExample(String name, int age, Set<String> occupations) {
    this.name = name;
    this.age = age;
    this.occupations = occupations;
  }
  
  private static long $default$created() {
    return System.currentTimeMillis();
  }
  
  public static BuilderExampleBuilder builder() {
    return new BuilderExampleBuilder();
  }
  
  public static class BuilderExampleBuilder {
    private long created;
    private boolean created$set;
    private String name;
    private int age;
    private java.util.ArrayList<String> occupations;
    
    BuilderExampleBuilder() {
    }
    
    public BuilderExampleBuilder created(long created) {
      this.created = created;
      this.created$set = true;
      return this;
    }
    
    public BuilderExampleBuilder name(String name) {
      this.name = name;
      return this;
    }
    
    public BuilderExampleBuilder age(int age) {
      this.age = age;
      return this;
    }
    
    public BuilderExampleBuilder occupation(String occupation) {
      if (this.occupations == null) {
        this.occupations = new java.util.ArrayList<String>();
      }
      
      this.occupations.add(occupation);
      return this;
    }
    
    public BuilderExampleBuilder occupations(Collection<? extends String> occupations) {
      if (this.occupations == null) {
        this.occupations = new java.util.ArrayList<String>();
      }

      this.occupations.addAll(occupations);
      return this;
    }
    
    public BuilderExampleBuilder clearOccupations() {
      if (this.occupations != null) {
        this.occupations.clear();
      }
      
      return this;
    }

    public BuilderExample build() {
      // complicated switch statement to produce a compact properly sized immutable set omitted.
      Set<String> occupations = ...;
      return new BuilderExample(created$set ? created : BuilderExample.$default$created(), name, age, occupations);
    }
    
    @java.lang.Override
    public String toString() {
      return "BuilderExample.BuilderExampleBuilder(created = " + this.created + ", name = " + this.name + ", age = " + this.age + ", occupations = " + this.occupations + ")";
    }
  }
}
```

### Supported configuration keys:

- `lombok.builder.className` = [a java identifier with an optional star to indicate where the return type name goes] (default: `*Builder`)

    Unless you explicitly pick the builder's class name with the `builderClassName` parameter, this name is chosen; any star in the name is replaced with the relevant return type.

- `lombok.builder.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@Builder` as a warning or error if configured.

- `lombok.singular.useGuava` = [`true` | `false`] (default: false)

    If `true`, lombok will use guava's `ImmutableXxx` builders and types to implement `java.util` collection interfaces, instead of creating implementations based on `Collections.unmodifiableXxx`. You must ensure that guava is actually available on the classpath and buildpath if you use this setting. Guava is used automatically if your field/parameter has one of the guava `ImmutableXxx` types.

- `lombok.singular.auto` = [`true` | `false`] (default: true)

    If `true` (which is the default), lombok automatically tries to singularize your identifier name by assuming that it is a common english plural. If `false`, you must always explicitly specify the singular name, and lombok will generate an error if you don't (useful if you write your code in a language other than english).

### Small print

@Singular support for `java.util.NavigableMap/Set` only works if you are compiling with JDK1.8 or higher.

You cannot manually provide some or all parts of a `@Singular` node; the code lombok generates is too complex for this. If you want to manually control (part of) the builder code associated with some field or parameter, don't use `@Singular` and add everything you need manually.

The sorted collections (java.util: `SortedSet`, `NavigableSet`, `SortedMap`, `NavigableMap` and guava: `ImmutableSortedSet`, `ImmutableSortedMap`) require that the type argument of the collection has natural order (implements `java.util.Comparable`). There is no way to pass an explicit `Comparator` to use in the builder.

An `ArrayList` is used to store added elements as call methods of a `@Singular` marked field, if the target collection is from the `java.util` package, *even if the collection is a set or map*. Because lombok ensures that generated collections are compacted, a new backing instance of a set or map must be constructed anyway, and storing the data as an `ArrayList` during the build process is more efficient that storing it as a map or set. This behavior is not externally visible, an implementation detail of the current implementation of the `java.util` recipes for `@Singular @Builder`.

With `toBuilder = true` applied to methods, any type parameter of the annotated method itself must also show up in the return type.

The initializer on a `@Builder.Default` field is removed and stored in a static method, in order to guarantee that this initializer won't be executed at all if a value is specified in the build. This does mean the initializer cannot refer to `this`, `super` or any non-static member. If lombok generates a constructor for you, it'll also initialize this field with the initializer.

The generated field in the builder to represent a field with a `@Builder.Default` set is called `*propertyName*$value`; an additional boolean field called `*propertyName*$set` is also generated to track whether it has been set or not. This is an implementation detail; do not write code that interacts with these fields. Instead, invoke the generated builder-setter method if you want to set the property inside a custom method inside the builder.

Various well known annotations about nullity cause null checks to be inserted and will be copied to parameter of the builder's 'setter' method. See [Getter/Setter](https://projectlombok.org/features/GetterSetter) documentation's small print for more information.

You can suppress the generation of the `builder()` method, for example because you *just* want the `toBuilder()` functionality, by using: `@Builder(builderMethodName = "")`. Any warnings about missing `@Builder.Default` annotations will disappear when you do this, as such warnings are not relevant when only using `toBuilder()` to make builder instances.

You can use `@Builder` for copy constructors: `foo.toBuilder().build()` makes a shallow clone. Consider suppressing the generating of the `builder` method if you just want this functionality, by using: `@Builder(toBuilder = true, builderMethodName = "")`.

Due to a peculiar way javac processes static imports, trying to do a non-star static import of the static `builder()` method won't work. Either use a star static import: `import static TypeThatHasABuilder.*;` or don't statically import the `builder` method.

If setting the access level to `PROTECTED`, all methods generated inside the builder class are actually generated as `public`; the meaning of the `protected` keyword is different inside the inner class, and the precise behavior that `PROTECTED` would indicate (access by any source in the same package is allowed, as well as any subclasses *from the outer class, marked with `@Builder`* is not possible, and marking the inner members `public` is as close as we can get.

If you have configured a nullity annotation flavour via [`lombok.config`](https://projectlombok.org/features/configuration) key `lombok.addNullAnnotations`, any plural-form generated builder methods for `@Singular` marked properties (these plural form methods take a collection of some sort and add all elements) get a nullity annotation on the parameter. You get a non-null one normally, but if you have configured the behavior on `null` being passed in as collection to `IGNORE`, a nullable annotation is generated instead.





# 十二、@SneakyThrows

### To boldly throw checked exceptions where no one has thrown them before!

### Overview

`@SneakyThrows` can be used to sneakily throw checked exceptions without actually declaring this in your method's `throws` clause. This somewhat contentious ability should be used carefully, of course. The code generated by lombok will not ignore, wrap, replace, or otherwise modify the thrown checked exception; it simply fakes out the compiler. On the JVM (class file) level, all exceptions, checked or not, can be thrown regardless of the `throws` clause of your methods, which is why this works.

Common use cases for when you want to opt out of the checked exception mechanism center around 2 situations:

- A needlessly strict interface, such as `Runnable` - whatever exception propagates out of your `run()` method, checked or not, it will be passed to the `Thread`'s unhandled exception handler. Catching a checked exception and wrapping it in some sort of `RuntimeException` is only obscuring the real cause of the issue.
- An 'impossible' exception. For example, `new String(someByteArray, "UTF-8");` declares that it can throw an `UnsupportedEncodingException` but according to the JVM specification, UTF-8 *must* always be available. An `UnsupportedEncodingException` here is about as likely as a `ClassNotFoundError` when you use a String object, and you don't catch those either!

Being constrained by needlessly strict interfaces is particularly common when using lambda syntax (`arg -> action`); however, lambdas cannot be annotated, which means it is not so easy to use `@SneakyThrows` in combination with lambdas.

Be aware that it is *impossible* to catch sneakily thrown checked types directly, as javac will not let you write a catch block for an exception type that no method call in the try body declares as thrown. This problem is not relevant in either of the use cases listed above, so let this serve as a warning that you should not use the `@SneakyThrows` mechanism without some deliberation!

You can pass any number of exceptions to the `@SneakyThrows` annotation. If you pass no exceptions, you may throw any exception sneakily.

### With Lombok

```java
import lombok.SneakyThrows;

public class SneakyThrowsExample implements Runnable {
  @SneakyThrows(UnsupportedEncodingException.class)
  public String utf8ToString(byte[] bytes) {
    return new String(bytes, "UTF-8");
  }
  
  @SneakyThrows
  public void run() {
    throw new Throwable();
  }
}
```

### Vanilla Java

```java

import lombok.Lombok;

public class SneakyThrowsExample implements Runnable {
  public String utf8ToString(byte[] bytes) {
    try {
      return new String(bytes, "UTF-8");
    } catch (UnsupportedEncodingException e) {
      throw Lombok.sneakyThrow(e);
    }
  }
  
  public void run() {
    try {
      throw new Throwable();
    } catch (Throwable t) {
      throw Lombok.sneakyThrow(t);
    }
  }
}
```

### Supported configuration keys:

- `lombok.sneakyThrows.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@SneakyThrows` as a warning or error if configured.

### Small print

Because `@SneakyThrows` is an implementation detail and not part of your method signature, it is an error if you try to declare a checked exception as sneakily thrown when you don't call any methods that throw this exception. (Doing so is perfectly legal for `throws` statements to accommodate subclasses). Similarly, `@SneakyThrows` does not inherit.

For the nay-sayers in the crowd: Out of the box, Eclipse will offer a 'quick-fix' for uncaught exceptions that wraps the offending statement in a try/catch block with just `e.printStackTrace()` in the catch block. This is so spectacularly non-productive compared to just sneakily throwing the exception onwards, that Roel and Reinier feel more than justified in claiming that the checked exception system is far from perfect, and thus an opt-out mechanism is warranted.

If you put `@SneakyThrows` on a constructor, any call to a sibling or super constructor is *excluded* from the `@SneakyThrows` treatment. This is a java restriction we cannot work around: Calls to sibling/super constructors MUST be the first statement in the constructor; they cannot be placed inside try/catch blocks.

`@SneakyThrows` on an empty method, or a constructor that is empty or only has a call to a sibling / super constructor results in no try/catch block and a warning.



# 十三、@Synchronized

### `synchronized` done right: Don't expose your locks.

### Overview

`@Synchronized` is a safer variant of the `synchronized` method modifier. Like `synchronized`, the annotation can be used on static and instance methods only. It operates similarly to the `synchronized` keyword, but it locks on different objects. The keyword locks on `this`, but the annotation locks on a field named `$lock`, which is private.
If the field does not exist, it is created for you. If you annotate a `static` method, the annotation locks on a static field named `$LOCK` instead.

If you want, you can create these locks yourself. The `$lock` and `$LOCK` fields will of course not be generated if you already created them yourself. You can also choose to lock on another field, by specifying it as parameter to the `@Synchronized` annotation. In this usage variant, the fields will not be created automatically, and you must explicitly create them yourself, or an error will be emitted.

Locking on `this` or your own class object can have unfortunate side-effects, as other code not under your control can lock on these objects as well, which can cause race conditions and other nasty threading-related bugs.

### With Lombok

```

import lombok.Synchronized;

public class SynchronizedExample {
  private final Object readLock = new Object();
  
  @Synchronized
  public static void hello() {
    System.out.println("world");
  }
  
  @Synchronized
  public int answerToLife() {
    return 42;
  }
  
  @Synchronized("readLock")
  public void foo() {
    System.out.println("bar");
  }
}
```

### Vanilla Java

```

public class SynchronizedExample {
  private static final Object $LOCK = new Object[0];
  private final Object $lock = new Object[0];
  private final Object readLock = new Object();
  
  public static void hello() {
    synchronized($LOCK) {
      System.out.println("world");
    }
  }
  
  public int answerToLife() {
    synchronized($lock) {
      return 42;
    }
  }
  
  public void foo() {
    synchronized(readLock) {
      System.out.println("bar");
    }
  }
}
```

### Supported configuration keys:

- `lombok.synchronized.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@Synchronized` as a warning or error if configured.

### Small print

If `$lock` and/or `$LOCK` are auto-generated, the fields are initialized with an empty `Object[]` array, and not just a `new Object()` as most snippets showing this pattern in action use. Lombok does this because a new object is *NOT* serializable, but 0-size array is. Therefore, using `@Synchronized` will not prevent your object from being serialized.

Having at least one `@Synchronized` method in your class means there will be a lock field, but if you later remove all such methods, there will no longer be a lock field. That means your predetermined `serialVersionUID` changes. We suggest you *always* add a `serialVersionUID` to your classes if you intend to store them long-term via java's serialization mechanism. If you do so, removing all `@Synchronized` annotations from your method will not break serialization.

If you'd like to know why a field is not automatically generated when you choose your own name for the lock object: Because otherwise making a typo in the field name will result in a *very* hard to find bug!



# 十四、@With

### Immutable 'setters' - methods that create a clone but with one changed field.

`@Wither` was introduced as experimental feature in lombok v0.11.4.

`@Wither` was renamed to `@With`, and moved out of experimental and into the core package, in lombok v1.18.10.

### Overview

The next best alternative to a setter for an immutable property is to construct a clone of the object, but with a new value for this one field. A method to generate this clone is precisely what `@With` generates: a `withFieldName(newValue)` method which produces a clone except for the new value for the associated field.

For example, if you create `public class Point { private final int x, y; }`, setters make no sense because the fields are final. `@With` can generate a `withX(int newXValue)` method for you which will return a new point with the supplied value for `x` and the same value for `y`.

The `@With` relies on a constructor for all fields in order to do its work. If this constructor does not exist, your `@With` annotation will result in a compile time error message. You can use Lombok's own [`@AllArgsConstructor`](https://projectlombok.org/features/constructor), or as [`Value`](https://projectlombok.org/features/Value) will automatically produce an all args constructor as well, you can use that too. It's of course also acceptable if you manually write this constructor. It must contain all non-static fields, in the same lexical order.

Like [`@Setter`](https://projectlombok.org/features/GetterSetter), you can specify an access level in case you want the generated with method to be something other than `public`:
`@With(level = AccessLevel.PROTECTED)`. Also like [`@Setter`](https://projectlombok.org/features/GetterSetter), you can also put a `@With` annotation on a type, which means a `with` method is generated for each field (even non-final fields).

To put annotations on the generated method, you can use `onMethod=@__({@AnnotationsHere})`. Be careful though! This is an experimental feature. For more details see the documentation on the [onX](https://projectlombok.org/features/experimental/onX) feature.

javadoc on the field will be copied to generated with methods. Normally, all text is copied, and `@param` is *moved* to the with method, whilst `@return` lines are stripped from the with method's javadoc. Moved means: Deleted from the field's javadoc. It is also possible to define unique text for the with method's javadoc. To do that, you create a 'section' named `WITH`. A section is a line in your javadoc containing 2 or more dashes, then the text 'WITH', followed by 2 or more dashes, and nothing else on the line. If you use sections, `@return` and `@param` stripping / copying for that section is no longer done (move the `@param` line into the section).

### With Lombok

```

import lombok.AccessLevel;
import lombok.NonNull;
import lombok.With;

public class WithExample {
  @With(AccessLevel.PROTECTED) @NonNull private final String name;
  @With private final int age;
  
  public WithExample(String name, int age) {
    if (name == null) throw new NullPointerException();
    this.name = name;
    this.age = age;
  }
}
```

### Vanilla Java

```

import lombok.NonNull;

public class WithExample {
  private @NonNull final String name;
  private final int age;

  public WithExample(String name, int age) {
    if (name == null) throw new NullPointerException();
    this.name = name;
    this.age = age;
  }

  protected WithExample withName(@NonNull String name) {
    if (name == null) throw new java.lang.NullPointerException("name");
    return this.name == name ? this : new WithExample(name, age);
  }

  public WithExample withAge(int age) {
    return this.age == age ? this : new WithExample(name, age);
  }
}
```

### Supported configuration keys:

- `lombok.with.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@With` as a warning or error if configured.

### Small print

With methods cannot be generated for static fields because that makes no sense.

With methods can be generated for abstract classes, but this generates an abstract method with the appropriate signature.

When applying `@With` to a type, static fields and fields whose name start with a $ are skipped.

For generating the method names, the first character of the field, if it is a lowercase character, is title-cased, otherwise, it is left unmodified. Then, `with` is prefixed.

No method is generated if any method already exists with the same name (case insensitive) and same parameter count. For example, `withX(int x)` will not be generated if there's already a method `withX(String... x)` even though it is technically possible to make the method. This caveat exists to prevent confusion. If the generation of a method is skipped for this reason, a warning is emitted instead. Varargs count as 0 to N parameters.

Various well known annotations about nullity cause null checks to be inserted and will be copied to the parameter. See [Getter/Setter](https://projectlombok.org/features/GetterSetter) documentation's small print for more information.

If you have configured a nullity annotation flavour via [`lombok.config`](https://projectlombok.org/features/configuration) key `lombok.addNullAnnotations`, the method or return type (as appropriate for the chosen flavour) is annotated with a non-null annotation.



# 十五、@Getter(lazy=true)

### Laziness is a virtue!

`@Getter(lazy=true)` was introduced in Lombok v0.10.

### Overview

You can let lombok generate a getter which will calculate a value once, the first time this getter is called, and cache it from then on. This can be useful if calculating the value takes a lot of CPU, or the value takes a lot of memory. To use this feature, create a `private final` variable, initialize it with the expression that's expensive to run, and annotate your field with `@Getter(lazy=true)`. The field will be hidden from the rest of your code, and the expression will be evaluated no more than once, when the getter is first called. There are no magic marker values (i.e. even if the result of your expensive calculation is `null`, the result is cached) and your expensive calculation need not be thread-safe, as lombok takes care of locking.

If the initialization expression is complex, or contains generics, we recommend moving the code to a private (if possible static) method, and call that instead.

### With Lombok

```

import lombok.Getter;

public class GetterLazyExample {
  @Getter(lazy=true) private final double[] cached = expensive();
  
  private double[] expensive() {
    double[] result = new double[1000000];
    for (int i = 0; i < result.length; i++) {
      result[i] = Math.asin(i);
    }
    return result;
  }
}
```

### Vanilla Java

```
public class GetterLazyExample {
  private final java.util.concurrent.AtomicReference<java.lang.Object> cached = new java.util.concurrent.AtomicReference<java.lang.Object>();
  
  public double[] getCached() {
    java.lang.Object value = this.cached.get();
    if (value == null) {
      synchronized(this.cached) {
        value = this.cached.get();
        if (value == null) {
          final double[] actualValue = expensive();
          value = actualValue == null ? this.cached : actualValue;
          this.cached.set(value);
        }
      }
    }
    return (double[])(value == this.cached ? null : value);
  }
  
  private double[] expensive() {
    double[] result = new double[1000000];
    for (int i = 0; i < result.length; i++) {
      result[i] = Math.asin(i);
    }
    return result;
  }
}
```

### Supported configuration keys:

- `lombok.getter.lazy.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@Getter(lazy=true)` as a warning or error if configured.

### Small print

You should never refer to the field directly, always use the getter generated by lombok, because the type of the field will be mangled into an `AtomicReference`. Do not try to directly access this `AtomicReference`; if it points to itself, the value has been calculated, and it is `null`. If the reference points to `null`, then the value has not been calculated. This behaviour may change in future versions. Therefore, *always* use the generated getter to access your field!

Other Lombok annotations such as `@ToString` always call the getter even if you use `doNotUseGetters=true`.



# 十六、@Log (and friends)

### Captain's Log, stardate 24435.7: "What was that line again?"

The various `@Log` variants were added in lombok v0.10. *NEW in lombok 0.10:* You can annotate any class with a log annotation to let lombok generate a logger field.
The logger is named `log` and the field's type depends on which logger you have selected.

*NEW in lombok v1.16.24:* Addition of google's FluentLogger (via [`@Flogger`](https://projectlombok.org/features/log#Flogger)).

*NEW in lombok v1.18.10:* Addition of [`@CustomLog`](https://projectlombok.org/features/log#CustomLog) which lets you add any logger by configuring how to create them with a config key.

### Overview

You put the variant of `@Log` on your class (whichever one applies to the logging system you use); you then have a static final `log` field, initialized as is the commonly prescribed way for the logging framework you use, which you can then use to write log statements.

There are several choices available:

- `@CommonsLog`

    Creates `private static final org.apache.commons.logging.Log log = org.apache.commons.logging.LogFactory.getLog(LogExample.class);`

- `@Flogger`

    Creates `private static final com.google.common.flogger.FluentLogger log = com.google.common.flogger.FluentLogger.forEnclosingClass();`

- `@JBossLog`

    Creates `private static final org.jboss.logging.Logger log = org.jboss.logging.Logger.getLogger(LogExample.class);`

- `@Log`

    Creates `private static final java.util.logging.Logger log = java.util.logging.Logger.getLogger(LogExample.class.getName());`

- `@Log4j`

    Creates `private static final org.apache.log4j.Logger log = org.apache.log4j.Logger.getLogger(LogExample.class);`

- `@Log4j2`

    Creates `private static final org.apache.logging.log4j.Logger log = org.apache.logging.log4j.LogManager.getLogger(LogExample.class);`

- `@Slf4j`

    Creates `private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);`

- `@XSlf4j`

    Creates `private static final org.slf4j.ext.XLogger log = org.slf4j.ext.XLoggerFactory.getXLogger(LogExample.class);`

- `@CustomLog`

    Creates `private static final *com.foo.your.Logger* log = *com.foo.your.LoggerFactory.createYourLogger*(LogExample.class);`

This option *requires* that you add a configuration to your [`lombok.config`](https://projectlombok.org/features/configuration) file to specify what `@CustomLog` should do.For example:`lombok.log.custom.declaration = com.foo.your.Logger com.foo.your.LoggerFactory.createYourLog(TYPE)(TOPIC)` which would produce the above statement. First comes a type which is the type of your logger, then a space, then the type of your logger factory, then a dot, then the name of the logger factory method, and then 1 or 2 parameter definitions; at most one definition with `TOPIC` and at most one without `TOPIC`. Each parameter definition is specified as a parenthesised comma-separated list of parameter kinds. The options are: `TYPE` (passes this `@Log` decorated type, as a class), `NAME` (passes this `@Log` decorated type's fully qualified name), `TOPIC` (passes the explicitly chosen topic string set on the `@CustomLog` annotation), and `NULL` (passes `null`).The logger type is optional; if it is omitted, the logger factory type is used. (So, if your logger class has a static method that creates loggers, you can shorten your logger definition).Please contact us if there is a public, open source, somewhat commonly used logging framework that we don't yet have an explicit annotation for. The primary purpose of `@CustomLog` is to support your in-house, private logging frameworks.

By default, the topic (or name) of the logger will be the (name of) the class annotated with the `@Log` annotation. This can be customised by specifying the `topic` parameter. For example: `@XSlf4j(topic="reporting")`.

### With Lombok

```

import lombok.extern.java.Log;
import lombok.extern.slf4j.Slf4j;

@Log
public class LogExample {
  
  public static void main(String... args) {
    log.severe("Something's wrong here");
  }
}

@Slf4j
public class LogExampleOther {
  
  public static void main(String... args) {
    log.error("Something else is wrong here");
  }
}

@CommonsLog(topic="CounterLog")
public class LogExampleCategory {

  public static void main(String... args) {
    log.error("Calling the 'CounterLog' with a message");
  }
}
```

### Vanilla Java

```

public class LogExample {
  private static final java.util.logging.Logger log = java.util.logging.Logger.getLogger(LogExample.class.getName());
  
  public static void main(String... args) {
    log.severe("Something's wrong here");
  }
}

public class LogExampleOther {
  private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExampleOther.class);
  
  public static void main(String... args) {
    log.error("Something else is wrong here");
  }
}

public class LogExampleCategory {
  private static final org.apache.commons.logging.Log log = org.apache.commons.logging.LogFactory.getLog("CounterLog");

  public static void main(String... args) {
    log.error("Calling the 'CounterLog' with a message");
  }
}
```

### Supported configuration keys:

- `lombok.log.fieldName` = *an identifier* (default: `log`).

    The generated logger fieldname is by default '`log`', but you can change it to a different name with this setting.

- `lombok.log.fieldIsStatic` = [`true` | `false`] (default: true)

    Normally the generated logger is a `static` field. By setting this key to `false`, the generated field will be an instance field instead.

- `lombok.log.custom.declaration` = *LoggerType* LoggerFactoryType.loggerFactoryMethod(loggerFactoryMethodParams)*(loggerFactoryMethodParams)*

    Configures what to generate when `@CustomLog` is used. (The italicized parts are optional). loggerFactoryMethodParams is a comma-separated list of zero to any number of parameter kinds to pass. Valid kinds: TYPE, NAME, TOPIC, and NULL. You can include a parameter definition for the case where no explicit topic is set (do not include the TOPIC in the parameter list), and for when an explicit topic is set (do include the TOPIC parameter in the list).

- `lombok.log.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of any of the various log annotations as a warning or error if configured.

- `lombok.log.custom.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@lombok.CustomLog` as a warning or error if configured.

- `lombok.log.apacheCommons.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@lombok.extern.apachecommons.CommonsLog` as a warning or error if configured.

- `lombok.log.flogger.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@lombok.extern.flogger.Flogger` as a warning or error if configured.

- `lombok.log.jbosslog.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@lombok.extern.jbosslog.JBossLog` as a warning or error if configured.

- `lombok.log.javaUtilLogging.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@lombok.extern.java.Log` as a warning or error if configured.

- `lombok.log.log4j.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@lombok.extern.log4j.Log4j` as a warning or error if configured.

- `lombok.log.log4j2.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@lombok.extern.log4j.Log4j2` as a warning or error if configured.

- `lombok.log.slf4j.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@lombok.extern.slf4j.Slf4j` as a warning or error if configured.

- `lombok.log.xslf4j.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of `@lombok.extern.slf4j.XSlf4j` as a warning or error if configured.

### Small print

If a field called `log` already exists, a warning will be emitted and no code will be generated.

A future feature of lombok's diverse log annotations is to find calls to the logger field and, if the chosen logging framework supports it and the log level can be compile-time determined from the log call, guard it with an `if` statement. This way if the log statement ends up being ignored, the potentially expensive calculation of the log string is avoided entirely. This does mean that you should *NOT* put any side-effects in the expression that you log.



# 十七、Lombok experimental features

The [Lombok javadoc](https://projectlombok.org/api/) is available, but we advise these pages.

Experimental features are available in your normal lombok installation, but are not as robustly supported as lombok's main features. In particular, experimental features:

- Are not tested as well as the core features.
- Do not get bugs fixed as quickly as core features.
- May have APIs that will change, possibly drastically if we find a different, better way to solve the same problem.
- May disappear entirely if the feature is too difficult to support or doesn't bust enough boilerplate.



Features that receive positive community feedback and which seem to produce clean, flexible code will eventually become accepted as a core feature and move out of the experimental package.

#### [`var`](https://projectlombok.org/features/experimental/var)

Modifiable local variables with a type inferred by assigning value.

#### [`@Accessors`](https://projectlombok.org/features/experimental/Accessors)

A more fluent API for getters and setters.

#### [`@ExtensionMethod`](https://projectlombok.org/features/experimental/ExtensionMethod)

Annoying API? Fix it yourself: Add new methods to existing types!

#### [`@FieldDefaults`](https://projectlombok.org/features/experimental/FieldDefaults)

New default field modifiers for the 21st century.

#### [`@Delegate`](https://projectlombok.org/features/experimental/Delegate)

Don't lose your composition.

#### [`onMethod= / onConstructor= / onParam=`](https://projectlombok.org/features/experimental/onX)

Sup dawg, we heard you like annotations, so we put annotations in your annotations so you can annotate while you're annotating.

#### [`@UtilityClass`](https://projectlombok.org/features/experimental/UtilityClass)

Utility, metility, wetility! Utility classes for the masses.

#### [`@Helper`](https://projectlombok.org/features/experimental/Helper)

With a little help from my friends... Helper methods for java.

#### [`@FieldNameConstants`](https://projectlombok.org/features/experimental/FieldNameConstants)

Name... that... field! String constants for your field's names.

#### [`@SuperBuilder`](https://projectlombok.org/features/experimental/SuperBuilder)

Bob now knows his ancestors: Builders with fields from superclasses, too.

#### [`@Tolerate`](https://projectlombok.org/features/experimental/Tolerate)

Skip, jump, and forget! Make lombok disregard an existing method or constructor.

#### [`@Jacksonized`](https://projectlombok.org/features/experimental/Jacksonized)

Bob, meet Jackson. Lets make sure you become fast friends.

#### [`@StandardException`](https://projectlombok.org/features/experimental/StandardException)

Standard.. Exceptional? This is not just an oxymoron, it's convenient!

### Supported configuration keys:

- `lombok.experimental.flagUsage` = [`warning` | `error`] (default: not set)

    Lombok will flag any usage of any of the features listed here as a warning or error if configured.

### Putting the "Ex" in "Experimental": promoted or deleted experimental features.

#### [`@Value: promoted`](https://projectlombok.org/features/Value)

`@Value` has proven its value and has been moved to the main package.

#### [`@Builder: promoted`](https://projectlombok.org/features/Builder)

`@Builder` is a solid base to build APIs on, and has been moved to the main package.

#### [`@Wither: renamed to @With, and promoted`](https://projectlombok.org/features/With)

Immutable 'setters' - methods that create a clone but with one changed field.