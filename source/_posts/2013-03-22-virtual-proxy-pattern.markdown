---
layout: post
title: "Virtual Proxy"
date: 2013-03-22 19:35
comments: false
categories: [design patterns, proxy, java, virtual proxy]
---
{% blockquote %}
It controls the access to a resource that is expensive to create, for instance data for object creation have to be retrieved from a network.
{% endblockquote %}
<!-- more -->
__Scenario.__ The client has to interact with an object which is very expensive to create. 

__Virtual proxy.__ The proxy hides the complexity of creating an managing the real subject. Moreover the proxy could _handle the request by itself_ or _creates the expensive object_ and directly delegate to it the request.

__Proxy in action.__ The proxy acts as a _surrogate_ for the real object before and while it is created.

##Example
Each contact of an address book has some details such as the name, surname, address and a picture. The picture should always be up to date, for this reason must be retrieved from the network or from a database. The creation of the picture is expensive involving a connection. 

The proxy will control how the picture is retrieved without blocking the main application.

{% img /images/posts/design-patterns/virtual_proxy_example.png %}

Implementing the same interface, `Picture`, `ProxyPicture` can be used in place of `ContactPicture` so the client will use an instance reference of type `Picture` without knowing that it is a proxy.

### Start
When the system starts will create a proxy which will be passed to the client, the `Contact` class. Calling `toString()` method the first time will start the retrieval process.
{% codeblock lang:java %}
public class AddressBook {
	public static void main(String[] args) {
		new AddressBook().run();
	}

	public void run() {
		NetworkService ns = new NetworkServiceImpl();
		Contact contact = new Contact(new PictureProxy(ns));
		System.out.println(contact.toString());
	}
}
{% endcodeblock %}

###The Client
It gets a reference of type `Picture` thinking to be the real object, but it is the proxy.
{% codeblock lang:java %}
public class Contact {

	private Picture picture;

	public Contact(Picture p) {
		picture = p;
	}

	@Override
	public String toString() {
		if (picture.getName() == null) {
			return "picture is going to be retrieved...";
		} else {
			return picture.getFormat() + " " + picture.getName() + " " + new String(picture.getImage());
		}
	}
}
{% endcodeblock %}

###The Proxy
It **_wraps_** the _real object_. At the first method call, the proxy will start a thread to retrieve the data and return a `null` value until data are not available then it **_builds_** the real object it wraps. Once the proxy will have the data it will _directly answer_ to the client.
{% codeblock lang:java %}
import java.net.MalformedURLException;
import java.net.URL;

public class PictureProxy implements Picture {

	private ContactPicture contactPicture;
	private boolean isRetrieving;
	private NetworkService networkService;

	public PictureProxy(NetworkService ns) {
		networkService = ns;
	}

	@Override
	public String getName() {
		if (contactPicture == null) {
			retrieveImage();
		} else {
			contactPicture.getName();
		}
		return null;
	}

	@Override
	public String getFormat() {
		// some code
	}

	@Override
	public byte[] getImage() {
		// some code
	}

	private void retrieveImage() {
		if (!isRetrieving) {
			isRetrieving = true;
			Thread thread = new Thread(new Runnable() {
				@Override
				public void run() {
					try {
						contactPicture = new ContactPicture("john picture", 
							"png", networkService.getImage(new URL("http://imagerepo.com")));
						System.out.println("picture retrieved");
					} catch (MalformedURLException e) {
						e.printStackTrace();
					}
				}
			});
			thread.start();
		}
	}
}
{% endcodeblock %}
