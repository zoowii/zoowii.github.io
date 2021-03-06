---
title: Lua的一些坑的记录
date: 2017-03-08 10:02:32
tags: Lua
---

Lua是一门很小巧的编程语言，不过使用过程中发下一些容易出现问题的地方，这里记录一下（API正常使用不记录）。记录时使用的版本是官方Lua 5.3.4版本源码.

* Lua的table区分数组部分和哈希表部分，数组部分索引从1开始，而不是0-based

* Lua的C API中的lua_isstring和lua_isnumber有点坑

```
    LUA_API int lua_isnumber (lua_State *L, int idx) {
        lua_Number n;
        const TValue *o = index2addr(L, idx);
        return tonumber(o, &n);   // 坑，会执行转换成number类型改动栈
    }

    LUA_API int lua_isstring (lua_State *L, int idx) {
        const TValue *o = index2addr(L, idx);
        // 坑，这里逻辑是看能否转换成string，之后如果调用lua_tostring会改动栈中值，在遍历table判断key的时候就悲催了
        return (ttisstring(o) || cvt2str(o));  
    }

    -- 容易导致错误的使用比如
    lua_State *L = luaL_newstate();
    luaL_dostring(L, "a={};a[2]=123");
    lua_getglobal(L, "a");

    lua_pushnil(L);
    while(lua_next(L, -2))
    {
        auto is_string_key = lua_isstring(L, -2);
        if (lua_isstring(L, -2))
        {
            printf("%s=", luaL_checkstring(L, -2));
        }
        else if(lua_isinteger(L, -2))
        {
            printf("%d=", luaL_checkinteger(L, -2));
        }
        printf("%d\n", luaL_checkinteger(L, -1));
        lua_pop(L, 1);
    }

    lua_close(L);

    -- 以上这段代码会进程崩溃,因为错误的对int值进行了luaL_checkstring(或lua_tostring)导致当前lua虚拟堆栈被改写，然后lua_next的时候会找不到正确的key，然后报错崩溃
    -- 而下面这样写就正常了
    lua_State *L = luaL_newstate();
        
    luaL_dostring(L, "a={};a[2]=123");
    lua_getglobal(L, "a");

    lua_pushnil(L);
    while(lua_next(L, -2))
    {
        auto is_string_key = lua_isstring(L, -2);
        if(lua_isinteger(L, -2))
        {
            printf("%d=", luaL_checkinteger(L, -2));
        } 
        else if (lua_isstring(L, -2))
        {
            printf("%s=", luaL_checkstring(L, -2));
        }
        printf("%d\n", luaL_checkinteger(L, -1));
        lua_pop(L, 1);
    }

    lua_close(L);

    -- lua_isnumber也有类似问题，实现中是调用lua_tonumber看能否转换成number类型来判断，而这会改动lua虚拟堆栈结构，在遍历lua table的时候有问题

```

* Lua的C API中的lua_type函数，得到的类型可以用来判断布尔，nil，字符串类型,比如lua_type(L, -1) == LUA_TSTRING,但是不能用来直接判断整数，不能lua_type(L, -1) == LUA_TNUMINT  因为Lua 5.3中虽然区分了int和number类型，把整数和浮点数区分出来了，但是lua_type的返回类型都是LUA_TNUMBER，LUA_TNUMINT和LUA_TNUMFLT都是LUA_TNUMBER进行偏移后的结果。需要判断类型的时候整数需要使用lua_isinteger(L,-1)，浮点数需要使用lua_type(L, -1) == LUA_TNUMBER

* Lua的一些函数比如#，ipairs, table中一些函数，都是只对Lua table中的数组部分起效果，建议不要对同一个lua table同时使用数组部分和哈希表部分，可能容易出错。我们的做法是对Lua语言做了修改，增加了可选的类型声明和编译期静态类型系统并区分了Array<T>和Map<T>类型，减少混用导致的问题。

* Lua中的数学操作符，必须参数都是数字，注意使用时候不要错误使用了类型，包括nil也不行，比如+/-/*//还有其他一些数学操作符

* 注意区分lua_pcall/lua_pcallk和lua_call/lua_callk的使用区分，前者是runproteced模式下运行，也就是会捕获运行时异常并处理，不对使用者再直接抛出异常崩溃.但是也不要乱用lua_pcall，只在最外层使用

* 暂时就想到这些，以前可能也碰到一些其他问题没记录下来，以后我想起或者又碰到了再补充

