mytestingframework
==================

Putting this together to share my preferred testing setup for C# apps


Test Framework
---------------
I have used mstest, nunit, etc and, by far, my favorite framework is [MSpec](https://github.com/machine/machine.specifications). In my opinion, the syntax allows for very quick testing and readable results.

Mocking Framework
-----------------
[NSubstitute](http://nsubstitute.github.io/help/getting-started/). Again, I like this for the syntax, specifically it does not force you to write a bunch of lamdas to set up stubs etc.

Automocking
------------
I feel that the key to writing tests quickly is to have an automocking framework setup. Basically, what automocking gives you is the ability to create your test subject class with all dependencies mocked and injected into the subject. For my container, I use [StructureMap](https://github.com/structuremap/structuremap), which comes with automocking functionality. I created a nuget package that provides nsubstitute integration for StructureMap's automocking functionality (I think that it only comes with RhinoMock and Moq integration) and you can find that [here](https://www.nuget.org/packages/structuremap.automocking.nsubstitute/). I then create a helper class which wraps all of the calls to the automocking functionality of structure map.


Putting it all together
-----------------------
Let's say you have a class LightSwitch, which depends on an interface IAmThePowerSupply:

```c#
public class LightSwitch{
  private IAmThePowerSupply _power;
  
  public LightSwitch(IAmThePowerSupply power){
    _power = power;
  }
  
  public void TurnOn(){
    _power.ToggleOn(Pin.4);
  }
}

```
And you want to make sure that calling the TurnOn() method does the right thing. You would write your spec like this:

```c#
[Subject(typeof(LightSwitch)]
public class When_turning_on_the_light_switch : With<LightSwitch>{
  Because of = () => Subject.TurnOn();
  
  It should_have_turned_pin_4_on = () => For<IAmThePowerSupply>().Received().ToggleOn(Pin.4);
}
```

Now, the helper class that I alluded to before and used in this example would look something like this:

```c#
public abstract class With<T> 
        where T : class
    {
        Establish context = () => Mocks = (AutoMocker<T>)NSubstituteAutoMockerBuilder.Build<T>();
            
        protected static TDependency For<TDependency>()
            where TDependency : class
        {
            return (typeof(TDependency).IsInterface) ? Mocks.Get<TDependency>() : Mocks.Container.GetInstance<TDependency>();
        }

        protected static AutoMocker<T> Mocks { get; private set; }
        protected static T Subject { get { return Mocks.ClassUnderTest; } }
    }

```

