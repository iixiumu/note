An interface variable can store any concrete (non-interface) value as long as that value implements the interface's methods.
只要某个具体值 （非接口）实现了某个接口中的方法，该接口类型的变量就能存储它。

An extremely important example of an interface type is the empty interface:interface{}
It represents the empty set of methods and is satisfied by any value at all, since any value has zero or more methods.
它表示空方法集。由于任何值都有零个 或多个方法，因此任何值都满足它。

1. Reflection goes from interface value to reflection object.
从接口值可反射出反射对象。
2. Reflection goes from reflection object to interface value.
从反射对象可反射出接口值
3. To modify a reflection object, the value must be settable.
要修改反射对象，其值必须可设置。