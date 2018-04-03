### 第3章 对所有对象都通用的方法

ALTHOUGH Object is a concrete class, it is designed primarily for

extension. All of its nonfinal methods

\(equals, hashCode, toString, clone, and finalize\) have explicit general

contracts because they are designed to be overridden. It is the

responsibility of any class overriding these methods to obey their

general contracts; failure to do so will prevent other classes that

depend on the contracts \(such as HashMap and HashSet\) from

functioning properly in conjunction with the class.

This chapter tells you when and how to override the

nonfinal Object methods. The finalizemethod is omitted from this

chapter because it was discussed in Item 8. While notan Object method, Comparable.compareTo is discussed in this chapter

because it has a similar character.
