---
title: My baby steps with System.Reflection.Emit
categories:
- hacks
tags:
- .Net
- IL
- IL rewriting
- Reflection Emit
---

Just a couple of days ago I was implementing a Generic Repository Pattern with Unit of Work for Entity Framework(which I'll share with in a close future). It all went well until I ran into a little difficult. My Repository concrete class took an extra Type parameter to refer the DbContext.

```csharp
public class Repository<TKey, T, TDbContext> : IRepository<TKey, T>
    where T : class, IEntityKey
    where TDbContext : DbContext, IContextDbSetWithEntityKeyResolver
{
    ...
}
```

At first glance, this way looks the most elegant, but it also turns impossible for the DI Container to resolve the three type parameters concrete class from the two types parameters service interface(at least for [Ninject](http://www.ninject.org "Ninject") , and I was not in the mood to try another DI Container , although I definitely have to research if another one cans). The only possible solution would be to create a wrapper inheritor class to hide the third type parameter by hardcoding it.

```csharp
public class RepositoryFaker<TKey, T>: Repository<TKey, T, DomainContext> where T : class, IEntityKey
{
    public RepositoryFaker(IEFUnitOfWork unitOfWork) : base(unitOfWork)
    {
    }
}
```

But that’s really ugly, unpractical, and it's a huge coupling factoring in its best times. So I said, what if a I can generate this wrapper inheritor class on the fly?...and I decided to give it a try to System.Reflection.Emit. So I took a helmet, just for listening to Phil Haack about beating your head with a hammer while about rewriting IL as he says in this [post](http://haacked.com/archive/2013/01/05/mitigate-the-billion-dollar-mistake-with-aspects.aspx "Mitigate The Billion Dollar Mistake with Aspects by Phil Haack").  
In order to dynamically generate a Type using System.Reflection.Emit First you must create a ModuleBuilder to dynamically build the assembly that will hold the dynamically created type:

```csharp
...

// Defining general assembly properties
AssemblyName assemblyName = new AssemblyName();
assemblyName.Name = "RepositoryFakerAssembly";
assemblyName.Version = new Version("1.0.0.0");
// Creating an assembly builder within the current AppDomain and just for runtime
AssemblyBuilder assemblyBuilder = AppDomain.CurrentDomain.DefineDynamicAssembly(assemblyName, AssemblyBuilderAccess.Run);
// Creating Module Builder
ModuleBuilder moduleBuilder = assemblyBuilder.DefineDynamicModule("RepositoryFakerAssembly");

...
```

Once done this, you should prepare the oven for baking the type:

```csharp
...

// Defining a type builder for a public class named "RepositoryFaker".
TypeBuilder repositoryFakerBuilder = GetFakeRepositoryModuleBuilder()
                                        .DefineType("RepositoryFakerAssembly.RepositoryFaker", TypeAttributes.Public);

// Defining type parameters
// .. <TKey, T>
GenericTypeParameterBuilder[] genericParamsBuilders = repositoryFakerBuilder.DefineGenericParameters(new string[]{"TKey","T"});

// Defining clause
// .. where T : class...
genericParamsBuilders[1].SetGenericParameterAttributes(GenericParameterAttributes.ReferenceTypeConstraint);

// Defining clause
// .. where T : ...,  IEntityKey
genericParamsBuilders[1]
    .SetInterfaceConstraints(new Type[]{typeof(IEntityKey<>)
      .MakeGenericType(new Type[]{repositoryFakerBuilder.GenericTypeParameters[0]})});

// Defining inherited class with their type params
// .. :Repository<TKey, T, DomainContext>
repositoryFakerBuilder
    .SetParent(typeof(Repository<,,>)
        .MakeGenericType(new Type[]
        {
          repositoryFakerBuilder.GenericTypeParameters[0], repositoryFakerBuilder.GenericTypeParameters[1], typeof(TDbContext)
        }
    ));

// Defining constructors parameters
// .. (IEFUnitOfWork unitOfWork)
Type[] constructorArgs = new Type[1];
constructorArgs[0] = typeof(IEFUnitOfWork<>).MakeGenericType(typeof(TDbContext));

// Definining constructor builder and passing it the params
// public RepositoryFaker(IEFUnitOfWork unitOfWork) : base(unitOfWork)
ConstructorBuilder constructor = repositoryFakerBuilder
                                    .DefineConstructor(MethodAttributes.Public, CallingConventions.Standard, constructorArgs);

// Getting the base constructor info
// base(unitOfWork)
ConstructorInfo baseConstructor = TypeBuilder
                                      .GetConstructor(repositoryFakerBuilder.BaseType, typeof(Repository<,,>).GetConstructors()[0]);

// Defining the constructor IL generator
ILGenerator constructorIL = constructor.GetILGenerator();
// Pushing "this"
constructorIL.Emit(OpCodes.Ldarg_0);
// Pushing the params
constructorIL.Emit(OpCodes.Ldarg_1);
// Calling the base constructor
constructorIL.Emit(OpCodes.Call, baseConstructor);
// Finishing the constructor block code
constructorIL.Emit(OpCodes.Ret);
...
```

The code above speaks by itself and it’s very straightforward. The only tricky part is where the base constructor info is obtained. It took me about a couple of hours to realize the proper workaround when you are trying something like this with generics its using the GetConstructor static method from the TypeBuilder Type.

At this point you’re ready to bake the type on the fly and the only left code is:

```csharp
...

repositoryFakerBuilder.CreateType();

...
```

I’ll illustrate who I ended using this code, but for better understanding I’ll show first the whole method code which is part of a larger class called EFRepositoryHelper

```csharp
public static Type GetContextBindableRepository()
    where TDbContext: DbContext, IContextDbSetWithEntityKeyResolver
{
    // Defining a type builder for a public class named "RepositoryFaker".
  	TypeBuilder repositoryFakerBuilder = GetFakeRepositoryModuleBuilder()
                                            .DefineType("RepositoryFakerAssembly.RepositoryFaker", TypeAttributes.Public);

  	// Defining type parameters
  	// .. <TKey, T>
  	GenericTypeParameterBuilder[] genericParamsBuilders = repositoryFakerBuilder.DefineGenericParameters(new string[]{"TKey","T"});

  	// Defining clause
  	// .. where T : class...
  	genericParamsBuilders[1].SetGenericParameterAttributes(GenericParameterAttributes.ReferenceTypeConstraint);

  	// Defining clause
  	// .. where T : ...,  IEntityKey
  	genericParamsBuilders[1]
        .SetInterfaceConstraints(
            new Type[]{typeof(IEntityKey<>).MakeGenericType(new Type[]{repositoryFakerBuilder.GenericTypeParameters[0]})});

  	// Defining inherited class with their type params
  	// .. :Repository<TKey, T, DomainContext>
  	repositoryFakerBuilder
        .SetParent(typeof(Repository<,,>)
        .MakeGenericType(new Type[]
        {
            repositoryFakerBuilder.GenericTypeParameters[0], repositoryFakerBuilder.GenericTypeParameters[1], typeof(TDbContext)
        }
    ));

  	// Defining constructors parameters
  	// .. (IEFUnitOfWork unitOfWork)
  	Type[] constructorArgs = new Type[1];
  	constructorArgs[0] = typeof(IEFUnitOfWork<>).MakeGenericType(typeof(TDbContext));

  	// Defining constructor builder and passing it the params
  	// public RepositoryFaker(IEFUnitOfWork unitOfWork) : base(unitOfWork)
  	ConstructorBuilder constructor = repositoryFakerBuilder
                                        .DefineConstructor(MethodAttributes.Public, CallingConventions.Standard, constructorArgs);

  	// Getting the base constructor info
  	// base(unitOfWork)
  	ConstructorInfo baseConstructor = TypeBuilder
                                          .GetConstructor(repositoryFakerBuilder.BaseType, typeof(Repository<,,>)
                                              .GetConstructors()[0]);

  	// Defining the constructor IL generator
  	ILGenerator constructorIL = constructor.GetILGenerator();
  	// Pushing "this"
  	constructorIL.Emit(OpCodes.Ldarg_0);
  	// Pushing the params
  	constructorIL.Emit(OpCodes.Ldarg_1);
  	// Calling the base constructor
  	constructorIL.Emit(OpCodes.Call, baseConstructor);
  	// Finishing the constructor block code
  	constructorIL.Emit(OpCodes.Ret);

  	return repositoryFakerBuilder.CreateType();

  	// Emitted Class

  	//public class RepositoryFaker<TKey, T>: Repository<TKey, T, DomainContext> where T : class, IEntityKey
  	//{
  	//    public RepositoryFaker(IEFUnitOfWork unitOfWork) : base(unitOfWork)
  	//    {
  	//    }
  	//}
}
```

Now you have a wider view of the whole thing, the custom Entity Framework Repository implementation can be resolved by Ninject with this tweak:

```csharp
...

kernel.Bind()
  .ToSelf()
  .InScope(ctx => StandardScopeCallbacks.Thread(ctx));
kernel.Bind(typeof(IEFUnitOfWork<>))
  .To(typeof(UnitOfWork<>))
  .InScope(ctx => StandardScopeCallbacks.Thread(ctx));
kernel.Bind(typeof(IRepository<,>))
  .To(EFRepositoryHelper.GetContextBindableRepository())
  .InScope(ctx => StandardScopeCallbacks.Thread(ctx));

...
```

Today we've learned to dynamically create a generic type on the fly with System.Reflection.Emit, but as I usually quote, there are no silver bullets, so feel free to do any suggestions about this code.  
I got to say this topic may seems easy with the proper knowledge, but believe me, don’t let the enthusiasm of an easy example drag you in the wrong way, that’s serious stuff that may become a big unraveled mess of code. So, respect for those who usually deal with this day by day like the guys from [Castle Project](http://www.castleproject.org "Castle Project").

See you on the next one.
