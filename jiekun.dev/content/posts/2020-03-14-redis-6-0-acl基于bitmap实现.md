---
title: Redis 6.0 ACL基于Bitmap实现
author: 小黄鸭
type: post
date: 2020-03-14T14:00:46+00:00
excerpt: Redis 6.0中使用Bitmap存储命令权限，List存储键匹配Pattern来实现ACL功能，并在执行命令、事务和Lua过程中判断权限。
featured_image: /wp-content/uploads/2020/03/redis_acl_cover.jpg
rank_math_primary_category:
  - 14
rank_math_seo_score:
  - 22
rank_math_internal_links_processed:
  - 1
categories:
  - 消息队列/中间件
tags:
  - ACL
  - Redis
  - 源码

---
Redis 6.0在4月30日就要和大家正式见面了，现在[redis.io][1]上已经提供了RC版本。在[之前的博客][2]中，已经介绍过权限控制新功能的一些用法，主要来源于作者Antirez在Redis Day上的一些演示。Antirez在最后提到，ACL的主要实现是基于Bitmap，因此对性能影响是可以忽略不计的。当时大致猜想了一下实现的思路，那么现在离发布已经很近了，作者也对ACL Logging进行了一些补充，不妨一起来看一下。

# user结构

server.h中定义了对应的`user`结构保存用户的ACL信息，包括：

  * 用户名
  * flag，主要是一些特殊状态，例如用户的启用与禁用、整体控制（所有命令可用与否、所有键可访问与否）、免密码等
  * 可用命令（allowed_commands），一个长整型数。每一位代表命令，如果用户允许使用这个命令则置位1
  * 可用子命令（allowed_subcommands），一个指针数组，值也为指针，数组与可用命令一一对应，值为一个SDS数组，SDS数组中存放的是这个命令可用的子命令
  * 用户密码
  * 可用的key patterns。如果这个字段为`NULL`，用户将不能使用任何Key，除非flag中指明特殊状态如`ALLKEYS`

```
typedef struct user {
    sds name;
    uint64_t flags;
    uint64_t allowed_commands[USER_COMMAND_BITS_COUNT/64];
    sds **allowed_subcommands;
    list *passwords;
    list *patterns;
} user;</code></pre>

```
补充一下一些新鲜的字段描述，`allowed_commands`实际上是一个（默认）长度为1024的位图，它的index对应各个命令的ID，在历史版本中命令结构`redisCommand`是通过名字（`name`）来查找的，`id`为这个版本中新增的属性，专门用于ACL功能。

```
truct redisCommand {
    ...
    int id;
};</code></pre>

```
`user`这个结构对应的是`client`结构的&#8221;user&#8221;字段，熟悉Redis的同学应该对`client`也有所了解，就不再赘述了。

# ACL操作选读

ACL的命令很多，总体而言都是围绕着`user`对象展开的，因此从中挑选了几个函数来看一下具体是如何操作`user`对象。

一个需要铺垫的通用方法就是`ACLGetUserCommandBit`，ACL操作中都会涉及到获取用户的命令位图，`ACLGetUserCommandBit()`接收一个`user`结构和命令ID，根据ID定位出命令在`allowed_commands`中的位置，通过位运算返回**用户是否有该命令权限**。

```
int ACLGetUserCommandBit(user *u, unsigned long id) {
    uint64_t word, bit;
    if (ACLGetCommandBitCoordinates(id,&word,&bit) == C_ERR) return 0;
    return (u->allowed_commands[word] & bit) != 0;
}</code></pre>

```
当用户进行Redis操作时，例如`set`操作，操作的命令会保存在`client`结构的`*cmd`字段中，`*cmd`字段就是一个`redisCommand`结构的指针，`redisCommand`结构包含了命令的`id`，因此在使用时通过`ACLGetUserCommandBit(u, cmd->id)`传入。

## 创建用户

创建用户分为两步，首先需要创建一个`user`，通过调用`ACLCreateUser(const char *name, size_t namelen)`实现，返回的是一个`user`对象的指针。在创建时，会在`server.h`定义的`Users`中查找是否有同名用户，也是本次功能新增的，因为旧版本中只有&#8221;default&#8221;用户。此时这个用户拥有名称，flag被初始化为禁用用户，其余的属性均为Null或空list等。

然后，通过调用`ACLSetUser(user *u, const char *op, ssize_t oplen)`，调整传入用户`u`的对应属性，调整内容放在名为`op`操作的参数中。这个函数非常长，主要是针对各种不同的“操作” switch case处理，节选部分如下：

```
int ACLSetUser(user *u, const char *op, ssize_t oplen) {
    if (oplen == -1) oplen = strlen(op);
    /* Part1 - 处理用户状态(flag)操作 */
    // 控制用户启用状态
    if (!strcasecmp(op,"on")) {
        u->flags |= USER_FLAG_ENABLED;
        u->flags &= ~USER_FLAG_DISABLED;
    } else if (!strcasecmp(op,"off")) {
        u->flags |= USER_FLAG_DISABLED;
        u->flags &= ~USER_FLAG_ENABLED;
    // 控制全局键、命令等可用与否
    } else if (!strcasecmp(op,"allkeys") ||
               !strcasecmp(op,"~*"))
    {
        u->flags |= USER_FLAG_ALLKEYS;
        listEmpty(u->patterns);
    }
    ...


    /* Part2 - 操作用户密码增删改查 */
    // > 和 &lt; 等控制密码的改动删除等
    else if (op[0] == '>' || op[0] == '#') {
        sds newpass;
        if (op[0] == '>') {
            newpass = ACLHashPassword((unsigned char*)op+1,oplen-1);
        }


    /* Part3 - 操作用户可用命令的范围 */
    else if (op[0] == '+' && op[1] != '@') {
        if (strchr(op,'|') == NULL) {
            if (ACLLookupCommand(op+1) == NULL) {
                errno = ENOENT;
                return C_ERR;
            }
            unsigned long id = ACLGetCommandID(op+1);
            // 根据传入的id参数设置对应allowed_commands位图的值
            ACLSetUserCommandBit(u,id,1);
            // 新调整的命令的子命令数组会被重置
            ACLResetSubcommandsForCommand(u,id);
        }
    }</code></pre>

```
补充一下具体调用例子，其实Redis的默认用户就是按照这套流程创建的：初始化名为“default”的空白无权限用户，然后为这个用户设置上所有权限：

```
DefaultUser = ACLCreateUser("default",7);
ACLSetUser(DefaultUser,"+@all",-1);
ACLSetUser(DefaultUser,"~*",-1);
ACLSetUser(DefaultUser,"on",-1);
ACLSetUser(DefaultUser,"nopass",-1);</code></pre>

```
## 拦截不可用命令/键

命令/键拦截操作非常简单：

  * 判断命令/键是否可用
      * 如果不可用，ACL Log处理以及返回错误

### ACL判断

我们先看一下“不可用”的判断逻辑，然后再回到命令执行流程中看判断方法的调用。

判断函数同样非常长，展示完后会进行总结：

```
int ACLCheckCommandPerm(client *c, int *keyidxptr) {
    user *u = c->user;
    uint64_t id = c->cmd->id;
    // 命令相关的全局flag的检查，若满足则跳过后续部分
    if (!(u->flags & USER_FLAG_ALLCOMMANDS) &&
        c->cmd->proc != authCommand)
    {
        // 即使当前命令没有在allowed_commands中，还要检查子命令是否可用
        // 以免出现仅开放了部分子命令权限的情况
        if (ACLGetUserCommandBit(u,id) == 0) {
            ...
            // 遍历子命令
            long subid = 0;
            while (1) {
                if (u->allowed_subcommands[id][subid] == NULL)
                    return ACL_DENIED_CMD;
                if (!strcasecmp(c->argv[1]->ptr,
                                u->allowed_subcommands[id][subid]))
                    break; // 子命令可用，跳出循环
                subid++;
            }
        }
    }

    // 键相关的全局flag检查，若满足则跳过后续部分
    if (!(c->user->flags & USER_FLAG_ALLKEYS) &&
        (c->cmd->getkeys_proc || c->cmd->firstkey))
    {
        int numkeys;
        // 先拿到当前要进行操作的Key
        int *keyidx = getKeysFromCommand(c->cmd,c->argv,c->argc,&numkeys);
        for (int j = 0; j &lt; numkeys; j++) {
            listIter li;
            listNode *ln;
            listRewind(u->patterns,&li);

            // 检查当前user所有的关于Key的匹配Pattern
            // 如果有任意命中则跳出，否则判定不可用
            int match = 0;
            while((ln = listNext(&li))) {
                sds pattern = listNodeValue(ln);
                size_t plen = sdslen(pattern);
                int idx = keyidx[j];
                if (stringmatchlen(pattern,plen,c->argv[idx]->ptr,
                                   sdslen(c->argv[idx]->ptr),0))
                {
                    match = 1;
                    break;
                }
            }
            if (!match) {
                if (keyidxptr) *keyidxptr = keyidx[j];
                getKeysFreeResult(keyidx);
                return ACL_DENIED_KEY;
            }
        }
        getKeysFreeResult(keyidx);
    }
    return ACL_OK;
}</code></pre>

```
那么为了方便喜欢跳过代码的同学看结论：

  * ACL限制围绕`user`的各个字段进行
  * 全局的flag优先级最高，例如设置为所有键可用，所有命令可用，会跳过后续的可用命令遍历和可用键Pattern匹配
  * 即使在allowed_commands位图中没有被置位，命令也可能可用，因为它是个子命令，而且命令只开放了部分子命令的使用权限
  * 键通过遍历所有定义了的Pattern检查，如果有匹配上说明可用
  * 先判断操作是否可用，再判断键（包括全局flag也在操作之后）是否可用，两种判断分别对应不同返回整数值：`ACL_DENIED_CMD`、`ACL_DENIED_KEY`

### 命令执行流程中的调用

判断逻辑之后到何时调用这套判断。我们先来复习一下Redis如何执行命令：

  * 用户操作
  * 客户端RESP协议（Redis 6.0中有RESP3新协议记得关注）压缩发送给服务端
  * 服务端解读消息，存放至`client`对象的对应字段中，例如`argc`、`argv`等存放命令和参数等内容
  * **执行前检查（各种执行条件）**
  * 执行命令
  * **执行后处理（慢查询日志、AOF等）**

目前执行命令的方法是在`server.c`中的`processCommand(client *c)`，传入`client`对象，执行，返回执行成功与否。我们节选其中关于ACL的部分如下：

```
int processCommand(client *c) {
    ...
    int acl_keypos;
    int acl_retval = ACLCheckCommandPerm(c,&acl_keypos);
    if (acl_retval != ACL_OK) {
        addACLLogEntry(c,acl_retval,acl_keypos,NULL);
        flagTransaction(c);
        if (acl_retval == ACL_DENIED_CMD)
            addReplyErrorFormat(c,
                "-NOPERM this user has no permissions to run "
                "the '%s' command or its subcommand", c->cmd->name);
        else
            addReplyErrorFormat(c,
                "-NOPERM this user has no permissions to access "
                "one of the keys used as arguments");
        return C_OK;
    }
    ...</code></pre>

```
在命令解析之后，真正执行之前，通过调用`ACLCheckCommandPerm`获取判断结果，如果判定不通过，进行以下操作：

  * 记录ACL不通过的日志，这个是作者在RC1之后新增的功能，还在Twitch上进行了直播开发，有兴趣的同学可以在Youtube上看到录播
  * 如果当前处于事务（MULTI）过程中，将client的`flag`置为`CLIENT_DIRTY_EXEC`
  * 根据命令还是键不可用，返回给客户端不同的信息

因此这次ACL功能影响的是执行命令前后的操作。

### 其他功能对ACL的调用

通过搜索可以发现一共有3处调用了`ACLCheckCommandPerm`方法：

```
/home/duck/study/redis/src/multi.c:
  179  
  180          int acl_keypos;
  181:         int acl_retval = ACLCheckCommandPerm(c,&acl_keypos);
  182          if (acl_retval != ACL_OK) {
  183              addACLLogEntry(c,acl_retval,acl_keypos,NULL);

/home/duck/study/redis/src/scripting.c:
  608      /* Check the ACLs. */
  609      int acl_keypos;
  610:     int acl_retval = ACLCheckCommandPerm(c,&acl_keypos);
  611      if (acl_retval != ACL_OK) {
  612          addACLLogEntry(c,acl_retval,acl_keypos,NULL);

/home/duck/study/redis/src/server.c:
 3394       * ACLs. */
 3395      int acl_keypos;
 3396:     int acl_retval = ACLCheckCommandPerm(c,&acl_keypos);
 3397      if (acl_retval != ACL_OK) {
 3398          addACLLogEntry(c,acl_retval,acl_keypos,NULL);</code></pre>

```
形式都是大同小异，了解一下即可。总结一下需要判定ACL的位置：

  * 正常命令执行流程中
  * MULTI事务执行过程中
  * Lua脚本

# 总结

补充一张图来描述新增的ACL功能相关的结构（点击放大查看）：

![]("./2020/03/Redis_ACL_user-1024x535.png)

图中部分的表达可能与实际的数据结构有所差异，主要原因是代码理解和C语言的语法掌握不到位所致。

阅读代码的过程中留意到，对命令的限制是通过Bitmap来实现的，而对Key的限制是通过特定Pattern来实现的。当对Key的限制Pattern数量特别多时，是否会因为匹配Pattern而对性能造成影响，例如超多次的`stringmatchlen()`执行。当然这一块内容似乎确实没有想到什么提升非常大的判断方式，后续也会继续关注ACL的相关改进。

 [1]: https://redis.io
 [2]: https://bytedance-hire.me/archives/235
 [3]: https://bytedance-hire.me/wp-content/uploads/2020/03/Redis_ACL_user.png