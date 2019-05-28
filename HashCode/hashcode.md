# HashCode

## Object类的hashCode()方法

### 是一个native方法

```java
public class Object {
    /*Returns a hash code value for the object.*/
    public native int hashCode();
}
```

This method is supported for the benefit of hash tables such as those provided by `java.util.HashMap`.

### 遵守的契约

The general contract<sub>【注：契约，期望人们遵守，但一定会有人不遵守】</sub> of `hashCode` is:

- Whenever it is invoked on the same object<sub>【注：同一个对象】</sub> more than once<sub>【注：执行多次】</sub> during an execution of a Java application<sub>【注：同一次执行过程中】</sub>, the `hashCode` method must consistently return the same integer<sub>【注：返回相同的值】</sub>, provided<sub>【注：前提条件】</sub> no information used in equals comparisons on the object is modified. This integer need not remain consistent<sub>【注：这里并不保证一致】</sub> from one execution of an application<sub>【注：一次执行】</sub> to another execution of the same application<sub>【注：另一次执行】</sub>.
- If two objects are equal according to the `equals(Object)` method<sub>【注：两个对象equal】</sub>, then calling the `hashCode` method on each of the two objects must produce the same integer result<sub>【注：两个对象的hashCode也要相等】</sub>.
- It is not required<sub>【注：并不要求这样】</sub> that if two objects are unequal according to the `equals(Object)` method<sub>【注：两个对象不equal】</sub>, then calling the `hashCode` method on each of the two objects must produce distinct integer results<sub>【注：产生不同的hashCode】</sub>. However, the programmer<sub>【注：对于开发人员而言】</sub> should be aware that producing distinct integer<sub>【注：产生不同的值】</sub> results for unequal objects<sub>【注：对于不equal的对象】</sub> may improve the performance of hash tables<sub>【注：会提高效率】</sub>.

契约规则总结：

- （1）在同一次执行中，同一个对象<sub>【注：注意，这是同一个对象】</sub>调用hashCode()多次，返回的hash值是一样的。
- （2）在同一次执行中，两个对象<sub>【注：注意，这是两个对象】</sub>，如果相等（equals），那么它们的hashCode也要相等<sub>【注：这里是equals对hashCode“施加压力”】</sub>。
- （3）在同一次执行中，两个对象<sub>【注：注意，这是两个对象】</sub>，如果不相等（not equals），那么它们的hashCode也可能相等。换句话说，hashCode相等的两个对象，这两个对象并不一定equals<sub>【注：这说明hashCode不能对equals“施加压力”】</sub>。

As much as is reasonably practical, the `hashCode` method defined by class `Object` does return distinct integers for distinct objects<sub>【注：Object类做到了这一点，即不同的object产生不同的hashCode值】</sub>. (This is typically implemented by converting the internal address of the object into an integer<sub>【注：实现的方式是将object的internal address转换成integer】</sub>, but this implementation technique is not required by the Java™ programming language.)

## String类的hashCode方法

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    /*Returns a hash code for this string.*/
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
}
```

The hash code for a `String` object is computed as

```java
s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
```

- [ ] [Why does Java's hashCode() in String use 31 as a multiplier?](https://stackoverflow.com/questions/299304/why-does-javas-hashcode-in-string-use-31-as-a-multiplier)

- The value `31` was chosen because it is an odd<sub>【注：奇数】</sub> prime<sub>【注：质数】</sub>. The advantage of using a prime is less clear, but it is traditional.
- A nice property of `31` is that the multiplication<sub>【注：乘法】</sub> can be replaced by a shift<sub>【注：位移】</sub> and a subtraction<sub>【注：减法】</sub> for better performance: `31 * i == (i << 5) - i`. Modern VMs do this sort of optimization automatically.

As Goodrich and Tamassia point out, If you take over `50,000` English words, using the constants `31`, `33`, `37`, `39`, and `41` will produce less than `7` collisions in each case. Knowing this, it should come as no surprise that many Java implementations choose one of these constants.

Actually, 37 would work pretty well! z := 37 * x can be computed as y := x + 8 * x; z := x + 4 * y. Both steps correspond to one LEA x86 instructions, so this is extremely fast.

In fact, multiplication with the even-larger prime 73 could be done at the same speed by setting y := x + 8 * x; z := x + 8 * y.

Using 73 or 37 (instead of 31) might be better, because it leads to denser code: The two LEA instructions only take 6 bytes vs. the 7 bytes for move+shift+subtract for the multiplication by 31. One possible caveat is that the 3-argument LEA instructions used here became slower on Intel's Sandy bridge architecture, with an increased latency of 3 cycles.

Moreover, `73` is Sheldon Cooper's favorite number.

### First: a not-so-good hash function

Recall that the Java String function combines successive characters by multiplying the current hash by `31` and then adding on the new character. To show why this works reasonably well for typical strings, let's start by modifying it so that we instead use the value `32` as the hash code. In other words, let's imagine the hash function is this:

```java
int hash = 0;
for (int i = 0; i < length(); i++) {
    hash = 32 * hash + charAt(i);
}
return hash;
```

### Improving the multiplier

Now we plot the same graph using 31 as the multiplier, as in the Java implementation of the String hash code function. Multiplying by 31 effectively means that we are shifting the hash by 5 places but then subtracting the original unshifted value:

```java
int hash = 0;
for (int i = 0; i < length(); i++) {
    hash = (hash << 5) - hash + charAt(i);
}
return hash;
```

