---
title: Testing Polly Nuget Package(It speaks by itself, just test the code)
categories:
- getting_started_with
tags:
- .Net
- Exceptions Policy
- Fluent
- Polly
---

[Polly](https://github.com/michael-wolfenden/Polly) is a .NET 3.5 / 4.0 / 4.5 / PCL (Profile 259) library that allows developers
to express transient exception handling policies such as Retry, Retry Forever, Wait and Retry or Circuit Breaker in a fluent manner.

Here I let you guys a self explained sample of how to use it.

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Polly;

namespace ConsoleApplication7
{
    class Program
    {
        private static int _loopCounter = 0;
        private static int _random = new Random().Next(2, 10);

        static void Main(string[] args)
        {
            Console.WriteLine("This app tests Polly Nuget Package");
            Console.WriteLine("\"https://www.nuget.org/packages/Polly/\")");
            Console.WriteLine("\nIt expresses transient exception handling policies such as Retry, ");
            Console.WriteLine("Retry Forever, Wait and Retry or Circuit Breaker in a fluent manner.");

            Console.WriteLine("\nThe example app tries to retrieve the current millisecond value ");
            Console.WriteLine("of the current time of day and if that value falls between 200 and 400");
            Console.WriteLine("it will raised an ArgumentOutOfRangeException.");
            Console.WriteLine("\nThe app will retry 300 times to retrieve a result");
            Console.WriteLine("before finally failing its purpose.\n ");
            try
            {
                var result = Policy.Handle()
                .Retry(300, (exception, retryCount, context) => Console.WriteLine("\n{0} times {1} raised with context info <{2}>", retryCount, exception.GetType().Name, context["info"]))
                .Execute(() => DoSomething(), new Dictionary<string, object>() { { "info", "context info" } });

                Console.WriteLine("\nResult is {0} after {1} iterations\n", result, _loopCounter);
            }
            catch (Exception e)
            {
                Console.WriteLine("\nThe application exceed the number of retries(300) to adquire a valid result\n");
            }

        }

        private static int DoSomething()
        {
            int result;
            // this is set to avoid an early application ends without retrying
            // this way it will always run at least a random number of times between 2 and 10
            result = _loopCounter < _random? 201 : DateTime.Now.Millisecond;             _loopCounter++;               if (result > 200 && result < 400)
            {
                throw new ArgumentOutOfRangeException();
            }
            return result;
        }
    }
}
```
