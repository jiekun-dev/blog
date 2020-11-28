---
title: 基于Redis的缓存装饰器及缓存预热设计
author: 小黄鸭
type: post
date: 2019-09-08T05:27:22+00:00
excerpt: 实现一个缓存装饰器，基于传入函数的参数控制缓存是否生效，并实现缓存的分批预热
featured_image: /wp-content/uploads/2019/09/cache-warmer-logo.jpg
rank_math_primary_category:
  - 14
rank_math_internal_links_processed:
  - 1
categories:
  - 消息队列/中间件
tags:
  - Redis
  - 缓存
  - 预热
archieved: Arc'd

---
## 业务背景

业务上使用的是类似MVC的架构，具体而言通过view层控制接口，logic层控制业务逻辑，models模型映射数据库。在logic层，使用了Redis缓存装饰器对满足参数要求的方法执行结果进行缓存，降低复杂逻辑页面、数据量大页面的请求执行耗时。

对于这种使用场景，当缓存失效时，因为缓存使用的位置都是复杂逻辑，再次生成缓存需要数据库层面执行，如果这一步是由用户触发（也就是cache失效后第一位用户请求的时候生成缓存），那对这位用户来说请求时间就会很长甚至504超时。因此，有必要设计缓存的自动预热以及手动预热，前者是为了避免由用户触发生成缓存的长时间等待，后者是为了特殊时候（如数据更新后，缓存还没到期）手动更新缓存。

## 原有的缓存装饰器

```
f redis_cache(ttl=None, cache_name=None, **kwargs):
    """
    Redis缓存装饰器 用于Logics层函数执行结果缓存
    全局配置: config/localsettings.py REDIS_ON = True
    使用示例和说明：

    @redis_cache(ttl=3600, cache_name="", arg1=1, arg2=20)
    def sample_method(arg1, arg2, arg3, arg4, arg5)

    ttl: 缓存失效3600秒
    cache_name: 函数功能名
    kwargs：参数中对应满足kwargs时进行缓存，示例中当函数参数包含args=1，args2=20时缓存
    缓存规则：logic_cache:function_name:参数作为键名进行缓存，示例：
    logic_cache:sample_method:arg1:1:arg2:20:arg3:rank:arg4:people:arg5:food

    :param ttl: 过期时间
    :param cache_name: 函数功能名 用于清理缓存时显示 方便非技术同时使用
    :param kwargs: 命中何种参数时进行缓存 示例 **{'arg1': 1, 'arg2': 20}
    :return: 返回函数执行结果 / 缓存结果
    """
    def decorator(func):
        cache_serv = cache

        @wraps(func)
        def returned_wrapper(*args, **innerkwargs):
            try:
                func_name = func.func_name
                default_kwargs = kwargs
                is_cache_target = cache_target_judger(default_kwargs, innerkwargs)
                # Cache Target
                if REDIS_ON and is_cache_target:
                    colon = ':'
                    redis_key_prefix_list = ['logic_cache', func_name]
                    for each in innerkwargs:
                        redis_key_prefix_list.append(each + ':' + str(innerkwargs[each]))
                    redis_key_str = colon.join(redis_key_prefix_list)

                    result = cache_serv.get(redis_key_str)
                    # Cache Exist:
                    if result is not None:
                        print(func_name + ': This is Cache')
                        return result
                    # Cache Not Exist / Force Update
                    else:
                        cache_data = func(*args, **innerkwargs)
                        cache = cache_serv.set(redis_key_str, cache_data, ttl)
                        if cache_name is not None:
                            redis_key_name_prefix_list = ['cache_name', func_name]
                            redis_key_name_str = ':'.join(redis_key_name_prefix_list)
                            cache = cache_serv.set(redis_key_name_str, cache_name, ttl)
                        print(func_name + ': Add to Cache now')
                        return cache_data
                # SQL/ES Target
                else:
                    direct_run_data = func(*args, **innerkwargs)
                    print(func_name + ': Not Cache Target')
                    return direct_run_data
            except Exception as e:
                logging.error("Redis decorator error: {}".format(e))
                direct_run_data = func(*args, **innerkwargs)
                print('Error Happened')
                return direct_run_data

        return returned_wrapper

    return decorator


def cache_target_judger(default_kwargs, innerkwargs):
    """
    缓存判断 返回Bool值
    判断是否为缓存目标
    :param default_kwargs:
    :param innerkwargs:
    :return:
    """
    return all(item in innerkwargs.iteritems() for item in default_kwargs.iteritems())

```
先来说明一下原有缓存的逻辑，其实实现非常简单：

对于如下使用：

```
@redis_cache(ttl=3600, cache_name='示例功能', arg1=1, arg2=20)
def sample_method(arg1, arg2, arg3, arg4):
        pass

sample_method(arg1=1, arg2=20, arg3='hello', arg4='world')

```
#### 判定缓存目标

在调用sample_method时，传入参数`arg1=1, arg2=20, arg3='hello', arg4='world'`，也就是`returned_wrapper(*args, **innerkwargs)`中的innerkwargs，代表方法的参数（字典形式）。  
在使用装饰器的时候，装饰器参数ttl, cache_name, 后面的都是kwargs，代表装饰器的参数（字典形式）。

现在将kwargs和innerkwargs比较，如果kwargs是innerkwargs的子集，也就是说innerkwargs满足kwargs的约束，说明这是一个要缓存的目标。以示例说明，redis_cache中kwargs是:`{'arg1': 1, 'arg2': 20}`，sample_method被捕获到的innerkwargs是`{'arg1':1, 'arg2': 20, 'arg3':  'hello', 'arg4': 'world'}`。因为满足了kwargs要求的arg1=1和arg2=20，因此这个sample_method的请求是需要被缓存的。

如果在这一步中判断不是要被缓存的那说明Redis中必然不存在这个缓存，则直接执行方法获返回结果即可。

#### 生成缓存键

显然，每个不同的函数调用的参数都不一样，我们需要使用当前调用的全部参数作为Redis键名以避免重复。  
设计键名为：

```
前缀 + 方法名 + 方法参数键值对

```
不使用hash是因为前期缺少cache_name注释的情况下无法通过redis键名（如果是hash）判断出这个缓存对应的功能。  
这里也会有小问题，如果方法名一样的方法在不同位置有出现的话也可能让键名重复。

#### 获取/生成缓存

有了缓存的键名之后，就可以到Redis中查看这个键是否存在，因此尝试取出这个键：  
`result = cache_serv.get(redis_key_str)`

  * 如果result不存在，说明缓存不存在，对本次调用需要执行方法，并将结果set进Redis中，并返回执行结果。
  * 如果result存在，说明缓存存在，对本次调用可以返回缓存结果。

#### 小结

通过上述的方法，已经可以实现一个缓存的装饰器，并且控制方法参数级别的缓存判定。

## 缓存系统改造

#### 允许更新缓存

旧方案的缓存生命周期是从第一次触发生成时刻至缓存过期时刻。对于这种对方法的缓存，要达到预热（更新）的效果，其实只要在判定上作小调整：  
在生成了缓存键名之后，原思路是看是否有result，有则取缓存，无则生成缓存。这里只需要改造成：

  * 当需要无视规则强制生成缓存的时候，执行函数并生成缓存。

这个改动非常简单，在调用方法时，传入一个force_update参数：

```
@redis_cache(ttl=3600, cache_name='示例功能', arg1=1, arg2=20)
def sample_method(arg1, arg2, arg3, arg4):
        pass

sample_method(arg1=1, arg2=20, arg3='hello', arg4='world', force_update=True)

```
  * 这个force_update要在装饰器执行func之前处理，因为`sampe_method`的定义中没有这个参数，因此当执行func的时候传的参数必须仍然只有前面几个。因此装饰器中加上：

```
force_update = innerkwargs.pop('force_update', False)

```
  * 正常调用不会包含这个方法，但是仍然会经过装饰器，因此默认False
  * 使用pop方法保证这个参数不会进入实际执行方法的过程中

有了force_update标志之后，对是否需要执行方法并写入缓存的判定就要修改一下：

  * 从`if result is not None:`改为`if result is not None and not force_update:`

这样当有force_update的时候也会继续执行方法和写缓存的操作。

#### 缓存定时预热

既然需要实现定时更新，那只要定时执行一下sample\_method，并且加上force\_update参数强制使它更新就行。

因此是否可以做一个定时脚本：

```
* */2 * * * python update_cache.py

```
update_cache.py内容如下：

```
if __name__ == '__main__':
    sample_method1(arg1=1, arg2=2, arg3='hello', arg4='world', force_update=True)
    sample_method2(arg1=2, arg2=2, arg3='hello', arg4='world', force_update=True)
    sample_method3(arg1=3, arg2=2, arg3='hello', arg4='world', force_update=True)
    sample_method4(arg1=4, arg2=2, arg3='hello', arg4='world', force_update=True)

```
这样就实现了缓存的定时更新

#### 缓存手动预热

思路：手动预热只需要将crontab定时改为（支持）手动触发即可，具体可以设计成一个调用接口。

抱歉，如果是这样设计，这套缓存系统就会面临类似缓存穿透的问题。

缓存穿透，故名思意就是请求不经过缓存，直接打在数据库上执行，在特定期间如果有大量的请求经数据库执行会给数据库造成很大压力。在我们的业务中，使用缓存的地方都是原来较为缓慢的查询，如果让大量请求同时执行SQL的话对MySQL负载时比较高的。

设想如果手动预热做成接口，意味着这个接口对应的sample_method的执行是不经过缓存层的而是直接在数据库执行，如果这个接口同一时间有多人调用的话，就会在SQL同时执行多个复杂查询，影响性能。

因此，为了让缓存手动预热对数据库更加友好，这里需要将预热改造成队列形式。对于这种的业务，不需要保证可靠性，也没有性能要求，因此无需使用MQ，直接借助Redis的LIST类型和lpush+rpop可以做一个非常简单的队列。

  * 在手动预热请求的logic层：

```
# 要使用集合常量来避免不合法的传参
WARMER_DICT = {
    'sample_method_collection'
}

def submit_warmer(warmer):
    """
    :return:
    """
    if warmer in WARMER_DICT:
        warmer_key = CACHE_WARMER_KEY
        redis_raw_serv.lpush(warmer_key, warmer)
    return 200, 0, "加入预热队列成功", {}

```
  * 在后台维护一个持续监控队列的进程

```
# 同样使用字典来避免不合法的队列元素
CACHE_FUNC = {
    'sample_method_collection': sample_method_collection
}


def sample_method_collection():
    sample_method1(arg1=1, arg2=2, arg3='hello', arg4='world', force_update=True)
    sample_method2(arg1=2, arg2=2, arg3='hello', arg4='world', force_update=True)
    sample_method3(arg1=3, arg2=2, arg3='hello', arg4='world', force_update=True)
    sample_method4(arg1=4, arg2=2, arg3='hello', arg4='world', force_update=True)

def warmer_server():
    """
    缓存更新后台进程
    不断读取redis中的LIST队列，如有则rpop出任务执行
    :return:
    """
    while True:
        cache_warmer_key = CACHE_WARMER_KEY
        try:
            func_name = redis_raw_serv.rpop(cache_warmer_key)
            if func_name not in CACHE_FUNC:
                raise KeyError
            print('Found warmer job %s at %s.' % (func_name, datetime.datetime.now().strftime('%m-%d %H:%M:%S')))
            # 执行一个方法来触发对应（或多个）带force_update参数的方法
            CACHE_FUNC[func_name]()
        except KeyError:
            pass
        print('Interval 5 seconds.')
        time.sleep(5)

```
## 总结

新完善的这套缓存方案实现了：

  * 方法参数粒度的缓存控制
  * 缓存的自动、手动预热

基于这些改造：

  * 通过缓存的自动预热，避免了缓存雪崩的问题；
  * 通过缓存手动预热（实际上自动预热也同样）使用队列，避免预热过程中给数据库带来过高负载。

Python的装饰器可以应用于MVC的各层，实现接口（View）层面的缓存同样可以使用（粗粒度的控制），但是因为Django中view层的参数为request对象，在使用中需要再进行改造，对request进行解析获取具体参数。

对于方法重名问题，实际上也可以将Redis键改造成`前缀:模块名:view层方法名:logic层方法名:参数字典`的形式，抽象来说只要能在`前缀`和`参数字典`之间能够定位到准确的使用装饰器的位置即可。

## 持续优化

#### 更简便的缓存预热目标生成

目前对于需要预热的目标方法，都是在crontab脚本中显式执行的，这样每当需要添加一个定期预热的目标方法，都要在脚本内添加代码。  
优化方案：额外维护一个动态的预热的目标队列，队列是由第一次生成缓存的时候判断这个方法后续是否需要自动预热，若需要，则加入预热队列中并定期执行，执行的时候又加入下一次预热队列中。这样只需要在缓存装饰器的使用中声明`redis_cache(ttl=3600, cache_name='示例', auto_update=True, args1=1, args2=20)`即可。

#### 预热参数组合

在其他业务有使用到，举例如args1可以为1或2或3， args2可以为20或30，这时候对于方法的缓存预热，需要对参数组合（1，20），（2，20），（3，20）和（1，30），（2，30），（3，30）共计6种。  
优化方案：这时候可能需要将面向业务层的缓存装饰器和面向预热的缓存装饰器分离开，因为业务层只对特定一次请求负责，也就是说用户只可能传6种请求的1种，服务只需要将这1种的结果，不管是缓存，还是生成缓存，处理好并返回即可； 对面向预热的装饰器，需要实现支持列表传参，生成参数组合，并逐一执行，调用示例：  
`redis_cache(ttl=3600, cache_name='示例', args1=[1, 2], args2=[20, 30])`  
在其他业务中，也有实现过组合参数的方法，如：

```
f param_combination_generator(kwargs):
    """
    参数排列组合生成器
    根据dict生成各个key名对应多个可能值的组合情况
    如{'key1':['value1', 'value2'], 'key2':['value3', 'value4']}可以有4种组合
    :param kwargs:
    :return: 返回组合list, 如[((page_num, 1)(sort_type, like_count)), ((page_num, 2)(sort_type, like_count))]
    """
    pre_dict = {}
    for k, v in kwargs.iteritems():
        pre_dict[k] = []
        for each_value in v:
            pre_dict[k].append((k, each_value))
    iter_list = []
    for each in pre_dict.values():
        iter_list.append(each)

    param_combination = list(itertools.product(*iter_list))

    return param_combination

```
尽管还需要优化，不过大体思路按照排列组合后的参数对函数逐一放入队列执行应该可以较为友善地实现。当参数排列组合特别多的时候，可能就不太适合缓存的场景，因为cache成本都是非常高昂的，可能需要优化成针对最热点的内容优先缓存的形式。