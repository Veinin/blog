---
title: Lua C API简介(中篇) 
date: 2017-08-06 22:15:58
categories: Lua
tags: [Lua, Lua C API]
---

在运行C程序时，可以调用Lua脚本。C程序可以向Lua传入参数，然后通过Lua返回结果。
使用C API调用Lua函数方法很简单，我们最开始先将要调用的函数入栈，然后依次将要传入的参数入栈，调用pcall函数。最后，再从栈中获取函数返回的值。

<!--more-->

## 调用Lua简单脚本

``` lua
-- helloscript.lua

io.write("This is comming from lua\n")
```

``` c
// helloscript.c

#include <stdio.h>
#include <stdlib.h>

#include "lua.h"
#include "lauxlib.h"
#include "lualib.h"

void error(lua_State *L, const char *fmt, ...);

int main()
{
	lua_State *L = luaL_newstate();
	luaL_openlibs(L);

	if(luaL_loadfile(L, "call_lua.lua"))
		error(L, "luaL_loadfile failed.\n");

	printf("In C, calling Lua\n");

	if(lua_pcall(L, 0, 0, 0))
		error(L, "lua_pcall() failed.", lua_tostring(L, -1));

	printf("Back in C again\n");

	lua_close(L);

	return 0;
}

void error(lua_State *L, const char *fmt, ...) 
{
	va_list argp;
	va_start(argp, fmt);
	vfprintf(stderr, fmt, argp);
	va_end(argp);
	lua_close(L);
	exit(EXIT_FAILURE);
}

```

### 相关函数解析
`luaL_loadfile(L, "script.lua");`
`lua_loadfile` 等价于 `luaL_loadfilex`，加载为 Lua 代码块。此函数的返回值和 `lua_load` 相同，不过它还可能产生一个叫做 `LUA_ERRFILE` 的出错码。这种错误发生于无法打开或读入文件时，或是文件的模式错误。和 `lua_load` 一样，这个函数仅加载代码块不运行。

`lua_pcall(L, number_of_args, number_of_returns, errfunc_idx);`
调用Lua函数，第二个参数位传入参数数量; 第三个参数为调用函数返回值数量; `errfunc_idx` 是0，则返回在栈顶的错误消息就和原始错误消息完全一致，否则，`msgh` 就被当成是错误处理函数在栈上的索引位置。

### 编译、运行(系统：OSX Lua版本:5.3.0)：
``` shell
$ cc -o helloscript helloscript.c -I /usr/local/include/ -L /usr/local/lib/ -llua
$ ./helloscript
In C, calling Lua
This is comming from lua
Back in C again
```

## 调用Lua函数，传值并返回

``` lua
-- call_lua_func.lua 

function sayHello()
	io.write("This is comming from lua.sayHello.\n")
end

function add(a, b)
	print("This is comming from lua.add. arg.a =", a, " arg.b =", b)
	return a + b
end
```

``` c
// 04_call_lua_func.c

#include <stdio.h>
#include <stdlib.h>

#include "lua.h"
#include "lauxlib.h"
#include "lualib.h"

void error(lua_State *L, const char *fmt, ...);

int main()
{
	lua_State *L = luaL_newstate();
	luaL_openlibs(L);

	if(luaL_loadfile(L, "call_lua_func.lua"))
		error(L, "luaL_loadfile failed.\n");

	if(lua_pcall(L, 0, 0, 0))
		error(L, "lua_pcall failed.\n");

	printf("In C, calling Lua->sayHello()\n");

	lua_getglobal(L, "sayHello");	//Tell it to run test2.lua -> sayHello()
	if(lua_pcall(L, 0, 0, 0))
		error(L, "lua_pcall failed.\n");

	printf("Back in C again\n");
	printf("\nIn C, calling Lua->add()\n");

	lua_getglobal(L, "add");		//Tell it to run test2.lua -> add()
	lua_pushnumber(L, 1);
	lua_pushnumber(L, 5);
	if(lua_pcall(L, 2, 1, 0))
		error(L, "lua_pcall failed.\n");

	printf("Back in C again\n");
	int returnNum = lua_tonumber(L, -1);
	printf("Returned number : %d\n", returnNum);

	lua_close(L);

	return 0;
}

void error(lua_State *L, const char *fmt, ...) 
{
	va_list argp;
	va_start(argp, fmt);
	vfprintf(stderr, fmt, argp);
	va_end(argp);
	lua_close(L);
	exit(EXIT_FAILURE);
}
```

### 相关函数解析
在 Lua 脚本中， `sayHello` 函数只做输出操作，而 `add` 函数则需要传入两值，并进行加法操作并返回结果。

从 C 层在调用 Lua 脚本层时，首先要把被调用的函数压入栈中，可以通过 `lua_getglobal` 函数把调用函数入栈。

把需要传递给被调用函数的参数用 `lua_push*` 函数按正序压栈。

最后调用一下 `lua_call`，把要传入的参数个数及返回值个数一起传进去。  

当函数调用完毕后，所有的参数以及函数本身都会出栈。紧接着函数的所有返回值这时则被压栈。Lua 会保证返回值都放入栈空间中。函数返回值将按正序压栈（第一个返回值首先压栈），因此在调用结束后，最后一个返回值将被放在栈顶。

### 编译、运行
``` shell
$ cc -o callluafunc callluafunc.c -I /usr/local/include/ -L /usr/local/lib -llua
$ ./callluafunc
In C, calling Lua->sayHello()

This is comming from lua.sayHello.

Back in C again

In C, calling Lua->add()

This is comming from lua.add. arg.a = 1.0 arg.b = 5.0

Back in C again

Returned number : 6
```


## 表传递
在对Lua函数调用值传递时经常涉及到表的传递。通过下面代码我们来讨论它是如何执行的。

``` lua
-- call_lua_table.lua

-- 该函数接受一个 table，并且将新建一个 talbe，将穿入的 table 键值都插入新建 table 中，并记录数据长度，最后返回新建立的 table。
function tablehandler(t)
	local returnedt = {numfields = 1}

	for i, v in pairs(t) do
		returnedt.numfields = returnedt.numfields + 1
		returnedt[tostring(i)] = tostring(v)
	end

	io.write("this is comming from table handler. table num fields : ", returnedt.numfields, "\n")
	return returnedt
end
```

``` c
// 05_call_lua_table.c 

#include <stdio.h>
#include <stdlib.h>

#include "lua.h"
#include "lauxlib.h"
#include "lualib.h"

void error(lua_State *L, const char *fmt, ...);

int main()
{
	lua_State *L = luaL_newstate();
	luaL_openlibs(L);

	if(luaL_loadfile(L, "call_lua_table.lua"))
		error(L, "luaL_loadfile failed.\n");

	if (lua_pcall(L, 0, 0, 0))
		error(L, "lua_pcall failed");

	printf("In C,  calling Lua->tablehandler()\n");
	lua_getglobal(L, "tablehandler");
	lua_newtable(L);			//新建一个table入栈
	lua_pushliteral(L, "firstname");	//键为"firstname"入栈，此时table位置-2
	lua_pushliteral(L, "Veinin");		//值为"Veinin"入栈，次数table位置-3
	lua_settable(L, -3);			//把key和value放入表中，操作完成后键和值都会被弹出栈

	lua_pushliteral(L, "lastname");
	lua_pushliteral(L, "Guo");
	lua_settable(L, -3);

	if(lua_pcall(L, 1, 1, 0))			//执行调用函数
		error(L, "lua_pcall failed");

	printf("============ Back in C again, Iterating thru returned table ============\n");

	lua_pushnil(L);					//第一个键，
	const char *k, *v;
	while(lua_next(L, -2)) {			//table现在在-2位置，lua_next得到一个键-值对，分别入栈
		v = lua_tostring(L, -1);
		k = lua_tostring(L, -2);

		lua_pop(L, 1);				//’值’出栈，‘键’不出栈，保留做下一次迭代

		printf("%s = %s\n", k, v);
	}

	lua_close(L);

	return 0;
}

void error(lua_State *L, const char *fmt, ...) 
{
	va_list argp;
	va_start(argp, fmt);
	vfprintf(stderr, fmt, argp);
	va_end(argp);
	lua_close(L);
	exit(EXIT_FAILURE);
}
```

### 相关函数解析
对于 C 层代码，我们可以通过 `lua_newtable` 函数创建一张空表，并将其压栈，该函数它等价于 `lua_createtable(L, 0, 0)`。
然后通过 `lua_pushliteral` 把键和值分别压入栈中，并通过 `lua_settable` 把 `key` 和 `value` 放入 table 中，做一个等价于 `t[k] = v` 的操作，在操作完成后，这个函数会将键和值都弹出栈。

在被调用函数 `tablehandler` 返回后，此时，栈中存在的只是函数的返回值，而它目前是一张 table，对于迭代输出 table 中的值，上面给出了一种典型的遍历方法。

``` c
int lua_next (lua_State *L, int index);
```

`lua_next` 可以从栈顶弹出一个键，然后把索引指定的表中的一个键值对压栈（弹出的键之后的 “下一” 对），在这一步开始之前我们需要调用 `lua_pushnil(L)` 设置第一个键为缺省值，然后再调用 `lua_next` 。
通常，我们会在下一个迭代到来之前，把‘值’出栈，但会保留‘键’做下一次迭代操作。

### 编译、运行
``` shell
$ cc -o 05_call_lua_table 05_call_lua_table.c -I /usr/local/include/ -L /usr/local/lib/ -llua
$ ./05_call_lua_table
In C,  calling Lua->tablehandler()

this is comming from table handler. table num fields : 3

============ Back in C again, Iterating thru returned table ============
lastname = Guo

firstname = Veinin

numfields = 3
```
