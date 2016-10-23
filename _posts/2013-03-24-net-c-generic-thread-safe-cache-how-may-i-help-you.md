---
title: .Net C# Generic Thread Safe Cache, how may I help you?
categories:
- ood
tags:
- .Net
- C#
- Cache
- Thread Safe
---

A few days ago I ran repeatedly in the need of using a cache abstract data structure for storing objects (.Net C# objects) with expensive creation process. So, I said to myself:

_“Myself, you need to make a generic cache abstract data structure you can use any time without BCL restrictions, and you will improve it as your needs over time dictate. In fact the first improvement dictated would be it must be thread safe, because many objects guys will want to flatter that cache girl.“_

, and after such revelation I put my fingers to work.

Was then when born the `ThreadSafeCache<TKey, TValue>` class with a static `Dictionary<TKey, TValue>` _cacheStore_ field acting as in memory cache store. There are two special thing to highlight in this code.

```csharp
public class ThreadSafeCache<TKey, TValue>: ICache<TKey, TValue>
{
    private static Dictionary<TKey, TValue> _cacheStore;

    public ThreadSafeCache()
    {
        _cacheStore = new Dictionary<TKey, TValue>();
    }

    public virtual TValue Get(TKey key)
    {
        lock (_cacheStore)
        {
            TValue value              
            if (_cacheStore.TryGetValue(key, out value))
            {
                return value;
            }
        }
        return default(TValue);
    }

    public virtual void Store(TKey key, TValue value)
    {
        lock (_cacheStore)
        {
            _cacheStore[key] = value;
        }
    }

    public virtual TValue this[TKey key]
    {
        get
        {
            return Get(key);
        }
        set
        {
            Store(key,value);
        }
    }

    public virtual TValue GetLazy(TKey key, Func<TValue> valueLazyInitializer)
    {
        lock (_cacheStore)
        {
            TValue value;
            if (_cacheStore.TryGetValue(key, out value))
            {
                return value;
            }
               
            value = valueLazyInitializer.Invoke();
            _cacheStore[key] = value;
            return value;
          }
    }

    public bool ContainsKey(TKey key)
    {
        return _cacheStore.ContainsKey(key);
    }   

    public bool ContainsValue(TValue value)
    {
        return _cacheStore.ContainsValue(value);
    }
}
```
The first is the logic of the Get and Store method:

```csharp
lock (_cacheStore)
{
      TValue value;
    if (_cacheStore.TryGetValue(key, out value))
    {
        return value;
    }
}
return default(TValue);

  // Store Logic
lock (_cacheStore)
{
    _cacheStore[key] = value;
}
```

The locking of the cache static field allows us to take  precise control over the scope and granularity of reading and storing operations. I choose this simple approach because thread safety is achieved primarily by locking to reduce the opportunities for thread interaction.

The second special things is this method called `GetLazy`, which is my favorite because it allows me to ask for an object with a 50 % possibilities of being previously cached, and it lets you provide a lambda for creating it(expensive process) if it wasn’t previously cached.

```csharp     
public virtual TValue GetLazy(TKey key, Func<TValue> valueLazyInitializer)
{
    lock (_cacheStore)
    {
        TValue value;
        if (_cacheStore.TryGetValue(key, out value))
        {
            return value;
        }

        value = valueLazyInitializer.Invoke();
        _cacheStore[key] = value;
        return value;
    }
}
```

This design may create a slight inefficiency while the entire cache would be locked up for the duration of the object creation operation. An alternative approach to solve this issue would be to deploy the locking in two moments, the first during the reading and the second during the storing, leaving out of lock the object creation operation and its expense of time (and processing capabilities). However, that way we are encouraging another performance inefficiency coming up if two(or more) threads simultaneously called this method looking (not locking) for the same uncached resource, the expensive object creation operation will be called twice(or more) and the Dictionary cache store will be updated unnecessarily.

Ending up, a little of advertising: I know it could be tricky to understand how to use the `GetLazy` method for some of you (I presented it to a folk and he was like: “_huhh?! It should be called GetOrSet_”), especially by the name and the lack of context. That’s why I‘d like to provide a practical real life example of something I’m working on In order to understand the ideal context for using `GetLazy`. Have a look at the next piece of code how it’s used to retrieve the correspondent `Mapper` for the given `TEntity_Type`.

![Image]({{ site.baseurl }}/images/po.png){:class="img-responsive"}

I hope this would be useful at any level. See you in the next one.
