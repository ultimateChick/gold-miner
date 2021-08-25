> * 原文地址：[Advanced Python: How To Implement Caching In Python Application](https://medium.com/fintechexplained/advanced-python-how-to-implement-caching-in-python-application-9d0a4136b845)
> * 原文作者：[Farhad Malik](https://medium.com/@farhadmalik)
> * 译文出自：[掘金翻译计划](https://github.com/xitu/gold-miner)
> * 本文永久链接：[https://github.com/xitu/gold-miner/blob/master/article/2021/advanced-python-how-to-implement-caching-in-python-application.md](https://github.com/xitu/gold-miner/blob/master/article/2021/advanced-python-how-to-implement-caching-in-python-application.md)
> * 译者：
> * 校对者：

# Advanced Python: How To Implement Caching In Python Application

Caching is an important concept to understand for every Python programmer.

In a nutshell, the concept of caching revolves around utilising programming techniques to store data in a temporary location instead of retrieving it from the source each time.

Subsequently, caching can provide an application performance boost as it is faster to access data from the temporary location than it is to fetch the data from the source each time, such as from database, web service, etc.

This article aims to explain how caching works in Python.

![](https://miro.medium.com/max/671/1*4f9xjNXx1FVXzIoQZ4CdpQ.png)

This is an advanced level topic for Python developers and I recommend it to everyone who is/or intends in using the Python programming language.

If you want to understand the Python programming language from the beginner to an advanced level then I highly recommend [**this article**](https://medium.com/fintechexplained/everything-about-python-from-beginner-to-advance-level-227d52ef32d2).

## Article Aim

In this article, I will explain what caching is, why we need caching and how we can build caching in a Python application.

> Caching is required when we want to improve the performance of our application.

I will outline the following three key points:

1. What And Why Do We Need To Implement Caching?
2. What Are The Rules Of Caching?
3. How To Implement Caching?

I will start by explaining what caching is, why we need to introduce caching in our applications, and how to implement caching.

## 1. What And Why Do We Need To Implement Caching?

To understand what caching is and why we need caching, consider the scenario below.

- We are building an application in Python that will display the list of products to the end-users.
- This application is going to be accessed multiple times on daily basis by over 100 users
- The application is going to be hosted on an application server and it will be accessible on the internet
- The products are going to be stored in a database that will be installed on a database server.
- Therefore the application server will query the database to fetch the relevant records.

The image below demonstrates how our target application is set up:

![](https://miro.medium.com/max/700/1*XcWM5A0G6yKWqr_yWUfgnw.png)

Image illustrating how the application server gets data from a database server

### Issue

Fetching the data from a database is an IO-bound operation. Hence it is slow by nature. If the request is sent frequently and the response is not updated as often then we can cache the response within the memory of the application.

Instead of querying the database each time, we can cache the results as shown below:

![Data is stored in the temporary Cache](https://miro.medium.com/max/700/1*3mv7SdqieweoDQYEhkNAhA.png)

The request to get the data has to go over the wire and the response has to come back over the wire.

This is slow in nature. Therefore, a cache is introduced.

> We can cache the results to reduce the computation time and save computer resources.

The cache is a temporary storage location. It works in a lazy load manner.

Initially, the cache is empty. When the application server gets the data from the database server, it populates the cache with the required data set. From then on, the subsequent requests get the data from the cache instead of going all the way to the application server.

We also need to invalidate the cache on timely basis to ensure we show up-to-dated information to the end-users.

This brings us to the next section of the article: The rules of caching.

## 2. Rules Of Caching

In my opinion, there are three rules of caching.

Before caching is enabled, we need to perform the key step of profiling the application.

Therefore the first step before introducing caching in an application is to profile the application.

Only then we can understand how long each function takes and how many times it is getting called.

I have explained the art of profiling in [**this article**](https://medium.com/fintechexplained/advanced-python-learn-how-to-profile-python-code-1068055460f9) and I highly recommend it to everyone.

Once the process of profiling is complete, we need to determine what we need to cache.

We need a mechanism to link the inputs to the outputs of a function and store them in memory. This brings us to the first rule of caching.

### 2.1. First Rule Of Caching:

The first rule is to ensure that the target function does take a long time to return the output, it is getting executed frequently and the output of the function does not change as often.

We do not want to introduce caching for functions that do not take a long time to complete or functions that are hardly getting called in the application or those functions that are returning results which are changing frequently in the source.

This is an important rule to remember.

> Suitable Candidate For Caching = Function Frequently Called, Output Not Changing As Often And It Takes A Long Time To Execute

As an instance, if a function is being executed 100 times, and the function takes a long time to return the results and it returns the same results for the given inputs then we can cache the results.

However, if the value to be returned by a function is updated every second in the source and we get a request to execute the function every minute then it's really important to understand whether we need to cache the results as we might end up sending stale data to the users. This can help us understand whether we need caching or if we need a different communication channel, data structures or serialising mechanism to retrieve data faster, such as sending data by using binary serialiser over sockets vs xml serialisation over http.

Additionally, it's important to know when to invalidate and re-load the cache with the fresh data.

### 2.2. Second Rule:

The second rule is to ensure that it is faster to get the data from the introduced caching mechanism than it takes to execute the target function.

We should only introduce caching if the time it takes to retrieve the results from the cache is faster than it takes to retrieve the data from its source.

> Caching Should Be Faster Than Getting The Data From The Current Data Source

Therefore it is vital to choose the appropriate data structures such as a dictionary or LRU cache as an instance.

### 2.3. Third Rule:

The third important rule is about memory footprint which is often ignored. Are you performing IO operations such as querying databases, web services, or are you performing CPU intensive operations such as crunching the numbers and performing in-memory calculations?

The memory footprint of the application will increase when we cache the results therefore it's crucial to choose the appropriate data structures and only cache the attributes of the data that are required to be cached.

Sometimes we query multiple tables to create an object of a class. However, we only need to cache the basic attributes in our application.

> Caching Impacts Memory Footprint

As an instance, consider that we built a reporting dashboard that queries the database and retrieves a list of orders. For the sake of illustration, let's also consider that only the order name is displayed on the dashboard.

We can, therefore, cache only the name of each order instead of caching the entire order object. Usually, the architects suggest creating a lean data transfer object (DTO) that has `__slots__` attribute to reduce the memory footprint. Named tuples or Python data classes are also used.

This brings us to the last section of the article outlining the details of how caching can be implemented.

## 3. How To Implement Caching?

There are multiple ways to implement caching.

We can create local data structures in our Python processes to build the cache or host the cache as a server that acts as a proxy and serves the requests.

There are built-in Python tools such as using cached_property decorator from functools library. I want to introduce the implementation of caching by providing an overview of the cached decorator property.

The code snippet below illustrates how cached property works.

```python
from functools import cached_property
class FinTech:
  
  @cached_property
  def run(self):
     return list(range(1,100))
```

As a result, the FinTech().run is now cached and the output of range(1,100) will only be generated once. However, we hardly need to cache the properties in real-case scenarios.

Let's review other approaches.

### 3.1. Dictionary Approach

For simplistic use cases, we can create/use a mapping data structure such as a dictionary that we can keep in memory and make it accessible on the global frame.

There are multiple ways to implement it. The simplest approach is to create a singleton-style module e.g. config.py

In the config.py, we can create a field of type dictionary that is populated once at the start.

From then on, the dictionary field can be used to get the results.

As an instance, consider this code.

This is the config.py with a property: cache

```python
cache = {}
```

Let's consider that our application attempts to query Yahoo finance web service to get the historic prices of a company via a function: `get_prices(symbol, start_date, end_date)`

The historical prices are not going to change therefore there is no need in querying the web service every time we need historical prices. We can cache the prices in memory.

Internally, the function `get_prices(symbol, start_date, end_date)`, can check whether the data is in the cache before it attempts to return the results.

Let me explain the strategy via code.

The function below `get_prices `accepts a parameter named companies.

1. Firstly, the function creates a start and end date variable where the start date is set to yesterday and the end date is set to 11 days before yesterday
2. Then it creates a variable named target_key of type tuple. It is a unique value that is the composite name of the module, function, start and end date
3. The function first looks for the key in config.cache. If it can find the key then it checks if the cache contains the target company name
4. If the cache does contain the company name then it returns the prices from the cache. If the cache does not contain the company name then it queries the yahoo finance library and gets the prices. It then saves the prices in the cache for future calls.
5. If the target_key is not in the cache then it queries yahoo finance for all of the companies and saves the prices in the cache for future calls. Finally, it returns the prices.

Therefore, the fundamental rule is to check for the target key in the cache, if it is not there then get it from its source and save it in the cache before returning it.

This is how the cache can be built.

```python
def get_prices(companies):
    end_date = datetime.datetime.now()- datetime.timedelta(days=1)
    start_date = end_date - datetime.timedelta(days=11)
    target_key = (__name__, '_get_prices_’, 
                   start_date.strftime("%Y-%m-%d"),
      end_date.strftime("%Y-%m-%d")) 

    if target_key in config.cache:
        cached_prices = config.cache[target_key]
        #cached_prices is a dictionary where 
         #key = company symbol and value = prices
        prices = {}
for company in companies:
            # for each company, only get the prices 
            # that are not cached
            if company in cached_prices:
                # we have cached the price for the symbol
                # set prices in a local variable
                prices[company] = cached_prices[company]
            else:
                # go to yahoo finance and get the price
                yahoo_prices = yf.download(company, 
              start=start_date, end=end_date)
                prices[company] = yahoo_prices['Close']
                cached_prices[company] = prices[company]
        return prices
    else:
        company_symbols = ' '.join(map(lambda x: x, companies))
        yahoo_prices = yf.download(company_symbols, start=start_date, end=end_date)
        # now store it in cache for future
        prices_per_company = yahoo_prices['Close'].to_dict()
        config.cache[target_key] = prices_per_company
        return prices_per_company
```

Note that the chosen key contains the start and end date. I could also include the company name in the key so that we store (company name, start, end, function name) as the key.

The key to the data structure needs to be unique. It can also be a tuple.

The historical prices will not change therefore it is safe to not build the logic to invalidate the cache on a timely basis.

This cache is a nested dictionary where the value is also a dictionary. It provides faster lookups as the time complexity of a lookup operation is O(1).

[Here is a great article](https://medium.com/fintechexplained/time-complexities-of-python-data-structures-ddb7503790ef) that explains time complexities of Python data structures in an easy to understand manner.

Sometimes the key will get too long, and it's preferred to hash it using MD5, SHA, etc. However, with hashing, we might get a collision whereby two strings yield the same hash.

We can also utilise the technique of memoisation.

Memoisation is usually used in recursive function calls where the intermediate results are stored in memory and are returned when they are required.

This is where I will introduce the LRU.

### 3.2. Least Recently Used Algorithm

We could use the in-built feature of Python called LRU.

LRU stands for the least recently used algorithm. LRU can cache the return values of a function that are dependent on the arguments that have been passed to the function.

LRU is particularly useful in recursive CPU bound operations.

It is essentially a decorator: [@lru_cache](http://twitter.com/lru_cache)(maxsize, typed) which we can decorate our functions with.

- `maxsize` tells the decorator the maximum size of the cache. If we don't want to set the size then simply set it to None.
- `typed` is used to indicate whether we want to cache the output as the same values that can be compared values of different types.

This works when we expect the same output for the same inputs.

Holding all of the data in your application's memory can be troubling.

This can become a problem in a distributed application with multiple processes where it is not appropriate to cache all of the results in memory in all of the processes.

A good use case is when the application runs on a cluster of machines. We can host caching as a service.

### 3.3. Caching As A Service

The third option is about hosting cached data as an external service. This service can be responsible for storing all of the requests and responses.
All of the applications can retrieve data via the caching service. It acts as a proxy.

Let's consider that we are building an application as big as Wikipedia and it is expected to service 1000s of requests concurrently and in parallel.

We need a caching mechanism and want to distribute the caching across servers.

We can use memcache and cache the data.

Memcached is highly popular in Linux and Windows because:

- It can be used for implementing memoisation caches that have states.
- It can even be distributed across servers.
- It is extremely simple to use, it's fast and it is being used across industry in multiple large organisations.
- It supports auto-expiring the cached data

There is a python library called `pymemcache `which we need to install.

Memcache requires the data to be either stored as strings or binary. Hence, we would have to serialise the cached objects and deserialise them when we want to retrieve them.

The code snippet shows how we can launch and use the memcache:

```python
client = Client(host, serialiser, deserialiser)
client.set(‘blog’: {‘name’:’caching’, ‘publication’:’fintechexplained’}}
blog = client.get(‘blog’)
```

### 3.4. Gotcha: Invalidating Cache

Lastly, I wanted to quickly provide an overview of the scenarios when the output of a function for the same inputs is changing on timely basis and we want to cache the results for a shorter period of time.

Consider the case whereby two applications are running on two different application servers.

The first application gets the data from the database server and the second application updates the data in the database server. The data is fetched frequently and we want to cache the data in the first application server:

![](https://miro.medium.com/max/700/1*2gi4DqkHWejxgU7sSRoDGQ.png)

There are a few ways to solve it.

- The application in the second application server can notify the first application server whenever new records are stored so that it can refresh its cache. It can post a message to a queue that the first application can subscribe to.
- The first application server can also make a light-weight call to the database server to find the last time when the data was updated and it can then use the time to determine whether it needs to refresh the cache or get the data from the cache.
- We can also add a timeout that will clear the cache so that the next request can reload it. It is easy to implement but it is not as reliable as the last options I have explained above. This feature can be achieved via signaling library whereby we can subscribe a handler to the signal.alarm(timeout) and after a timeout period is called, we can clear the cache in the handler. We can also run a background thread to invalidate the cache, it's, however, important to ensure appropriate synchronisation objects are used.

## Summary

Caching is an important concept to understand for every Python programmer and data scientist.

> 如果发现译文存在错误或其他需要改进的地方，欢迎到 [掘金翻译计划](https://github.com/xitu/gold-miner) 对译文进行修改并 PR，也可获得相应奖励积分。文章开头的 **本文永久链接** 即为本文在 GitHub 上的 MarkDown 链接。

---

> [掘金翻译计划](https://github.com/xitu/gold-miner) 是一个翻译优质互联网技术文章的社区，文章来源为 [掘金](https://juejin.im) 上的英文分享文章。内容覆盖 [Android](https://github.com/xitu/gold-miner#android)、[iOS](https://github.com/xitu/gold-miner#ios)、[前端](https://github.com/xitu/gold-miner#前端)、[后端](https://github.com/xitu/gold-miner#后端)、[区块链](https://github.com/xitu/gold-miner#区块链)、[产品](https://github.com/xitu/gold-miner#产品)、[设计](https://github.com/xitu/gold-miner#设计)、[人工智能](https://github.com/xitu/gold-miner#人工智能)等领域，想要查看更多优质译文请持续关注 [掘金翻译计划](https://github.com/xitu/gold-miner)、[官方微博](http://weibo.com/juejinfanyi)、[知乎专栏](https://zhuanlan.zhihu.com/juejinfanyi)。