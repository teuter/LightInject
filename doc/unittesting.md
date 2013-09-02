#Unit testing#

**LightInject** provides native support for mocking services during unit testing.

## Installing ##

<div class="nuget-badge" >
   <p>
         <code>PM&gt; Install-Package LightInject.Mocking </code>
   </p>
</div>

**Note:** *Use the [InternalVisibleTo](http://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.internalsvisibletoattribute.aspx) attribute to give the test project access to the **LightInject** internal types.*

## Example ##

	public class FooWithDependency : IFoo
	{
		public FooWithDependency(IBar bar)
		{
		}
	}


The container is configured as usual at the composition root.

	container.Register<IFoo,FooWithDependency>();
	container.Register<IBar,Bar>();

Now, in our unit test project we would like to write tests against the **IFoo** service, but we would also like to replace the dependency (**Bar**) with a mock instance.

The following example uses the [Moq](http://code.google.com/p/moq/) library to provide a mock instance for the **IBar** dependency.

	barMock = new Mock<IBar>();
	container.StartMocking<IBar>(() => barMock.Object);	

	var foo = (FooWithDependency)container.GetInstance<IFoo>();

	Assert.IsNotInstanceOfType(foo.Bar, typeof(Bar));
	container.EndMocking<IBar>()

When the **StartMocking** method is called, the container will replace the original service registration with a new service registration that uses our mock instance. 

**Note:** *The mock instance uses the same lifetime as the original registration.*

The **StopMocking** method tells the container to replace the mock registration with our original service registration.

	barMock = new Mock<IBar>();
	container.StartMocking<IBar>(() => barMock.Object);	
	container.EndMocking<IBar>();
	
	var foo = (FooWithDependency)container.GetInstance<IFoo>();

	Assert.IsInstanceOfType(foo.Bar, typeof(Bar));