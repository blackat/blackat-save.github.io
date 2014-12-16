---
layout: post
title: "Protection Proxy"
date: 2013-03-22 19:38
comments: false
categories: [design patterns, proxy, protection proxy, java]
---
{% blockquote %}
It controls the access to a resource based on access rights.
{% endblockquote %}
<!-- more -->

__Scenario.__ A proxy _controls the access_ to the real object applying _protection_ to the method calls in a transparent way. The client will invoke methods against the proxy thinking it is the real object.

__Java dynamic proxy support.__ `java.lang.reflect` package can be used to create a proxy class _on the fly_. The _proxy class_ implements one or more interfaces and delegates method invocation to a  specified class, the _invoker handler_.

__Class diagram.__ The use of `java.lang.reflect` package imposes a change in the proxy pattern class diagram.
{% img /images/posts/design-patterns/protection_proxy.jpg %}

__Proxy in action.__ The handler answers to any method call made by the client on the proxy. The proxy implements _the same interface_ as the real object. 

##Example
A reviewer can only get information about a movie and write comments, he cannot change the title or the actors of the movie.

###Start
The `MovieReviews` class simply creates a of two movie proxies, each proxy manages one movie.
Using the _static method_ `Proxy.newProxyInstance(Movie.class.getClassLoader(), new Class[]{Movie.class}, forrestGumpHandler);` allows the creation of a new proxy _on the fly_ passing:

* the same class loader of the `Movie` interface or of the _real movie object_,
* the interface the proxy has to expose,
* the handler every method call has to be delegated to, it also _wraps_ the _real object_.
{% codeblock lang:java %}
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;
import java.util.ArrayList;
import java.util.List;

public class MovieReviews {
	public static void main(String[] args) {
		new MovieReviews().run();
	}

	public void run() {
		Reviewer reviewer = new Reviewer(buildMovieList());
		try {
			reviewer.spoilMovieTitle();
		} catch (Exception e) {
			System.out.println("operation not permitted.");
		}
		reviewer.printMovieTiles();
	}

	public List<Movie> buildMovieList() {
		List<Movie> movieList = new ArrayList<Movie>();

		Movie forrestGumpMovie = new MovieImpl("Forrest Gump", "Tom Hanks", "Gary Sinise", "Robin Wright");
		InvocationHandler forrestGumpHandler = new InvocationHandlerImpl(forrestGumpMovie);
		Movie forrestGumpProxy = (Movie) Proxy.newProxyInstance(Movie.class.getClassLoader(), 
			new Class[]{Movie.class}, forrestGumpHandler);

		Movie djangoMovie = new MovieImpl("Django", "Quentin Tarantino", "Jamie Foxx", "Franco Nero");
		InvocationHandler djangoHandler = new InvocationHandlerImpl(djangoMovie);
		Movie djangoProxy = (Movie) Proxy.newProxyInstance(Movie.class.getClassLoader(), 
			new Class[]{Movie.class}, djangoHandler);

		movieList.add(forrestGumpProxy);
		movieList.add(djangoProxy);

		return movieList;
	}
}
{% endcodeblock %}

###The Interface
Very simple interface with some getters, setters and the method to comment the movie.
{% codeblock lang:java %}
import java.util.List;

public interface Movie {
	String getTitle();
	void setTitle(String title);
	List<String> getActors();
	void setActors(List<String> actors);
	void comment(String comment);
}
{% endcodeblock %}


###The Invocation Handler
This class will invoke the method, by using _reflection_, on the _real object_ that is _wrapped_. The proxy object will create at runtime, so the handler is the only place where the protection logic can be put. 

The protection disallow to use any `setter` method, only `getters` and `comment` methods can be invoked.
{% codeblock lang:java %}
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class InvocationHandlerImpl implements InvocationHandler {

	private Movie movie;

	public InvocationHandlerImpl(Movie c) {
		movie = c;
	}

	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		try {
			if (method.getName().startsWith("set")) {
				throw new IllegalAccessException();
			} else if (method.getName().equals("comment")) {
				return method.invoke(movie, args);
			} else if (method.getName().startsWith("get")) {
				return method.invoke(movie, args);
			}
		} catch (InvocationTargetException e) {
			e.printStackTrace();
		}
		return null;
	}
}
{% endcodeblock %}


###The Client
The client will receive the list of the movies which he can write a comment on. From the reviewer point of view every object exposes the `Movie` interface, so he doesn't know to invoke methods against a proxy object. 

Remember, both the proxy and the real movie object implement the _same interface_ so the proxy can _take the place_ of the _real object_.
{% codeblock lang:java %}
import java.util.List;

public class Reviewer {

	List<Movie> movieList;

	public Reviewer(List<Movie> list) {
		movieList = list;
	}

	public void spoilMovieTitle() {
		for (Movie movie : movieList) {
			movie.setTitle("spoiled");
		}
	}

	public void printMovieTiles() {
		for (Movie movie : movieList) {
			System.out.println(movie.getTitle());
		}
	}
}
{% endcodeblock %}