---
layout: post
title: "Remote Proxy"
date: 2013-03-22 19:36
comments: false
categories: [design patterns, proxy, remote proxy, java]
---
{% blockquote %}
The proxy acts as a _local representative_ for an object that lives somewhere else on the network on a different JVM.
{% endblockquote %}
<!-- more -->
__Scenario.__ The client uses a service invoking some methods. Actually the service is located on a remote machine but the location the service should be transparent respect to the client.

__Remote proxy.__ It behaves as a _local representative_ for an remote object living on a _different JVM_. 

__Method call.__ A method call against the proxy results in the _transfer of the call_ over the wire to the remote JVM. Once there the method call is invoked against the real object. The result of the call is returned back to the _proxy_, then from the proxy to the _client_.

##RMI
Java Remote Method Invocation (RMI) is an example of remote proxy. RMI build two _helper objects_, _stub_ and _skeleton_, which _hide_ the communication and technical details about the transfer of the method call and the result return. The client will just interact with the proxy, one of the two helper objects.

__RMI vs. Remote proxy__ Remote proxy does not involve any helper object as RMI does, but just the proxy concept.

##Example
A very simple example consists of a client that uses a service to get something done. The client is unaware of the service location and type. The client does not know if the service is remote or local, if it will do database or disk access to provide data.

There are _five steps_ to implement a _remote service_ based on RMI. 

###Remote Service Interface
Define the _interface_ the client will use to interact with the service.
{% codeblock lang:java %}
import java.rmi.Remote;
import java.rmi.RemoteException;
import java.util.List;

public interface ContactService extends Remote {
	List<String> listAll() throws RemoteException;

	int getContactSize() throws RemoteException;
}

{% endcodeblock %}
__Remote interface.__ `java.rmi.Remote` is just a _marker_ that is an interface without methods. The marker tells that the interface will be used to support _remote calls_.

__Client.__ It uses a service of type `ContactService`, the _remote service interface_, to invoke methods without knowing any implementation detail and thinking it is the real object.

__Proxy.__ It implements the _remote interface_ as the real object does, so the proxy can be used as a _surrogate_ of the real object or, better, the proxy _substitute_ the real object for all the request.

__Stub.__ In RMI the proxy is called _stub_. It will manage all the networking and I/O operations. Something could go wrong (ex. network failure) so every remote method call is _risky_ and has to declare to throw a `java.rmi.RemoteException` to handle possible communication failures.

__Arguments and return values.__ They must be `primitive` or `Serializable`. Serialization is used to package values and transfer them across the network. In the example above, all the types implement natively `Serializable` interface as many other types from Java API.


###Remote Service Implementation
The implementation of the service is very simple, this is the _real object_ where the calls will be invoked on. The service will reside on the server machine.
{% codeblock lang:java %}
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.util.ArrayList;
import java.util.List;

public class ContactServiceImpl extends UnicastRemoteObject implements ContactService {

	protected ContactServiceImpl() throws RemoteException {
	}

	@Override
	public List<String> listAll() {
		List<String> contacts = new ArrayList<String>();
		contacts.add("Fermi");
		contacts.add("Majorana");
		return contacts;
	}

	@Override
	public int getContactSize() {
		return 18;
	}
}
{% endcodeblock %}
__Being remote.__ Service implementation must extends `java.rmi.server.UnicastRemoteObject` to make it _remote_ and so working as a _remote service object_. This class has some functionality (ex. read and write values on the socket) that allow it to _be remote_.

__Superclass constructor.__ Superclass `UnicastRemoteObject` _constructor_ throws a `RemoteException`. Superclass constructor is always called so no choice but declare that a constructor throws an exception.

__Remote service registration.__ The _remote service implementation_ needs be registered in the registry to make it available to remote clients. The registration is done by the `Server` class.
{% codeblock lang:java %}
import java.net.MalformedURLException;
import java.rmi.Naming;
import java.rmi.RemoteException;

public class Server {
	public static void main(final String[] args) {
		try {
			ContactService contactService = new ContactServiceImpl();
			Naming.rebind("/contact_service", contactService);
		} catch (MalformedURLException e) {
			e.printStackTrace();
		} catch (RemoteException e) {
			e.printStackTrace();
		}
	}
}
{% endcodeblock %}
__RMI registry.__ Statement `Naming.rebind("/service", service)` puts the remote service _implementation_ into the RMI registry with a _service name_. The client will use this name in order to look up for the _stub_. 

__Stub.__ The server registers or puts the _stub_ in the _registry_.


###Generate Stub and Skeleton
They are the client and server _helpers_ created automatically by `rmic`. These classes implements all the code necessary to manage the connection socket and transfer the method call to the real object residing on the remote JVM.
{% codeblock lang:bat %}
MacBook:remote blackcat$ rmic ContactServiceImpl
{% endcodeblock %}
__Stub and skeleton.__ Invoking `rmic` on _service implementation_ generates `ContactServiceImpl_Stub.class` and `ContactServiceImpl_Skeleton.class`, the two helper objects. They will manage the transfer over the wire of the request about the method call which, finally, will be executed against the real object, `ContactServiceImpl`.


###Start the Registry
The `rmiregistry` is a sort of white pages 	where services can be registered and looked up. The client will look for the _proxy_ or _client helper_ or _stub_ into the register. _Classes must be available to the rmi registry._
{% codeblock lang:bat %}
MacBook:remote blackcat$ rmiregistry &
{% endcodeblock %}


###Start the Server
Once the registry has been started, it is possible to run the server which will register the service implementation.
{% codeblock lang:bat %}
MacBook:remote blackcat$ java Server
{% endcodeblock %}


### The Client
The client looks up the service, gets the reference to the _stub_ and invokes the method against it.
{% codeblock lang:java %}
import java.net.MalformedURLException;
import java.rmi.Naming;
import java.rmi.NotBoundException;
import java.rmi.RemoteException;

public class Client {
	public static void main(final String[] args) {
		new Client().start();
	}

	public void start() {
		try {
			ContactService contactService = (ContactService) Naming.lookup("rmi://127.0.0.1/contact_service");
			System.out.println("number of contacts in the address book is: " + contactService.getContactSize());
			for (String contact : contactService.listAll()) {
				System.out.println(contact);
			}
		} catch (NotBoundException e) {
			e.printStackTrace();
		} catch (MalformedURLException e) {
			e.printStackTrace();
		} catch (RemoteException e) {
			e.printStackTrace();
		}
	}
}
{% endcodeblock %}
__Naming lookup.__ Static method `Naming.lookup("rmi://127.0.0.1/contact_service")` allows the client to get the reference to an instance which represents the _helper_ or _stub_.

__Remote interface.__ The client uses the remote interface as service type without knowing the real _class name_ of the remote service.

__Cast.__ The `lookup` method returns an instance of type `Object` which has to be casted to the remote service type.