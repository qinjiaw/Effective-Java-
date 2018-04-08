### 第17条：使可变性最小化

An immutable class is simply a class whose instances cannot be modified. All of the information contained in each instance is fixed for the lifetime of the object, so no changes can ever be observed. The Java platform libraries contain many immutable classes, includingString, the boxed primitive classes, and BigInteger and BigDecimal. There are many good reasons for this: Immutable classes are easier to design, implement, and use than mutable classes. They are less prone to error and are more secure. To make a class immutable, follow these five rules:

1. **Don’t provide methods that modify the object’s state**\(known as mutators\).
2. **Ensure that the class can’t be extended.**This prevents careless or malicious subclasses from compromising the immutable behavior of the class by behaving as if the object’s state has changed. Preventing subclassing is generally accomplished by making the class final, but there is an alternative that we’ll discuss later.

3. **Make all fields final. **This clearly expresses your intent in a manner that is enforced by the system. Also, it is necessary to ensure correct behavior if a reference to a newly created instance is passed from one thread to another without synchronization, as spelled out in the memory model\[JLS, 17.5; Goetz06, 16\].

4. **Make all fields private. **This prevents clients from obtaining access to mutable objects referred to by fields and modifying these objects directly. While it is technically permissible for immutable classes to have public final fields containing primitive values or references to immutable objects, it is not recommended because it precludes changing the internal representation in a later release \(Items 15and16\).

5. **Ensure exclusive access to any mutable components. **If your class has any fields that refer to mutable objects, ensure that clients of the class cannot obtain references to these objects. Never initialize such a field to a client-provided object reference or return the field from an accessor. Make defensive copies\(Item 50\) in constructors, accessors, and readObject methods \(Item 88\).

Many of the example classes in previous items are immutable. One such class is PhoneNumber in Item 11, which has accessors for each attribute but no corresponding mutators. Here is a slightly more complex example:

```
// Immutable complex number class 
public final class Complex {
    private final double re; 
    private final double im;
    public Complex(double re, double im) { 
        this.re = re;
        this.im = im;
    }
    public double realPart() { 
        return re; 
    } 
    public double imaginaryPart() { 
        return im; 
    }
    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }
    public Complex minus(Complex c) { 
        return new Complex(re - c.re, im - c.im);
    }
    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im, re * c.im + im * c.re); 
    }
    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp, (im * c.re - re * c.im) / tmp);

    }
    @Override 
    public boolean equals(Object o) { 
        if (o == this) return true;
        if (!(o instanceof Complex)) return false;
        Complex c = (Complex) o;
        // See page 47 to find out why we use compare instead of == 
        return Double.compare(c.re, re) == 0 && Double.compare(c.im, im) == 0; 
    }
    @Override 
    public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }
    @Override 
    public String toString() { 
        return "(" + re + " + " + im + "i)";
    } 
}
```

This class represents a complex number\(a number with both real and imaginary parts\). In addition to the standard Object methods, it provides accessors for the real and imaginary parts and provides the four basic arithmetic operations: addition, subtraction, multiplication, and division. Notice how the arithmetic operations create and return a new Complex instance rather than modifying this instance. This pattern is known as the functional approach because methods return the result of applying a function to their operand, without modifying it. Contrast it to the procedural or imperative approach in which methods apply a procedure to their operand, causing its state to change. Note that the method names are prepositions \(such as plus\) rather than verbs \(such as add\). This emphasizes the fact that methods don’t change the values of the objects. The BigInteger and BigDecimal classes did not obey this naming convention, and it led to many usage errors.  

