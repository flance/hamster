#freemarker不显示对象的属性值的原因

新手使用时有可能遇到这个问题，遇到一次就不会再犯。
简直是很隐蔽的巨坑，可能原因有以下两个：

	1. Java对象的field没有对应的getter, freemarker是调用getter方法来输出数据值的。

	2. Java对象不是public class, freemarker永远只处理public class，其他任何class都直接忽略。。。