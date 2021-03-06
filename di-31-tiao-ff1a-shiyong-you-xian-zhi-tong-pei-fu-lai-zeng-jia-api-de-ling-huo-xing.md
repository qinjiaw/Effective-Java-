### 第31条：使用有限制通配符来增加API的灵活性

As noted in Item 28, parameterized types are invariant. In other words, for any two distinct types Type1 and Type2, List&lt;Type1&gt; is neither a subtype nor a supertype of List&lt;Type2&gt;. Although it is counterintuitive that List&lt;String&gt; is not a subtype of List&lt;Object&gt;, it really does make sense. You can put any object into a List&lt;Object&gt;, but you can put only strings into a List&lt;String&gt;. Since a List&lt;String&gt; can’t do everything a List&lt;Object&gt; can, it isn’t a subtype \(by the Liskov substitution principal, Item 10\).

就像条目28里提到的，参数化类型是受约束的。换句话说，对于任意两个不同的类型Type1和Type2，List&lt;Type1&gt;既不是List&lt;Type2&gt;的子类型也不是List&lt;Type2&gt;的父类型。List&lt;String&gt;不是List&lt;Object&gt;的子类型，虽然这看起来有点违反直觉，但这的确是有道理的。你可以将任意对象放入List&lt;Object&gt;，但你只能将字符串放入List&lt;String&gt;。由于List&lt;String&gt;并不具备List&lt;Object&gt;的每一项功能，所以它并不是List&lt;String&gt;并不是List&lt;Object&gt;的子类型（条目 10 中的里氏替代原则）。

Sometimes you need more flexibility than invariant typing can provide. Consider the Stack class from Item 29. To refresh your memory, here is its public API:

有时，不可变类型不足以为我们提供足够的灵活性。考虑条目29的Stack类。我们来回忆一下，下面是它的公共API：

```
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```

Suppose we want to add a method that takes a sequence of elements and pushes them all onto the stack. Here’s a first attempt:

假设我们需要添加一个方法，这个方法接受一系列元素并将这些元素推入栈顶。以下是第一个尝试：

```
// pushAll method without wildcard type - deficient!
public void pushAll(Iterable<E> src) {
    for (E e : src)
        push(e);
}
```

This method compiles cleanly, but it isn’t entirely satisfactory. If the element type of the Iterable src exactly matches that of the stack, it works fine. But suppose you have a Stack&lt;Number&gt; and you invoke push\(intVal\), where intVal is of type Integer. This works because Integer is a subtype of Number. So logically, it seems that this should work, too:

这个方法编译时不会有任何问题，但它并不完全满足我们的需求。如果src的元素类型与栈的元素类型相匹配，这个方法就能正确运行。但假设你有一个Stack&lt;Number&gt;并且调用了push\(intVal\)，intVal是Integer类型。这也应该能运行，因为Integer是Number的子类型。所以从逻辑上看，这么做也应该是可行的：

```
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ... ;
numberStack.pushAll(integers);
```

If you try it, however, you’ll get this error message because parameterized types are invariant:

然而，假如你真这么做，你将会得到以下的错误信息，因为参数化类型是不可变的：

```
StackTest.java:7: error: incompatible types: Iterable<Integer>
cannot be converted to Iterable<Number>
numberStack.pushAll(integers);
^
```

Luckily, there’s a way out. The language provides a special kind of parameterized type call a bounded wildcard type to deal with situations like this. The type of the input parameter to pushAll should not be “Iterable of E” but “Iterable of some subtype of E,” and there is a wildcard type that means precisely that: Iterable&lt;? extends E&gt;. \(The use of the keyword extends is slightly misleading: recall from Item 29 that subtype is defined so that every type is a subtype of itself, even though it does not extend itself.\) Let’s modify pushAll to use this type:

幸运的是，有个办法可以解决这个问题。Java语言提供了一种特殊的参数化类型来处理这类情况，它就是有限制通配符类型。pushAll方法的输入参数类型不应该是“E类型的Iterable”，而应该是“E的子类型的Iterable”，并且有一个通配符类型能恰当地表述这一点：Iterable&lt;? extends E&gt;。（关键字extends的使用会带点误导性：回忆一下条目29，子类型确定后，每个类型就是其自身的子类型，即便没有扩展其自身。）让我们修改下pushAll方法来用上这个类型：

```
// Wildcard type for a parameter that serves as an E producer
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
```

With this change, not only does Stack compile cleanly, but so does the client code that wouldn’t compile with the original pushAll declaration. Because Stack and its client compile cleanly, you know that everything is typesafe. Now suppose you want to write a popAll method to go with pushAll. The popAll method pops each element off the stack and adds the elements to the given collection. Here’s how a first attempt at writing the popAll method might look:

修改之后，不仅Stack类编译不会出现任何问题，而且使用了原始pushAll方法的客户端代码编译也同样不会出现任何问题。因为Stack类和它的客户端都能完美编译，所以你可以确定一切都是类型安全的了。现在假设你想给pushAll方法写一个对应的popAll方法。popAll方法将栈里的每一个元素依次弹出，并将弹出的元素添加到指定的集合里去。我们先来试着写popAll方法的第一个版本，如下所示：

```
// popAll method without wildcard type - deficient!
public void popAll(Collection<E> dst) { 
    while (!isEmpty())
        dst.add(pop()); 
}
```

Again, this compiles cleanly and works fine if the element type of the destination collection exactly matches that of the stack. But again, it isn’t entirely satisfactory. Suppose you have a Stack&lt;Number&gt; and variable of type Object. If you pop an element from the stack and store it in the variable, it compiles and runs without error. So shouldn’t you be able to do this, too?

同样，上面的方法编译也没有任何问题，而且在指定集合的元素类型与栈的元素类型完全一致的情况下，这个方法也能正确运行。但它也同样不能完全满足我们的要求。假设你有一个Stack&lt;Number&gt;和一个类型为Object的变量。如果你从栈里弹出一个元素并将它存储在变量里，它编译和运行都没有错误。那么为什么你不应该这么做？

```
Stack<Number> numberStack = new Stack<Number>(); 
Collection<Object> objects = ... ; 
numberStack.popAll(objects);
```

If you try to compile this client code against the version of popAll shown earlier, you’ll get an error very similar to the one that we got with our first version of pushAll: Collection&lt;Object&gt;is not a subtype of Collection&lt;Number&gt;. Once again, wildcard types provide a way out. The type of the input parameter to popAll should not be “collection of E” but “collection of some supertype of E” \(where supertype is defined such that E is a supertype of itself \[JLS, 4.10\]\). Again, there is a wildcard type that means precisely that: Collection&lt;? super E&gt;. Let’s modify popAll to use it:

如果你尝试编译基于前面所示的popAll方法写出的客户端代码，你将会得到一个类似于第一个版本的pushAll方法所遇到的错误：Collection&lt;Object&gt;不是Collection&lt;Number&gt;的子类型。同样，通配符类型解决了这个问题。popAll方法的输入参数类型不应该是“E的集合”，而应该是“E的父类型的集合”（其中父类型被定义为E是其自身的父类型\[JLS, 4.10\]）。同样地，有一个通配符类型可以恰当地表示这一点：Collection&lt;? super E&gt;。下面我们修改popAll方法来用上这个类型：

```
// Wildcard type for parameter that serves as an E consumer
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop()); 
}
```

With this change, both Stack and the client code compile cleanly.

这么修改之后，Stack类和客户端代码都能编译通过，而且不会出现错误和警告。

The lesson is clear. **For maximum flexibility, use wildcard types on input parameters that represent producers or consumers.** If an input parameter is both a producer and a consumer, then wildcard types will do you no good: you need an exact type match, which is what you get without any wildcards. Here is a mnemonic to help you remember which wildcard type to use:

**结论很明显。为了最大的灵活性，请在那些表示生产者或者消费者的输入参数上使用通配符类型。**如果一个输入参数同时是生产者和消费者，那么通配符类型不会带来什么好处：你需要一个精确的类型匹配，而不能去用通配符。以下是一个助记符，它帮助你记住应该使用哪种通配符类型：

**PECS stands for producer-extends, consumer-super.** In other words, if a parameterized type represents a T producer, use&lt;? extends T&gt;; if it represents a T consumer, use&lt;? super T&gt;. In our Stack example, pushAll’s src parameter produces E instances for use by the Stack, so the appropriate type for src is Iterable&lt;? extends E&gt;; popAll’s dst parameter consumes E instances from the Stack, so the appropriate type for dst is Collection&lt;? super E&gt;. The PECS mnemonic captures the fundamental principle that guides the use of wild-card types. Naftalin and Wadler call it the Get and Put Principle\[Naftalin07, 2.4\].

**PECS代表for producer-extends, consumer-super。**换句话说，如果一个参数化类型代表一个T类型的生产者，则使用&lt;? extends T&gt;；如果一个参数化类型代表一个T类型的消费者，则使用&lt;? super T&gt;。在我们的Stack类的例子当中，pushAll方法的src参数为了Stack类生产E类型的实例，所以适合于src的类型是Iterable&lt;? extends E&gt;；popAll的dst参数从Stack类里消费E类型的实例，所以适合于dst的类型是Collection&lt;? super E&gt;。PECS助记符体现了使用通配符类型的基本原则。Naftalin和Wadler把它称为Get和Put原则\[Naftalin07, 2.4\]。

With this mnemonic in mind, let’s take a look at some method and constructor declarations from previous items in this chapter.  
The Chooser constructor in Item 28 has this declaration:

记住这个助记符后，我们一起来看看本章前面条目里的一些方法声明和构造器声明。条目28里的Chooser类的构造器是这么声明的：

```
public Chooser(Collection<T> choices)
```

This constructor uses the collection choices only to produce values of type T\(and stores them for later use\), so its declaration should use a wildcard type that extends T. Here’s the resulting constructor declaration:

这个构造器使用集合choices仅仅是为了生产T类型的值（并将它们存储起来以备后用），所以它的声明应该用一个扩展自T的通配符类型。以下是新的构造器声明：

```
// Wildcard type for parameter that serves as an T producer
public Chooser(Collection<? extends T> choices)
```

And would this change make any difference in practice? Yes, it would. Suppose you have a List&lt;Integer&gt;, and you want to pass it in to the constructor for a Chooser&lt;Number&gt;. This would not compile with the original declaration, but it does once you add the bounded wildcard type to the declaration.

这么修改之后在实践当中会造成什么不同的？答案是会的。假设你有一个List&lt;Integer&gt;，而且你想将它传给Chooser&lt;Number&gt;的构造方法。用原来的声明的话是不会编译通过的，但一旦你将有限制通配符类型加到声明上，它就可以编译通过了。

Now let’s look at the union method from Item 30. Here is the declaration:

现在我们来看看条目30的union方法。下面是它的声明：

```
public static <E> Set<E> union(Set<E> s1, Set<E> s2)
```

Both parameters, s1and s2, are E producers, so the PECS mnemonic tells us that the declaration should be as follows:

参数s1和参数s2都是E类型的生产者，所以根据PECS助记符，上述声明应该修改成以下的样子：

```
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```

Note that the return type is still Set&lt;E&gt;. **Do not use bounded wildcard types as return types. **Rather than providing additional flexibility for your users, it would force them to use wildcard types in client code. With the revised declaration, this code will compile cleanly:

注意，返回类型仍然是Set&lt;E&gt;。**返回类型不要用有限制通配符类型。**因为这将强制客户端用通配符类型，而不是给它们提供额外的灵活性。使用修改后的声明，代码可以完美编译通过：

```
Set<Integer> integers = Set.of(1, 3, 5);
Set<Double> doubles = Set.of(2.0, 4.0, 6.0); 
Set<Number> numbers = union(integers, doubles);
```

Properly used, wildcard types are nearly invisible to the users of a class. They cause methods to accept the parameters they should accept and reject those they should reject. **If the user of a class has to think about wildcard types, there is probably something wrong with its API.**

如果使用得当，通配符类型对类的使用者几乎是不可见的。这些通配符类型使得方法接受它们应该接受的参数，拒绝应该拒绝的参数。**如果类的用户必须对通配符类型做出思考，很可能就是API哪里有问题了。**

Prior to Java 8, the type inference rules were not clever enough to handle the previous code fragment, which requires the compiler to use the contextually specified return type \(or target type\) to infer the type of E. The target type of the union invocation shown earlier is Set&lt;Number&gt;. If you try to compile the fragment in an earlier version of Java \(with an appropriate replacement for the Set.of factory\), you’ll get a long, convoluted error message like this:

在Java 8之前，类引用规则还不够智能，不足以处理前面的代码片段，其要求编译器根据上下文指定的返回类型（或目标类型）来推断E的类型。前面展示的union调用的目标类型是Set&lt;Number&gt;。如果你用早期的Java版本（以及合适的Set.of工厂方法的替代版本）来编译那段代码，你将会得到一条又长又复杂的错误信息，就像下面这样：

```
Union.java:14: error: incompatible types
Set<Number> numbers = union(integers, doubles);
            ^ 
required: Set<Number>
found: Set<INT#1>
where INT#1,INT#2 are intersection types:
INT#1 extends Number,Comparable<? extends INT#2> INT#2 extends Number,Comparable<?>
```

Luckily there is a way to deal with this sort of error. If the compiler doesn’t infer the correct type, you can always tell it what type to use with an explicit type argument\[JLS, 15.12\]. Even prior to the introduction of target typing in Java 8, this isn’t something that you had to do often, which is good because explicit type arguments aren’t very pretty. With the addition of an explicit type argument, as shown here, the code fragment compiles cleanly in versions prior to Java 8:

幸运的是，有种办法可以处理这种问题。如果编译器无法推断出正确的类型，你可以用一个显式的类型参数来告诉它要用什么类型\[JLS, 15.12\]。即使在Java 8引入目标类型之前，你也不用总是这么做，因为显式类型参数不是很优雅。如下所示，在加入显式类型参数后，在Java 8之前的版本可以完美编译：

```
// Explicit type parameter - required prior to Java 8 
Set<Number> numbers = Union.<Number>union(integers, doubles);
```

Next let’s turn our attention to the max method in Item 30. Here is the original declaration:

接下来我们来看看条目30里的max方法。以下是原来的声明：

```
public static <T extends Comparable<T>> T max(List<T> list)
```

Here is a revised declaration that uses wildcard types:

下面是用通配符类型进行修改后的声明：

```
public static <T extends Comparable<? super T>> T max( List<? extends T> list)
```

To get the revised declaration from the original, we applied the PECS heuristic twice. The straightforward application is to the parameter list. It produces T instances, so we change the type from List&lt;T&gt; to List&lt;? extends T&gt;. The tricky application is to the type parameter T. This is the first time we’ve seen a wildcard applied to a type parameter. Originally,T was specified to extend Comparable&lt;T&gt;, but a comparable of T consumes T instances \(and produces integers indicating order relations\). Therefore, the parameterized type Comparable&lt;T&gt; is replaced by the bounded wildcard type Comparable&lt;? super T&gt;. Comparables are always consumers, so you should generally **use Comparable&lt;? super T&gt; in preference to Comparable&lt;T&gt;**.The same is true of comparators; therefore, you should generally **use Comparator&lt;? super T&gt; in preference to Comparator&lt;T&gt;**.

为了对原来的声明进行修改，我们应用了PECS转换两次。参数列表就直接应用了这个转换规则。它生成T类型的实例，所以我们可以将类型从List&lt;T&gt;转换成List&lt;? extends T&gt;。比较棘手的是对类型参数T的转换。这是我们第一次遇到将通配符应用到类型参数上。本来T是指定来扩展Comparable&lt;T&gt;的，但T类型的Comparable接口消费了T类型的实例（同时生成表明顺序关系的整数）。一次，参数类型Comparable&lt;T&gt;可以用有限制通配符类型Comparable&lt;? super T&gt;来替换。Comparable接口通常都是消费者，所以在一般情况下，你应该**优先用Comparable&lt;? super T&gt;，而不是Comparable&lt;T&gt;。**对于Comparator接口也应该如此，也就是应该**优先使用Comparator&lt;? super T&gt;，而不是Comparator&lt;T&gt;。**

The revised max declaration is probably the most complex method declaration in this book. Does the added complexity really buy you anything? Again, it does. Here is a simple example of a list that would be excluded by the original declaration but is permitted by the revised one:

修改后的max方法声明可能是本书最复杂的方法声明。增加的复杂度是否真的带来了好处呢？答案依旧是肯定的。以下是列表的一个简单例子，在原始的版本中不允许的，在修改后的版本则可以：

```
List<ScheduledFuture<?>> scheduledFutures = ... ;
```

The reason that you can’t apply the original method declaration to this list is that Scheduled Future does not implement Comparable&lt;ScheduledFuture&gt;. Instead, it is a sub interface ofDelayed, which extendsComparable&lt;Delayed&gt;. In other words, a Scheduled Future instance isn’t merely comparable to other Scheduled Future instances; it is comparable to any Delayed instance, and that’s enough to cause the original declaration to reject it. More generally, the wildcard is required to support types that do not implement Comparable\(or Comparator\) directly but extend a type that does.



There is one more wildcard-related topic that bears discussing. There is a duality between type parameters and wildcards, and many methods can be declared using one or the other. For example, here are two possible declarations for a static method to swap two indexed items in a list. The first uses an unbounded type parameter \(Item 30\) and the second an unbounded wildcard:

```
// Two possible declarations for the swap method 
public static <E> void swap(List<E> list, int i, int j); 
public static void swap(List<?> list, int i, int j);
```

Which of these two declarations is preferable, and why? In a public API, the second is better because it’s simpler. You pass in a list—any list—and the method swaps the indexed elements. There is no type parameter to worry about. As a rule, **if a type parameter appears only once in a method declaration, replace it with a wildcard. **If it’s an unbounded type parameter, replace it with an unbounded wildcard; if it’s a bounded type parameter, replace it with a bounded wildcard.

There’s one problem with the second declaration for swap. The straightforward implementation won’t compile:

```
public static void swap(List<?> list, int i, int j) { 
    list.set(i, list.set(j, list.get(i)));
}
```

Trying to compile it produces this less-than-helpful error message:

```
Swap.java:5: error: incompatible types: Object cannot be converted to CAP#1
list.set(i, list.set(j, list.get(i)));
                        ^
where CAP#1 is a fresh type-variable: CAP#1 extends Object from capture of ?
```

It doesn’t seem right that we can’t put an element back into the list that we just took it out of. The problem is that the type of list is List&lt;?&gt;, and you can’t put any value except null into a List&lt;?&gt;. Fortunately, there is a way to implement this method without resorting to an unsafe cast or a raw type. The idea is to write a private helper method to capture the wildcard type. The helper method must be a generic method in order to capture the type. Here’s how it looks:

```
public static void swap(List<?> list, int i, int j) { 
    swapHelper(list, i, j);
}
// Private helper method for wildcard capture
private static <E> void swapHelper(List<E> list, int i, int j) { 
    list.set(i, list.set(j, list.get(i)));
}
```

The swap Helper method knows that list is aList&lt;E&gt;. Therefore, it knows that any value it gets out of this list is of type E and that it’s safe to put any value of type E into the list. This slightly convoluted implementation of swap compiles cleanly. It allows us to export the nice wildcard-based declaration, while taking advantage of the more complex generic method internally. Clients of the swap method don’t have to confront the more complex swap Helper declaration, but they do benefit from it. It is worth noting that the helper method has precisely the signature that we dismissed as too complex for the public method.

In summary, using wildcard types in your APIs, while tricky, makes the APIs far more flexible. If you write a library that will be widely used, the proper use of wildcard types should be considered mandatory. Remember the basic rule: producer-extends, consumer-super\(PECS\). Also remember that all comparables and comparators are consumers.

