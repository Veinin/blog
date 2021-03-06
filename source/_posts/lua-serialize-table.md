---
title: Lua 表序列化与反序列化
date: 2017-11-01 22:11:17
categories: Lua
tags: [Lua, Lua Serialize Table]
---

## 前言

今天看了下同事写的关于 Lua 序列化的代码，觉得代码存在几个问题，其主要欠缺以下几点：
### 1.支持循环引用的 table，反序列后，能正确恢复循环引用状态，如：
``` lua
local a = {1, 2, 3}
a.b = {4, 5, 6, a}
```

<!--more-->

----

### 2.字符串内支持内嵌双引号、支持转义字，如一下字符串：
``` lua
local s = "ss\"aa\"bb\ncc"
```
序列化后我希望是这样子：
``` lua
'ss"aa"bb\ncc'
```

----

### 3.Table 数组部分序列化后隐藏每个值得索引值，如：
``` lua
local t = {4, 7, 9}
```
如果保留数组的索引值，会是这样子：
``` lua
{[1]=4,[2]=7,[3]=9}
```
为了更加节省空间，我希望的是这样子：
``` lua
{4,7,9}
```

----

## 实现
对于以上几点要求，Google 搜了下，并没有找到满足上面需求的合适版本，于是在前人的基础上做了一些改进。
实现部分，序列化函数函数为 table.tostring。反序列化函数相对来说比较简单，可以直接通过函数loadstring进行加载，下面实现为函数 table.loadstring

``` lua
function table.tostring(t)
	local mark   = {}
	local assign = {}
 
	local function serialize(tbl, parent)
		mark[tbl] = parent
		local tmp = {}
		for k, v in pairs(tbl) do
			local typek = type(k)
			local typev = type(v)

			local key = typek == "number" and "[" .. k .."]" or k

			if typev == "table" then
				local dotkey = parent .. (typek == "number" and key or "." .. key)
				if mark[v] then
					table.insert(assign, dotkey .. "=" .. mark[v])
				else
					if typek == "number" then
						table.insert(tmp, serialize(v,dotkey))
					else
						table.insert(tmp, key .. "=" .. serialize(v, dotkey))
					end
				end
			else
				if typev == "string" then
					v = string.gsub(v, "\n", "\\n")
					if string.match( string.gsub(v,"[^'\"]",""), '^"+$' ) then
						v = "'" .. v .. "'"
					else
						v = '"' .. v .. '"'
					end
				else
					v = tostring(v)
				end

				if typek == "number" then
					table.insert(tmp, v)
				else
					table.insert(tmp, key .. "=" .. v)
				end
			end
		end
		return "{" .. table.concat(tmp, ",") .. "}"
	end
 
	return serialize(t, "ret") .. table.concat(assign," ")
end

function table.loadstring(str)
	local chunk = loadstring("do local ret = " .. str .. " return ret end")
	if chunk then
		return chunk()
	end
end
```

测试代码：
```lua
local t = {a = 1, b = 2}
t.rt = {c = 3, d = 4, t}

local s = table.tostring(t)
print(s) -- 输出 {b=2,a=1,rt={c=3,d=4}}ret.rt[1]=ret

local tl = table.loadstring(s)
assert(tl.a == t.a)

----------------------------------------------

local t = {['foo']='bar', 11, 22, 33, "ss\"aa\"bb\ncc", "hello", {'a','b'}}

local s = table.tostring(t)
print(s) -- 输出 {11,22,33,'ss"aa"bb\ncc',"hello",{"a","b"},foo="bar"}

local tl = table.loadstring(s)
assert(tl.foo == t.foo)
assert(tl[4] == 'ss"aa"bb\ncc')
```

----

### 参考
- [云风的个人空间](https://blog.codingnow.com/cloud/LuaSerializeTable)
- [lua-users wiki](http://lua-users.org/wiki/TableUtils)