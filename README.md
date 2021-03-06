# CodeDetector
代码相似度匹配

新的方法：
1. 先将conala-test.json的代码用 replaceCode方法保证语义不变的情况下，替换成相对"泛化"一点的代码，暂时保存在文件里
		
2. 然后对于一段输入的代码，做以下操作:

	a) 先把字符串，普通变量名之类的常量做替换，
	b) 用code2tokens函数切割代码为一个个token组成的list
	c) 遍历这个token_list，每次用正则在文件中匹配有这个token的行代码，返回行序号
	d) 对于每个token，都有一个由行序号组成的list
	e) 对这些list，找出它们之间的重合度最高的（就是这个token_list里的token在哪一行代码里出现的最多），生成一个dict，键是行序号，值是次数
	f) 也就是 次数越大，就跟目标代码越相似
	
	注: 考虑可以优化的地方，就是token的优先级。

2018-10-31
实现步骤：存取方式暂时用文件，后面考虑用数据库
1. 对库代码做一些准备：
	对库代码求一个高优先级token，生成文件high_PRI_tokens（后面都用数据库
	文件格式: 每一行=> 行序号+对应行的高优先级token （以空格间隔
	
2. 输入待检测代码（一行）
	a) 分割token，求出高优先级token h_tokens和低优先级token l_tokens
	b) 遍历高优先级token，
		每次在high_PRI_tokens中查找对应行，每个token返回一个行数的列表
		然后进行一次字典汇总，字典的键是行数，值是此行出现的次数
	c) 对上一步中的字典，通过字典的键（也就是行数）从库代码中获取字典对应的行代码
		生成一个文本（比原来缩小很多），格式是：原始行序号+行代码
	d) 在这个文本中遍历低优先级token l_tokens
		跟b步中后面相同，正则查找对应行，每个token返回一个行数的列表，
		然后进行一次字典汇总，字典的键是行数，值是此行出现的次数
	e) 最后想要获取相似度前n(可自己设定)的代码行数，通过行数也可以从库中获取相应行的代码

2018-11-1：
考虑：
1. token顺序
2. 更加准确的提取有意义的token

对于1，就考虑在提取一行代码的时候，在token的最后添加上一个数字表示token的顺序

对于2，主要修改code2tokens和splitTokensByPRI函数
