# 什么是 APT

APT： Annotation Process Tools。注解处理工具。



## APT 特点

1. 与 Javac 一样，apt 是拿来操作 Java 源文件的，而不是编译后的类

2. apt 会处理完源文件后编译它们

3. apt 处理源文件过程中如果产生新的源文件，再新一轮检查中新新的源文件的注解也会被处理，直到没有新的源文件产生之后，再编译所有源文件

4. apt 能将多个注解处理器组合在一起

5. 还能添加监听器，方便一轮处理结束时收到通知

   

# 发展

- APT 最早提及是在 JDK 1.5，注解也是这个时候支持的，APT 作为补充性 API 存在：**Mirror API**

- JDK 1.6 开始，写入编译器 javac 中，相关包：javax.annotation.processing

# 应用

非常出门的应用框架就是 **Butterknife** 黄油刀，使用注解绑定控件 id 和控件，省略了 **findViewById** 的绑定控件的重复代码的编写，提供效率

## 模仿 Butterknife 写个例子

虽然没有读过 ButterKnife 的源码，但是根据使用加上 APT 的知识应该可以大致模仿写个简单的工具：

#### 1. 模仿 Butterknife 的依赖创建两个 Java Lib

两个Java Library：只有 Java Library 可以使用 JDK 的 javax.annotation.processing

1. buildCompiler：注解解析器

2. buildLib：注解，工具类

#### 2. 把 Java Lib 添加到 app 模块的依赖上

```java
// build.gradle (app)
annotationProcessor project(':bindCompiler')
implementation project(':bindLib')
```

同时，把 bindLib 添加到 bindCompiler 的依赖上：

```
// build.gradle (bindCompiler)
implementation project(':bindLib')
```

#### 3. bindLib 添加注解，添加工具类

注解：作用到 View 的成员变量，并且生成代码后就不需要了，因此：

```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.SOURCE)
public @interface BindView {
    int value();
}
```

工具类：

- 所有 Activity 的总入口，每个 Activity 都调用此类，传入 Activity 实例，有了实例，就能执行 Activity#findViewById 的操作

- 根据之前 Butterknife 使用 apt 生成的代码，实际上是为每个 Activity 和 Fragment 都生成了一个工具类，例如 MainActivity_ViewBinding，来做 Activity#findViewById 的操作

- 因此，工具类里面要无差别调用所有使用 apt 生成的工具类代码。那如何无差别？让所有工具类继承统一接口，工具类调用接口的实例

```java

// bindLib
public class BindLib {
    public static void bind(Object activity) {
        String canonicalName = activity.getClass().getCanonicalName();
        Class<?> utilClass = null;
        try {
            utilClass = Class.forName(canonicalName + "_Binder");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        if (utilClass != null) {
            try {
                Object o = utilClass.newInstance();
                if (o instanceof IBinder) {
                    ((IBinder) o).bind(activity);
                }
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (InstantiationException e) {
                e.printStackTrace();
            }
        }
    }
}
```

这里，每个 Activity 的生成的工具类为 Activity 全称 + "_Binder"，例如 MainActvity_Binder，并且继承了 IBinder 接口：

```java

public interface IBinder {
    void bind(Object o);
}
```

#### 4.  bindCompiler 添加 annotationProcessor 生成工具代码

思路：

1. 获取所有注解的元素

2. 获取所有元素所在的 Activity 类元素

3. 基于 "Activity 类元素：带注解的 View 元素列表" 的键值对形式储存

4. 遍历 "Activity 类元素" 创建 "类名_Binder" 的工具类，要求实现 IBinder 接口，并在实现方法里面做 findViewById 操作

```java

public class BindViewProcessor extends AbstractProcessor {

    private static class ViewInfo {
        public ViewInfo(int id, String viewName) {
            this.id = id;
            this.viewName = viewName;
        }

        public int id;
        public String viewName;

        @Override
        public boolean equals(Object o) {
            return this.viewName.equals(((ViewInfo) o).viewName);
        }
    }

    private Map<TypeElement, Set<ViewInfo>> mMap = new HashMap<>();
    private Filer mFiler;
    private Elements mElementUtils;

    @Override
    public synchronized void init(ProcessingEnvironment processingEnvironment) {
        super.init(processingEnvironment);
        mElementUtils = processingEnvironment.getElementUtils();
        mFiler = processingEnvironment.getFiler();
    }

    @Override
    public Set<String> getSupportedAnnotationTypes() {
        Set<String> set = new HashSet<>();
        set.add(BindView.class.getCanonicalName());
        return set;
    }

    @Override
    public SourceVersion getSupportedSourceVersion() {
        return SourceVersion.latestSupported();
    }

    @Override
    public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment) {
        Set<? extends Element> elements = roundEnvironment.getElementsAnnotatedWith(BindView.class);
        for (Element element : elements) {
            if (element instanceof VariableElement) {
                Element enclosingElement = element.getEnclosingElement();
                if (enclosingElement instanceof TypeElement) {
                    Set<ViewInfo> viewInfoSet = mMap.get(enclosingElement);
                    if (viewInfoSet == null) {
                        viewInfoSet = new HashSet<>();
                    }
                    BindView annotation = element.getAnnotation(BindView.class);
                    String viewName = element.getSimpleName().toString();
                    viewInfoSet.add(new ViewInfo(annotation.value(), viewName));
                    mMap.put((TypeElement) enclosingElement, viewInfoSet);
                }
            }
        }

        Set<TypeElement> typeElementSet = mMap.keySet();
        for (TypeElement element : typeElementSet) {
            Set<ViewInfo> viewInfoSet = mMap.get(element);
            String code = generateUtilClassCode(element, viewInfoSet);
            String utilClassName = element.getSimpleName().toString() + "_Binder";
            try {
                JavaFileObject sourceFile = mFiler.createSourceFile(utilClassName, element);
                Writer writer = sourceFile.openWriter();
                writer.write(code);
                writer.flush();
                writer.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return true;
    }

    private String generateUtilClassCode(TypeElement element, Set<ViewInfo> viewInfoSet) {
        String className = element.getSimpleName().toString();
        PackageElement packageOf = mElementUtils.getPackageOf(element);
        String packageName = packageOf.getQualifiedName().toString();
        String utilClassName = className + "_Binder";

        StringBuilder builder = new StringBuilder();
        builder.append("package ").append(packageName).append(";").append("\n")
                .append("import com.john.bindlib.IBinder;").append("\n")
                .append("public class ").append(utilClassName).append(" implements IBinder ").append("{")
                .append("\n")
                .append("\t").append("@Override").append("\n")
                .append("\t").append("public void bind(Object o) {").append("\n")
                .append("\t\t").append(className).append(" act = (").append(className).append(") o;")
                .append("\n");
        for (ViewInfo viewInfo : viewInfoSet) {
            builder.append("\t\t").append("act.").append(viewInfo.viewName).append(" = ")
                    .append("act.findViewById(").append(viewInfo.id).append(");").append("\n");
        }
        builder.append("\t}").append("\n")
                .append("}");
        return builder.toString();
    }
}
```

其中 process() & generateUtilClassCode() 为核心代码

> 补充 Elements 对应关系：
>
> - ExecutableElement：成员方法
>
> - PackageElement：包
>
> - TypeElement：类，接口
>
> - TypeParameterElement：带泛型参数的类和接口
>
> - VariableElement：成员变量

#### 5. 注册 annotationProcessor

注册 annotationProcessor 是固定步骤，Google 提供了 AutoService 框架帮处理这个工作，但是由于简单固定，这里手工添加也是可以的：

buildCompiler 模块：新建路径和文件

```java

// 建路径和文件
src/main/resources/META-INF/services/javax.annotation.processing.Processor

// 内容为完整处理器名称
com.john.bindcompiler.BindViewProcessor
```

#### 6. 运行查看生成的源文件

新生成的源代码路径：app/build/generated/ap_generated_sources/debug/out

```java

package com.john.newtest;
import com.john.bindlib.IBinder;
public class MainActivity_Binder implements IBinder {
	@Override
	public void bind(Object o) {
		MainActivity act = (MainActivity) o;
		act.mTextView2 = act.findViewById(2131165354);
		act.mTextView = act.findViewById(2131165353);
	}
}

// Activity 源文件：
public class MainActivity extends AppCompatActivity {

    @BindView(R.id.textView)
    TextView mTextView;
    @BindView(R.id.textView2)
    TextView mTextView2;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        BindLib.bind(this);

        mTextView.setText("Hello Text");
        mTextView2.setText("Hello Text 2");
    }
}
```

# 调试

1. 打开 Android Studio 内置的 Terminal 输入：

```shell
./gradlew --no-daemon -Dorg.gradle.debug=true :app:clean :app:compileDebugJavaWithJavac
```

1. 创建远程调试配置：

- Edit Configurations..
-   点击 ‘+’ 创建 Remote，输入名字，确认下端口是不被占用的，一般默认 5005 就 ok
-  Apply + OK

2. 代码上设置好断点位置，点击 Debug 按钮，等待一会儿

# 补充资料

[JavaPoet: Java API for generating .java source file](#https://github.com/square/javapoet)
