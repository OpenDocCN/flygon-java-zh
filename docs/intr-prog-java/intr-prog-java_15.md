# Managing Objects, Strings, Time, and Random Numbers

The classes that we will be discussing in this chapter, belong—together with Java collections and arrays discussed in the previous chapters—to the group of classes (mostly utilities from the Java Standard Library and Apache Commons) that every programmer has to master in order to become an effective coder. They also illustrate various software designs and solutions that are instructive and can be used as patterns for best coding practices.

We will cover the following areas of functionality:

*   Managing objects
*   Managing strings
*   Managing time
*   Manage random numbers

The list of the overviewed classes includes:

*   `java.util.Objects`
*   `org.apache.commons.lang3.ObjectUtils`
*   `java.lang.String`
*   `org.apache.commons.lang3.StringUtils`
*   `java.time.LocalDate`
*   `java.time.LocalTime`
*   `java.time.LocalDateTime`
*   `java.lang.Math`
*   `java.util.Random`

# Managing objects

You may not need to manage arrays and may even not need to manage collections (for some time, at least), but you cannot avoid managing objects, which means that the classes described in this section you are probably going to use every day.

Although the `java.util.Objects` class was added to the Java Standard Libraries in 2011 (with the Java 7 release), while the `ObjectUtils` class has existed in the Apache Commons libraries since 2002, their use grew slowly. This may be partially explained by the small number of methods they had originally—only six in `ObjectUtils` in 2003 and only nine in `Objects` in 2011\. However, they were very helpful methods that could make the code more readable and robust—less prone to errors. So, why these classes were not used more often from the very beginning remains a mystery. We hope that you start using them immediately with your very first project.

# Class java.util.Objects

The class `Objects` has only 17 methods—all static. We have already used some of them in the previous chapter when we implemented the class `Person`:

[PRE0]

We used the class `Objects` in the methods `equals()` and `hashCode()` previously. Everything worked fine. But, notice how we check the parameter `name` in the preceding constructor. If the parameter is `null`, we assign to the field `name` an empty `String` value. We did it to avoid `NullPointerException` in line 25\. Another way to do it is to use the class `ObjectUtils` from the Apache Commons library. We will demonstrate it in the next section. Methods of the class `ObjectUtils` handle `null` values and make the conversion of a `null` parameter to an empty `String` unnecessary.

But first, let's review the methods of the class `Objects`.

# equals() and deepEquals() 

We talked about the `equals()` method implementation extensively, but always assumed that it was invoked on a non-`null` object, `obj`, so the call `obj.equals(anotherObject)` could not generate `NullPointerException`.

Yet, sometimes we need to compare two objects, `a` and `b`, when one or both of them can be `null`. Here is typical code for such a case:

[PRE1]

This is the actual source code of the `boolean Objects.equals(Object a, Object b)` method. It allows comparing two objects using the method `equals(Object)` and handles cases where one or both of them are `null`.

Another related method of the class `Objects` is `boolean deepEquals(Object a, Object b)`. Here is its source code:

[PRE2]

As you can see, it is based on `Arrays.deepEquals()`, which we discussed in the previous section. The demonstration code for these methods helps to understand the difference:

[PRE3]

In the preceding code, `Objects.equals(as1, as2)` and `Objects.equals(aas1, aas2)` return `false` because arrays cannot override the method `equals()` of the class `Object` and are compared by references, not by value.

The method `Arrays.equals(aas1, aas2)` returns `false` for the same reason: because the elements of the nested array are arrays and are compared by references. 

To summarize, if you would like to compare two objects, `a` and `b`, by the values of their fields, then:

*   If they are not arrays and `a` is not `null`, use `a.equals(b)`
*   If they are not arrays and both objects can be `null`, use `Objects.equals(a, b)`
*   If both can be arrays and both can be `null`, use `Objects.deepEquals(a, b)`

That said, we can see that the method `Objects.deepEquals()` is the safest one, but it does not mean you must always use it. Most of the time, you will know whether the compared objects can be `null` or can be arrays, so you can safely use other `equals()` methods too.

# hash() and hashCode()

The hash values returned by the methods `hash()` or `hashCode()` are typically used as a key for storing the object in a hash-using collection, such as `HashSet()`. The default implementation in the  `Object` superclass is based on the object reference in memory.  It returns different hash values for two objects of the same class with the same values of the instance fields. That is why, if you need two class instances to have the same hash value for the same state, it is important to override the default `hashCode()` implementation using one of these methods:

*   `int hashCode(Object value)`: calculates a hash value for a single object
*   `int hash(Object... values)`: calculates a hash value for an array of objects (see how we used it in the class `Person` in our previous example)

Please notice that these two methods return different hash values for the same object when it is used as a single-element input array of the method `Objects.hash()`:

[PRE4]

The only value that yields the same hash from both methods is `null`:

[PRE5]

When used as a single not-null parameter, the same value has different hash values returned from the methods `Objects.hashCode(Object value)` and `Objects.hash(Object... values)`. The value `null` yields the same hash value, `0`, returned from each of these methods.

Another advantage of using the class `Objects` for hash value calculation is that it tolerates `null` values, while the attempt to call the instance method `hashCode()` on the `null` reference generates `NullPointerException`.

# isNull() and nonNull()

These two methods are just thin wrappers around Boolean expressions, `obj == null` and  `obj != null`:

*   `boolean isNull(Object obj)`: returns the same value as `obj == null`
*   `boolean nonNull(Object obj)`: returns the same value as `obj != null`

And here is the demo code:

[PRE6]

# requireNonNull() 

The following methods of the class `Objects` check the value of the first parameter and, if the value is `null`, either throw `NullPointerException` or return the provided default value:

*   `T requireNonNull(T obj)`: Throws `NullPointerException` without a message if the parameter is `null`:

[PRE7]

*   `T requireNonNull(T obj, String message)`: Throws `NullPointerException` with the message provided if the first parameter is `null`:

[PRE8]

*   `T requireNonNull(T obj, Supplier<String> messageSupplier)`: returns the message generated by the provided function if the first parameter is `null` or, if the generated message or the function itself is `null`, throws `NullPointerException`:

[PRE9]

*   `T requireNonNullElse(T obj, T defaultObj)`: returns the first parameter value if it is non-null, or the second parameter value if it is non-null, or throws `NullPointerException` with the message `defaultObj`:

[PRE10]

*   `T requireNonNullElseGet(T obj, Supplier<? extends T> supplier)`: returns the first parameter value if it is non-null, or the object produced by the provided function if it is non-null, or throws `NullPointerException` with the message `defaultObj`:

[PRE11]

# checkIndex()

The following group of methods checks whether the index and the length of a collection or an array are compatible:

*   `int checkIndex(int index, int length)`: throws `IndexOutOfBoundsException` if the provided `index` is bigger than `length - 1`
*   `int checkFromIndexSize(int fromIndex, int size, int length)`: throws `IndexOutOfBoundsException` if the provided `index + size` is bigger than `length - 1`
*   `int checkFromToIndex(int fromIndex, int toIndex, int length)`: throws `IndexOutOfBoundsException` if the provided `fromIndex` is bigger than `toIndex`, or `toIndex` is bigger than `length - 1`

Here is the demo code:

[PRE12]

# compare()

The method `int compare(T a, T b, Comparator<T> c)` of the class `Objects` uses the provided comparator's method `compare(T o1, T o2)` for comparing the two objects. We have described already the behavior of the `compare(T o1, T o2)` method while talking about sorting collections, so the following results should be expected:

[PRE13]

As we have mentioned already, the method `compare(T o1, T o2)` returns the difference of positions of objects `o1` and `o2` in a sorted list for `String` objects and just `-1`, `0`, or `1` for `Integer` objects. The API describes it as returning `0` when objects are equal and a negative number when the first object is smaller than the second; otherwise, it returns a positive number.

To demonstrate how the method `compare(T a, T b, Comparator<T> c)` works, let's assume that we want to sort objects of the class `Person` so that the name and age are arranged in a natural order of `String` and `Integer` classes, respectively:

[PRE14]

And here is the result of this new implementation of the `compareTo(Object)` method of the class `Person`: 

[PRE15]

As you can see, the `Person` objects are ordered by name in their natural order first, then by age in their natural order too. If we need to reverse the order of names, for example, we change the `compareTo(Object)` method to the following:

[PRE16]

The results looks as like we expected:

[PRE17]

The weakness of the method `compare(T a, T b, Comparator<T> c)` is that it does not handle `null` values. Adding the `new Person(25, null)` object to the list triggers `NullPointerException` during sorting. In such cases, it is better to use the `org.apache.commons.lang3.ObjectUtils.compare(T o1, T o2)` method, which we are going to demonstrate in the next section.

# toString()

There are cases when you need to convert an `object` (which is a reference to some class type) to its `String` representation. When the reference `obj` is assigned a `null` value (the object is not created yet), writing `obj.toString()`  generates `NullPointerException`. For such cases, using the following methods of the class `Objects` is a better choice:

*   `String toString(Object o)`: returns the result of calling `toString()` on the first parameter when it is not `null` and `null` when the first parameter value is `null`
*   `String toString(Object o, String nullDefault)`: returns the result of calling `toString()` on the first parameter when it is not `null` and the second parameter value `nullDefault` when the first parameter value is `null`

Here is the code that demonstrates how to use these methods:

[PRE18]

By the way, unrelated to the current discussion, please notice how we used the method `print()` instead of `println()` to show all the results in one line, because the method `print()` does not add an end-of-line symbol.

# Class lang3.ObjectUtils

The class `org.apache.commons.lang3.ObjectUtils` of the Apache Commons library complements the methods of the class `java.util.Objects` described previously. The scope of this book and the allocated size does not allow for a detailed review of all the methods of the class `ObjectUtils`, so we will describe them briefly, grouped by related functionality, and will demonstrate only those that are aligned with the examples we have provided already.

All the methods of the class `ObjectUtils` can be organized into seven groups:

*   Object cloning methods:
    *   `T clone(T obj)`: returns a copy of the provided object if it implements the interface `Cloneable`; otherwise, returns `null`.
    *   `T cloneIfPossible(T obj)`: returns a copy of the provided object if it implements the interface `Cloneable`; otherwise, returns the original provided object.
*   Methods that support object comparison:
    *   `int compare(T c1, T c2)`: compares newly ordered positions of the two objects that implement the interface `Comparable`; allows any or both parameters to be `null`; places a `null` value in front of a non-null value.
    *   `int compare(T c1, T c2, boolean nullGreater)`: behaves exactly as the previous method if the value of parameter `nullGreater` is `false`; otherwise, places a `null` value behind a non-null value. We can demonstrate the last two methods by using them in our class `Person`:

[PRE19]

The result of this change allows us to use a `null` value for the `name` field:

[PRE20]

Since we have used the method `Objects.compare(T c1, T c2)`, the `null` value was placed in front of non-null values. By the way, have you noticed that we do not display `null` anymore? That is because we have changed the method `toString()` of the class `Person` as follows:

[PRE21]

Instead of just displaying the value of the field `name`, we used the method `Objects.toString(Object o, String nullDefault)`, which substitutes the object with the provided `nullDefault` value when the object is `null`. As to whether to use this method, in this case, is a matter of style. Many programmers would probably argue that we must display the actual value without substituting it for something else. But, we have done it just to show how the method `Objects.toString(Object o, String nullDefault)` could be used.

If we now use the second `compare(T c1, T c2, boolean nullGreater)` method, the `compareTo()` method of the class `Person` will look as follows:

[PRE22]

Then, `Person` objects with their `name` set to `null` will be shown at the end of the sorted list:

[PRE23]

And, to complete the discussion of `null` values, the preceding code will break with `NullPointerException` when a `null` object is added to the list: `list.add(null)`. To avoid the exception, you can use a special `Comparator` object that handles the `null` elements of a list:

[PRE24]

In this code, you can see how we have indicated the desire to see the `null` objects at the end of the list. Instead, we could use another `Comparator` that places null objects at the beginning of the sorted list:

[PRE25]

*   `notEqual`:
    *   `boolean notEqual(Object object1, Object object2)`: compares two objects for inequality, where either one or both objects may be `null`
*   `identityToString`:
    *   `String identityToString(Object object)`: returns the `String` representation of the provided object as if produced by the default method `toString()` of the base class `Object`
    *   `void identityToString(StringBuffer buffer, Object object)`: appends the `String` representation of the provided object as if produced by the default method `toString()` of the base class `Object`
    *   `void identityToString(StringBuilder builder, Object object)`: appends the `String` representation of the provided object as if produced by the default method `toString()` of the base class `Object`
    *   `void identityToString(Appendable appendable, Object object)`: appends the `String` representation of the provided object as if produced by the default method `toString()` of the base class `Object`

The following code demonstrates two of these methods:

[PRE26]

*   `allNotNull` and `anyNotNull`:

    *   `boolean allNotNull(Object... values)`: returns `true` when all values in the provided array are not `null`
    *   `boolean anyNotNull(Object... values)`: returns `true` when at least one value in the provided array is not `null`
*   `firstNonNull` and `defaultIfNull`:

    *   `T firstNonNull(T... values)`: returns the first value from the provided array that is not `null`
    *   `T defaultIfNull(T object, T defaultValue)`: returns the provided default value if the first parameter is `null`
*   `max`, `min`, `median`, and `mode`:
    *   `T max(T... values)`: returns the last in the ordered list of provided values that implement the `Comparable` interface; returns `null` only when all values are `null`
    *   `T min(T... values)`: returns the first in the ordered list of provided values that implement the `Comparable` interface; returns `null` only when all values are `null`
    *   `T median(T... items)`: returns the value that is in the middle of the ordered list of provided values that implement the `Comparable` interface; if the count of the values is even, returns the smallest of the two in the middle
    *   `T median(Comparator<T> comparator, T... items)`: returns the value that is in the middle of the list of provided values ordered according to the provided `Comparator` object; if the count of the values is even, returns the smallest of the two in the middle
    *   `T mode(T... items)`: returns the most frequently occurring item from the items provided; returns `null` when such an item occurs most often or when there is no one item that occurs most often; here is the code that demonstrates this last method:

[PRE27]

# Managing strings

The class `String` is used a lot. So, you have to have a good handle on its functionality. We talked already talked about `String` value immutability in [Chapter 5](ddf91055-8610-4b8c-acc5-453cfa981760.xhtml), *Java Language Elements and Types*. We have shown that every time a `String` value is "modified", a new copy of the value is created, which means that in the case of multiple "modifications", many `String` objects are created, consuming memory and putting a burden on the JVM. 

In such cases, it is advisable to use the class `java.lang.StringBuilder` or `java.lang.StringBuffer` because they are modifiable objects and do not have an overhead of creating `String` value copies. We will show how to use them and explain the difference between these two classes in the first part of this section.

After that, we will review the methods of the class `String` and then provide an overview of the class `org.apache.commons.lang3.StringUtils`, which complements the class `String` functionality.

# StringBuilder and StringBuffer

 The classes `StringBuilder` and `StringBuffer` have exactly the same list of methods. The difference is that the methods of the class `StringBuilder` perform faster than the same methods of the class `StringBuffer`. That is because the class `StringBuffer` has an overhead of not allowing concurrent access to its values from different application threads. So, if you are not coding for multithreaded processing, use `StringBuilder`. 

There are many methods in the classes `StringBuilder` and `StringBuffer`.  But, we are going to show how to use only the method `append()`, which is by far the most popular, used for cases when multiple `String` value modifications are required. Its main function is to append a value to the end of the value already stored inside the `StringBuilder` (or `StringBuffer`) object. 

The method `append()` is overloaded for all primitive types and for the classes `String`, `Object`, `CharSequence`, and `StringBuffer`, which means that a `String` representation of the passed-in object of any of these classes can be appended to the existing value. For our demonstration, we are going use only the `append(String s)` version because that is what you are probably going to use most of the time. Here is an example:

[PRE28]

There are also methods `replace()`, `substring()`, and `insert()` in the class `StringBuilder` (and `StringBuffer`) that allow modifying the value further. They are used much less often than the method `append()` though, and we are not going to discuss them as they are outside the scope of this book.

# Class java.lang.String

The class `String` has 15 constructors and almost 80 methods. To talk details and demonstrate each of them is just too much for this book, so we will comment only on the most popular methods and just mention the rest. After you master the basics, you can read the online documentation and see what else you can do with other methods of the class `String`.

# Constructors

The constructors of the class `String` are useful if you are concerned that the strings your application creates consume too much memory. The problem is that `String` literals (`abc`, for example) are stored in a special area of the memory called the "string constant pool" and never garbage collected. The idea behind such a design is that `String` literals consume substantially more memory than numbers. Also, the handling of such large entities has an overhead that may tax the JVM. That is why the designers figured it is cheaper to store them and share them between all application threads than allocate new memory and then clean it up several times for the same value.

But if the rate of reuse of the `String` values is low, while the stored `String` values consume too much memory, creating a `String` object with a constructor may be the solution to the problem. Here is an example:

[PRE29]

A `String` object created this way resides in the heap area (where all objects are stored) and is garbage collected when not used anymore. That is when the `String` constructor shines.

If necessary, you can use the method `intern()` of the class `String` to create a copy of the heap `String` object in the string constant pool. It allows us not only to share the value with other application threads (in multithreaded processing), but also to compare it with another literal value by reference (using the operator `==`). If the references are equal, it means they point to the same `String` value in the pool.  

But, mainstream programmers rarely manage the memory this way, so we will not discuss this topic further.

# format()

The method `String format(String format, Object... args)` allows insertion of the provided objects into specified locations of a string and formatting them as needed. There are many format specifiers in the class `java.util.Formatter`. We will demonstrate here only `%s`, which converts the passed-in object to its `String` representation by invoking it on the object method `toString()`:

[PRE30]

# replace()

The method `String replace(CharSequence target, CharSequence replacement)` in the `String` value replaces the value of the first parameter with the value of the second one:

[PRE31]

There are also the methods `String replaceAll(String regex, String replacement)` and `String replaceFirst(String regex, String replacement)`, which have similar capabilities.

# compareTo()

We have already used the `int compareTo(String anotherString)` method in our examples. It returns the difference between the positions of this `String` value and the value of `anotherString` in an ordered list. It is used for the natural ordering of strings since it is an implementation of the `Comparable` interface.

The method `int compareToIgnoreCase(String str)` performs the same function but ignores the case of the compared strings and is not used for natural ordering because it is not an implementation of the `Comparable` interface. 

# valueOf(Objectj)

The static method `String valueOf(Object obj)` returns `null` if the provided object is `null`, or calls the method `toString()` on the object provided.

# valueOf(primitive or char[])

Any primitive type value can be passed as the parameter into the static method `String valueOf(primitive value)`, which returns the String representation of the value provided. For example, `String.valueOf(42)` returns `42`. This group of methods includes the following static methods:

*   `String valueOf(boolean b)`
*   `String valueOf(char c)`
*   `String valueOf(double d)`
*   `String valueOf(float f)`
*   `String valueOf(int i)`
*   `String valueOf(long l)`
*   `String valueOf(char[] data)`
*   `String valueOf(char[] data, int offset, int count)`

# copyValueOf(char[])

The method `String copyValueOf(char[] data)` is equivalent to `valueOf(char[])`, while the method `String copyValueOf(char[] data, int offset, int count)` is equivalent to `valueOf(char[], int, int)`. They return a `String` representation of a char array or its subarray.

And the method `void getChars(int srcBegin, int srcEnd, char[] dest, int dstBegin)` copies characters from this `String` value into the destination character array.

# indexOf() and substring()

Various `int indexOf(String str)` and `int lastIndexOf(String str)` methods return the position of a substring  in a string:

[PRE32]

Notice that the position count starts from zero.

The method `String substring(int beginIndex)` returns the rest of the string value, starting from the position (index) passed in as the parameter:

[PRE33]

The character with the `beginIndex` position is the first that is present in the preceding substring.

The method `String substring(int beginIndex, int endIndex)` returns the substring, starting from the position passed in as the first parameter, to the position passed in as the second parameter:

[PRE34]

As with the method `substring(beginIndex)`, the character with the `beginIndex` position is the first that is present in the preceding substring, while the character with the `endIndex` position is not included. The difference `endIndex - beginIndex` equals the length of the substring.

This means that the following two substrings are equal:

[PRE35]

# contains() and matches()

The method `boolean contains(CharSequence s)` returns `true` when the provided sequence of characters (substring) is present:

[PRE36]

Other similar methods are:

*   `boolean matches(String regex)`: uses a regular expression (not a subject of this book)
*   `boolean regionMatches(int tOffset, String other, int oOffset, int length)`: compares regions of two strings
*   `boolean regionMatches(boolean ignoreCase, int tOffset, String other, int oOffset, int length)`: same as above, but with the flag `ignoreCase` indicating whether to ignore the case

# split(), concat(), and join()

The methods `String[] split(String regex)` and `String[] split(String regex, int limit)` use the passed-in regular expression to split the strings into substrings. We do not explain regular expressions in this book. However, there is a very simple one that is easy to use even if you know nothing about regular expressions: if you just pass into this method any symbol or substring present in a string, the string will be broken (split) into parts separated by the passed-in value, for example:

[PRE37]

This code just illustrates the functionality. But the following code snippet is more practical:

[PRE38]

As you can see, the second parameter in the `split()` method limits the number of resulting substrings.

The method `String concat(String str)` adds the passed-in value to the end of the string:

[PRE39]

The `concat()` method creates a new `String` value with the result of concatenation, so it is quite economical. But if you need to add (concatenate) many values, using `StringBuilder` (or `StringBuffer`, if you need protection from concurrent access) would be a better choice. We discussed it in the previous section. Another option would be to use the operator `+`:

[PRE40]

The operator `+`, when used with `String` values, is implemented based on `StringBuilder`, so allows the addition of `String` values by modifying the existing one. There is no performance difference between using StringBuilder and just the operator `+` for adding `String` values.

The methods `String join(CharSequence delimiter, CharSequence... elements)` and `String join(CharSequence delimiter, Iterable<? extends CharSequence> elements)` are based on `StringBuilder` too. They assemble the provided values in one `String` value using the passed-in `delimiter` to separate the assembled values inside the created `String` result. Here is an example:

[PRE41]

# startsWith() and endsWith()

The following methods return `true` when the String value starts (or ends) with the provided substring `prefix`:

*   `boolean startsWith(String prefix)`
*   `boolean startsWith(String prefix, int toffset)`
*   `boolean endsWith(String suffix)`

Here is the demo code:

[PRE42]

# equals() and equalsIgnoreCase()

We have used the method `boolean equals(Object anObject)` of the class `String` several times already and have pointed out that it compares this `String` value with other objects. This method returns `true` only when the passed-in object is `String` with the same value.

The method `boolean equalsIgnoreCase(String anotherString)` does the same but also ignores case, so the strings `AbC` and `ABC` are considered equal.

# contentEquals() and copyValueOf()

The method `boolean contentEquals(CharSequence cs)` compares this `String` value with the `String` representation of an object that implements the interface `CharSequence`. The popular `CharSequence` implementations are `CharBuffer`, `Segment`, `String`, `StringBuffer`, and `StringBuilder`.

The method `boolean contentEquals(StringBuffer sb)` does the same but for `StringBuffer` only. It has slightly different implementation than `contentEquals(CharSequence cs)` and may have some performance advantages in certain situations, but we are not going to discuss such details. Besides, you probably will not even notice which of the two methods is used when you call `contentEquals()` on a `String` value unless you make an effort to exploit the difference.

# length(), isEmpty(), and hashCode()

The method `int length()` returns the number of characters in a `String` value.
The method `boolean isEmpty()` returns `true` when there are no characters in the `String` value and the method `length()` returns zero.

The method `int hashCode()` returns a hash value of the `String` object.

# trim(), toLowerCase(), and toUpperCase()

The method `String trim()` removes leading and trailing whitespaces from a `String` value.

The following methods change the case of the characters in a `String` value:

*   `String toLowerCase()`
*   `String toUpperCase()`
*   `String toLowerCase(Locale locale)`
*   `String toUpperCase(Locale locale)`

# getBytes(), getChars(), and toCharArray()

The following methods convert the `String` value to a byte array, optionally encoding it using the given charset:

*   `byte[] getBytes()`
*   `byte[] getBytes(Charset charset)`
*   `byte[] getBytes(String charsetName)`

And these methods convert all the `String` value to other types, or only part of it:

*   `IntStream chars()`
*   `char[] toCharArray()`
*   `char charAt(int index)`
*   `CharSequence subSequence(int beginIndex, int endIndex)`

# Get code point by index or stream

The following group of methods convert all the `String` value, or only part of it, into Unicode code points of its characters: 

*   `IntStream codePoints()`
*   `int codePointAt(int index)`
*   `int codePointBefore(int index)`
*   `int codePointCount(int beginIndex, int endIndex)`
*   `int offsetByCodePoints(int index, int codePointOffset)`

We explained Unicode code points in [Chapter 5](ddf91055-8610-4b8c-acc5-453cfa981760.xhtml), *Java Language Elements and Types*. These methods are especially useful when you need to represent characters that *do not fit* into the two bytes of the `char` type. Such characters have code points bigger than `Character.MAX_VALUE`, which is  `65535`.

# Class lang3.StringUtils

The class `org.apache.commons.lang3.StringUtils` of the Apache Commons library has more than 120 static utility methods that complement those of the class `String` we described in the previous section. 

Among the most popular are the following static methods:

*   `boolean isBlank(CharSequence cs)`: returns `true` when the passed-in parameter is an empty `String` "", `null`, or whitespace
*   `boolean isNotBlank(CharSequence cs)`: returns `true` when the passed-in parameter is not an empty `String` "", `null`, or whitespace
*   `boolean isAlpha(CharSequence cs)`: returns `true` when the passed-in parameter contains only Unicode letters
*   `boolean isAlphaSpace(CharSequence cs)`: returns `true` when the passed-in parameter contains only Unicode letters and spaces ('  ')
*   `boolean isNumeric(CharSequence cs)`: returns `true` when the passed-in parameter contains only digits
*   `boolean isNumericSpace(CharSequence cs)`: returns `true` when the passed-in parameter contains only digits and spaces ('  ')
*   `boolean isAlphaNumeric(CharSequence cs)`: returns `true` when the passed-in parameter contains only Unicode letters and digits
*   `boolean isAlphaNumericSpace(CharSequence cs)`: returns `true` when the passed-in parameter contains only Unicode letters, digits, and spaces ('  ')

We highly recommend you look through the API of this class and get a feel for what you can find there.

# Managing time

There are many classes in the `java.time` package and its sub-packages. They were introduced as a replacement for other—older—packages that handle date and time. The new classes are thread-safe (so better suited for multithreaded processing) and, no less important, are more consistently designed and easier to understand. Also, the new implementation follows the **International Standard Organization** (**ISO**) for date and time formats, but allows the use of any other custom format too.

We will describe the main five classes and demonstrate how to use them:

*   `java.util.LocalDate`
*   `java.util.LocalTime`
*   `java.util.LocalDateTime`
*   `java.util.Period`
*   `java.util.Duration`

All these, and other classes of the `java.time` package and its sub-packages, are rich in various functionalities that cover all practical and any imaginable cases. But we are not going to cover all of them, just introduce the basics and most popular use cases.

# java.time.LocalDate

The class `LocalDate` does not carry time. It represents a date in ISO 8601 format, yyyy-MM-DD:

[PRE43]

As you can see, the method `now()` returns the current date as it is set on your computer: `April 14, 2018` was the date when this section was written. 

Similarly,  you can get the current date in any other timezone using the static method `now(ZoneId zone)`. The `ZoneId` object can be constructed using the static method `ZoneId.of(String zoneId)`, where `String zoneId` is any of the `String` values returned by the method `ZonId.getAvailableZoneIds()`: 

[PRE44]

This code prints many timezone IDs, one of them being `Asia/Tokyo`. Now, we can find what the date is now, in that time zone:

[PRE45]

An object of `LocalDate` can represent any date in the past or in the future too, using the following methods:

*   `LocalDate parse(CharSequence text)`: constructs an object from a string in ISO 8601 format, yyyy-MM-DD
*   `LocalDate parse(CharSequence text, DateTimeFormatter formatter) `: constructs an object from a string in a format specified by the object `DateTimeFormatter`, which has many predefined formats
*   `LocalDate of(int year, int month, int dayOfMonth)`: constructs an object form a year, month, and day
*   `LocalDate of(int year, Month month, int dayOfMonth)`: constructs an object from a year, month (as `enum` constant), and day
*   `LocalDate ofYearDay(int year, int dayOfYear)`: constructs an object from a year and day-of-year

The following code demonstrates these methods:

[PRE46]

Using the `LocalDate` object, you can get various values:

[PRE47]

The `LocalDate` object can be modified:

[PRE48]

The `LocalDate` objects can be compared:

[PRE49]

There are many other helpful methods in the `LocalDate` class. If you have to work with dates, we recommend that you read the API of this class and other classes of the `java.time` package and its sub-packages.

# java.time.LocalTime

The class `LocalTime` contains the time without a date. It has methods similar to those of the class `LocalDate`.

Here is how an object of the `LocalTime` class can be created:

[PRE50]

Each component of the time value can be extracted from a `LocalTime` object as follows:

[PRE51]

The object can be modified:

[PRE52]

And two objects of the `LocalTime` class can be compared as well:

[PRE53]

There are many other helpful methods in the `LocalTime` class. If you have to work with time, we recommend you read the API of this class and other classes of the `java.time` package and its sub-packages.

# java.time.LocalDateTime

The class `LocalDateTime` contains both date and time, and has all the methods the classes `LocalDate` and `LocalTime` have, so we are not going to repeat them here. We will only show how an object of `LocalDateTime` can be created:

[PRE54]

There are many other helpful methods in the `LocalDateTime` class. If you have to work with date and time, we recommend you read the API of this class and other classes of the `java.time` package and its sub-packages.

# Period and Duration

The classes `java.time.Period` and `java.time.Duration` are designed to contain an amount of time:

*   The `Period` object contains an amount of time in units of years, months, and days
*   The `Duration` object contains an amount of time in hours, minutes, seconds, and nanoseconds

The following code demonstrates their creation and use with the class `LocalDateTime`, but the same methods exist in the classes `LocalDate` (for `Period`) and `LocalTime` (for `Duration`):

[PRE55]

Some other ways to create and use `Period` objects are demonstrated in the following code:

[PRE56]

Objects of `Duration` can be similarly created and used:

[PRE57]

There are many other helpful methods in the classes `Period` and `Duration`. If you have to work with the amount of time, we recommend you read the API of these classes and other classes of the `java.time` package and its sub-packages.

# Managing random numbers

Generating a truly random number is a big topic that does not belong to this book. But for the vast majority of practical purposes, the pseudo-random number generators provided by Java are good enough, and that is what we are going to discuss in this section.

There are two primary ways to generate a random number in Java Standard Library:

*   The `java.lang.Math.random()` method
*   The `java.util.Random` class

There is also the `java.security.SecureRandom` class, which provides a cryptographically strong random number generator, but it is outside the scope of an introductory course.

# Method java.lang.Math.random()

The static method `double random()` of the class `Math` returns a `double` type value greater than or equal to `0.0` and less than `1.0`:

[PRE58]

We captured the result in the previous comments. But in practice, more often than not, a random integer from a certain range is required. To accommodate such a need, we can write a method that, for example, produces a random integer number from 0 (inclusive) to 10 (exclusive):

[PRE59]

Here is the result of one run of the previous code:

[PRE60]

As you can see, it generates a random integer value that can be one of the following 10 numbers: 0, 1, ..., 9\. And here is the code that uses the same method and produces random integer numbers from 0 (inclusive) to 100 (exclusive):

[PRE61]

And when you need a random number between 100 (inclusive) and 200 (exclusive), you can just add 100 to the preceding result:

[PRE62]

Including both ends of the range in the result can be done by rounding the generated `double` value:

[PRE63]

When we used the preceding method, the results were:

[PRE64]

As you can see, the upper end of the range (the number 200) is included in the possible results set. The same effect can be achieved by just adding 1 to the requested upper range:

[PRE65]

If we use the previous method, we can get the following result:

[PRE66]

But if you look at the source code of the `Math.random()` method, you will see that it uses the `java.util.Random` class and its `nextDouble()` method to generate a random double value. So, let's look at how to use the `java.util.Random` class directly.

# Class java.util.Random

The method `doubles()` of the class `Random` generates a `double` type value greater than or equal to `0.0` and less than `1.0`:

[PRE67]

We can use the method `nextDouble()` the same way we used `Math.random()` in the previous section. But class have other methods that can be used without creating a custom `getInteger()` method when a random integer value of a certain range is required. For example, the `nextInt()` method returns an integer value between `Integer.MIN_VALUE` (inclusive) and `Integer.MAX_VALUE` (inclusive):

[PRE68]

And the same method with a parameter allows us to limit the range of the returned values by the upper limit (exclusive):

[PRE69]

This code generates a random integer value between 0 (inclusive) and 10 (inclusive). And the following code returns a random integer value between 11 (inclusive) and 20 (inclusive):

[PRE70]

Another way to generate a random integer from the range is by using the `IntStream` object returned by the method `ints(int count, int min, int max)`, where `count` is the number of requested values, `min` is the minimum value (inclusive), and `max` is the maximum value (exclusive): 

[PRE71]

This code returns three integer values from 0 (inclusive) to 100 (inclusive). We will talk more about streams in [Chapter 18](be052e15-ac84-4e19-9bd9-24548aa3f904.xhtml), *Streams and Pipelines*.

# Exercise – Objects.equals() result

There are three classes:

[PRE72]

What is going to be displayed when we run the `main()` method of the `Exercise` class? `Error`? `False`? `True`?

# Answer

The display will show only one value: `True`. The reason is that both private fields—`a` and `b`—are initialized to `null`.

# Summary

In this chapter, we introduced the reader to the most popular utilities and some other classes from the Java Standard Library and Apache Commons libraries. Every Java programmer has to have a solid understanding of their capabilities in order to become an effective coder. Studying them also helps to get exposure to various software designs pattern and solutions that are instructive and can be used as patterns for best coding practices in any application.

In the next chapter, we are going to demonstrate to the reader how to write Java code that can manipulate—insert, read, update, and delete—data in a database. It will also provide a short introduction to SQL and basic database operations.