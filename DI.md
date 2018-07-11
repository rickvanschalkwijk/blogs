# Dependency injection, so what - a deep dive into IOC
All right, all of you have heard of dependency injection. 
Most of you probably use it every day, some maybe not that often or you’ve never used it before. 
Why should you use dependency injection? What is it good for? What problem does it solve? 
These are questions you maybe know the answer to, but let’s take a step back and see what dependency injection really is about.  

## Problem solving
DI mainly solves two problems: testability and dependency resolving. 
The next question you are probably asking yourself is: "Testability? I always test my code in production." 
WRONGWRONGWRONG. Over the years I've worked on many different software projects; 
the projects that were most successful were always those in which code is fully unit tested.
Dependency injection makes sure that your code is decoupled and therefore easy to unit test. 
For example, when you create a new instance of a class inside another class’s method you couple these two classes
together making them hard or even impossible to unit test. The next problem dependency injection solves is dependency resolving.
Have you ever seen code like this: 

![Alt text](img/di-wrong.png)


Firstly, you can imagine the maintenance nightmare it causes when the CustomerService class is instantiated ten different times in your application. Secondly, think about the consequences when the DomainModelFactory gets a new argument in its constructor. You will have to build your project and work though all the errors to make sure the DomainModelFactory is instantiated correctly. By the way, if you haven't noticed, this is a relatively simple instantiation. Imagine if the SomeValidator class took three more arguments in its constructor. Pretty scary right? Luckily, dependency injection can resolve both these problems for us. Killing two birds with one stone.  
 
Previously, I mentioned that dependency injection decouples your classes. Let me give you an example. Look at the next piece of code:
Ok, what is happening here? We’ve got some customer service class that updates an order for a customer. It saves the customer details to a database through a repository class, same goes for the order details. Let’s say that I want to unit test this method. You know the answer to this question, right? You can’t unit test this piece of code without hitting the database every time you run it. Furthermore, we don’t want to hit the database in a unit test - that sort of stuff is reserved for integration testing. What we need to do is remove the two hard references from the repository classes. We have to make sure the CustomerService class is only depending on interfaces.
  
Here is the refactored class. I’ve taken the two repository classes, created interfaces for them and used the CustomerService constructor to set them. Now our dependency injection container can wire up the repository classes in production code. We can inject mock classes in our unit tests so we don’t hit the database and the best thing about it is that the CustomerService class won’t notice a thing! This is depending on an interface/abstraction and not on concrete class. 
 
Constructor injection versus setter injection
There’re two forms of dependency injection. The first one, as described above, is constructor injection where all the dependencies are resolved using the class’s constructor. The second one is property injection where all the dependencies are resolved using writeable public or internal class properties. Let me give you an example:

using System;
 
namespace CustomerStuff
{
    public class CustomerService
    {
        public ICustomerRepository _customerRepository { get; set; }
        public IOrderRepository _orderRepository { get; set; }

        public void UpdateOrder(Order order, Customer customer)
        {
            // save customer details to database
            _customerRepository.Save();
            // save order details to database
            _orderRepository.Update();
         }
    }
}

So the main disadvantage of setter injection is that we don’t know which properties are required and with not. This is really run time error prone. With the constructor version of injection you have to inject everything or your code doesn’t even compile.
(Still need to explain more about property injection)
So, we now know what dependency injection can do for us, but dependency injection itself can't solve it because dependency injection is an architectural design pattern, not some program that can solve it for us. For that we need a dependency injection container also called inversion of control container. Over the years there have been a lot of good dependency injection containers: StructureMap, AutoFac, Ninject and CastleWinsor are just a small haul out of a big box of tools. They practically all do the same thing, taking care of two things: object registering and object resolving. 

By the way, .NET Core also has a dependency injection container built into the framework and it’s pretty awesome! This built-in dependency injection container is not the most advanced one but for seventy percent of the time it gets things done. The .NET Core DI container is exposed by the IServiceProvider interface. By using this provider, you can get constructor injection out of the box by default.

IServiceCollection services
	.AddOptions()
	.AddScoped<IDomainModelFactory, DomainModelFactory>()
	.AddTransient<IDateTransferFactory, DateTransferFactory>()
	.AddSingleton<IRestClientWrapper, RestClientWrapper>();


The code snipped above shows a basic ServiceCollection configuration

Conclusion
Hopefully you’ve got some more insights into dependency injection. Just to wrap things up, dependency injection is not a goal but a solution to a particular set of problems. Dependency Injection makes it easy to replace abstractions for unit testing and makes your application more flexible, since you can swap, decorate and intercept dependencies without the consuming class’s knowledge. That doesn't mean that you should inject every dependency a class has,  only if it helps you in making the class more testable and the system more maintainable. So you have to ask yourself whether it makes sense from a testing perspective to inject those dependencies. 

