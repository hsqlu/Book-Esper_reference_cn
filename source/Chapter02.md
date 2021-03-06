﻿ Chapter02
 
 事件展现形式

	2.1 用Java对象表示事件

	
	事件是对一个过去发生的动作或者状态改变的不可变记录。事件属性能用来描述一个事件的状态信息。

	
	在Esper里面，一个事件可以用下面任意一种Java对象来表示：
	
	+-----------------------------------------+-----------------------------------------------+
	|Java Class				  |			描述			  |
	+=========================================+===============================================+	
	|java.lang.Object			  |	任何有getter方法，遵守JavaBean规范的Java  |
	|					  |	POJO;那些遗留下来但不遵守JavaBean规范的   |
	|					  |	Java类也可以当作事件。			  |
	+-----------------------------------------+-----------------------------------------------+
	|java.util.Map	 			  |	继承自java.util.Map接口的Map事件，每一个  |
	|					  |	Map entry是一个属性值。			  |
	+-----------------------------------------+-----------------------------------------------+
	|Object[] (array of object)		  |	数组对象事件是一些对象数组，每个数组元素是|
	|					  |	一个属性值。			          |
	+-----------------------------------------+-----------------------------------------------+			
	|org.w3c.dom.Node			  |	XML文件对象模型(DOM)			  |
	+-----------------------------------------+-----------------------------------------------+
	|org.apache.axiom.om.OMDocument		  |	XML - XML的流API(StAX) - Apache Axiom(Esp-|
	|or OMElement				  |	erIO包提供)				  |
	+-----------------------------------------+-----------------------------------------------+
	|Application classes			  |	通过扩展API表示的插件事件。		  |	
	+-----------------------------------------+-----------------------------------------------+	

	Esper为表示一个事件提供了多种选择。没有必须要求你创建一个新的Java类来表示它。
	
	事件表示有一下共同之处：
	
	- 所有的事件表示形式都支持嵌套，索引和映射属性，后面有详细解释。没有嵌套层数限制。
	- 所有的事件表示形式都提供事件类型元数据。包括嵌套属性的类型元数据。
	- 所有的事件表示形式都允许将一个事件或它的部分属性图置换到一个新的事件。这里的置换是指选择事件
	本身或者嵌套在它们内部的属性图，然后在之后的语句查询。Apache Axiom事件表示是一个例外，它现在不
	支持置换事件属性但是支持置换事件本身。
	- Java Object, Map和对象数组支持超类型。
	
	对于所有事件表示形式的API行为是一样的，细小的差别会的本章指出。
	
	多事件表示形式的好处有：
	
	- 对于那些已经使用上面所述的一种形式来表示事件的应用来说，不需要再在处理前把他们转换成Java对象了。
	- 在对象表示形式发生变化时，可以替换，减少甚至消除更改语句的必要。
	- 这些事件表示形式是可以互操作的，允许它们在相同或者不同的语句下互相操作。
	- 这样可以有意识的权衡性能，易用性，改进的能力和导入或者外部化一个事件所需的工作以及使用已有的类
	型元数据。	
	
	2.2 事件属性
	
	事件属性描述了一个事件的状态信息。事件属性有简单，索引，映射以及嵌套其他的事件属性。下面的表格
	给出了不同的属性类型以及它们在事件表示中的语法。这些语法可以使一个语句能够查询JavaBean对象图，XML结构
	和Map事件。
	
	表2.2. 事件属性类型
	
	+-------+-------------------------------------------------------+-----------------------+---------------+
	| 类型	| 描述							| 语法			| 示例		|
	+=======+=======================================================+=======================+===============+
	| 简单	| 一个属性有一个可以被检索的单一值			| name			| sensorId	|
	+-------+-------------------------------------------------------+-----------------------+---------------+
	| 索引	| 一个索引属性存储一个对象的有序集合（同一类型的对象）。| name[index]		| sensor[0]	|
	|	| 可以用非负的整数下标单独访问每个对象。		|			|		|	
	+-------+-------------------------------------------------------+-----------------------+---------------+
	| 映射	| 一个映射属性存储一组可以通过key集合索引的对象集合（同 | name['key']		| sensor('light'|
	|	| 一类型）。						|			| )		|
	+-------+-------------------------------------------------------+-----------------------+---------------+
	| 嵌套	| 嵌套属性可以在替他的属性里面嵌套			| name.nestedname	| sensor.value	|
	+-------+-------------------------------------------------------+-----------------------+---------------+
	
	不同类型的属性也是可以组合的。比如：person.address('home').street[0].
	
	可以使用使用任意表达式作为映射属性的key和索引属性的索引。试着在下文里找些例子。
	
	2.2.1 转义字符
	
	如果你的应用有在使用java.util.Map，Object[]或者XML来表示事件，那么事件的属性名称里面可能会包含'.'这个字符。
	反斜杠'\'可以用来做转义字符，从而是属性名称里面可以包含'.'。
	
	例如，下面的EPL希望从MyEvent中得到一个以part1.part2命名的事件类型：
	
	select part1\.part2 from MyEvent
	
	有时你命名的事件属性可能跟EPL语句的关键词重复了，或者包含空格或者其他的特殊字符。这时候你可能要使用'`'(后
	撇号)来做转义处理。
	
	下面的例子假设一个Quote事件有一个名为order的属性，然后order是一个保留关键字：
	
	select `order`, price as `price.for.goods` from Quote
	
	当对映射或者索引属性进行转义的时候，要确保转义在映射的key或索引里面进行。
	
	下面的EPL是一个属性名称包含空格(e.g. candidate book), 有特殊标记(e.g. children's books), 是一个索引(e.g. 
	children's books[0])和一个映射属性并且key是一个保留关键字(e.g. book select('isbn'))的语句: 
	
	select `candidate book` , `children's books`[0], `book select`('isbn') from MyEventType
	
	2.2.2 Key或者index value是表达式
	键或者索引表达式必须放在括号内。当使用表达式作为键时，这个表达式必须返回的是一个String类型。同理，在使用表达式作为索引时，这个表达式必须返回的是一个int类型。
	
	下面的例子是使用Java类来演示的。同样的原理适用于所有事件表示方式。
	
	假设一个类声明了下面这些属性（为了简单省略了getters）
	
		public class MyEventType {
			String myMapKey;
			
			int myIndexValue;
			
		  	int myInnerIndexValue;
		  	
		  	Map<String, InnerType> innerTypesMap;	// mapped property
		  	
		  	InnerType[] innerTypesArray; // indexed property
		}
		
		public class InnerType {
		
		  	String name;
		  	
		  	int[] ids;
		}
	
	下面用表达式作为Key和Index的一个简单的EPL表达式：
	
		select innerTypesMap('somekey').ids[1],
		
	  		innerTypesMap(myMapKey).getIds(myIndexValue),
	  		
	  		innerTypesArray[1].ids[2],
	  		
	  		innerTypesArray(myIndexValue).getIds(myInnerIndexValue)
	  		
		from MyEventType
	  	
	注意以下限制：
	
	对应方括号里的索引，可以是一个表示式并且是常量。
	
	对于使用'.'操作符来说，用作Key或者Index的表达式必须遵循方法链调用语法。
	
		
	2.3 事件的动态属性
	
	

	2.4. Fragment 和 Fragment类型 （ 是否翻译为碎片？ ）

	2.5. POJO 对象事件

	2.6 java.util.Map 事件

	2.7 对象数组 ( Object[] ) 事件

	2.8 org.w3c.dom.Node XML事件

	2.9 扩展事件表示

	2.10 更新，合并事件和对事件的版本控制

	2.11 粗粒度表示事件

	2.12 使用Insert Into 实例化和填充事件对象

	2.13 不同类型事件的对比
