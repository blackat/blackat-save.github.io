---
layout: post
title: "Observer Pattern"
date: 2013-03-25 23:25
comments: false
categories: [java, design patterns]
---
{% blockquote Bates and Sierra, http://shop.oreilly.com/product/9780596007126.do Head First Design Patterns %}
The Observer Pattern defines a one-to-many dependency between objects so that when one object changes state, all of its dependents are notified and updated automatically.
{% endblockquote %}
<!-- more -->

##Class diagram
{% img /images/posts/design-patterns/observer_pattern.png %}

Use `Subject` interface to register or remove an observer. A potential _observer_ must implements the `Observer` interface, its method will be called when the _subject's_ state changes.

The `ConcreteSubject` contains and controls the _state_, it is the sole _owner of the data_.
##Design Principle
[Loosely Coupled Design Principle.](/oo-design-principles/index.html#loosely_coupled_principle)

##Scenario
A news stand produces news during all the day. Each person interested in having news has to subscribe to the stand to get notifications. In this way the person doesn't have to check every time if a new article has been published.

##Key Points

* __Subject.__ It is the observable and updates the observers using a common interface.
* __Loosely coupled.__ Observable doesn't know anything about observers except that they implement an interface.
	* Allow flexible design minimizing the interdependency between objects.
* __Pull data.__ When data changes, observable _pull data_ to the observers.
* __Order.__ There is not any _notification order_.
* __Register.__ An  observer can register to an observable at any time.

##Observer Pattern in Action
{% codeblock lang:java %}
public interface Subject {
	void registerObserver(Observer observer);

	void removeObserver(Observer observer);

	void notifyObservers();
}
{% endcodeblock %}

{% codeblock lang:java %}
import java.util.ArrayList;
import java.util.List;

public class NewsProvider implements Subject {

	private List<Observer> observerList;
	private News news;

	public NewsProvider() {
		observerList = new ArrayList<Observer>();
	}

	@Override
	public void registerObserver(Observer observer) {
		observerList.add(observer);
	}

	@Override
	public void removeObserver(Observer observer) {
		observerList.remove(observer);
	}

	@Override
	public void notifyObservers() {
		for (Observer o : observerList) {
			o.update(news);
		}
	}

	void setNews(News n) {
		news = n;
		notifyObservers();
	}
}
{% endcodeblock %}

{% codeblock lang:java %}
public interface Observer {
    void update(News news);
}
{% endcodeblock %}

{% codeblock lang:java %}
public class NewsObserver implements Observer {

	private Subject subject;

	public NewsObserver(Subject s) {
		subject = s;
		s.registerObserver(this);
    }

	@Override
	public void update(News news) {
		System.out.println("new news published: " + news.toString());
	}
}

{% endcodeblock %}

{% codeblock lang:java %}
public class News {
	private String title;
	private String author;
	private String content;

	public News(String t, String a, String c) {
		title = t;
		author = a;
		content = c;
	}

	@Override
	public String toString() {
		return title + " " + author + ": " + content;
	}
}
{% endcodeblock %}

{% codeblock lang:java %}
public class NewsStand {
	public static void main(String[] args) {
		NewsProvider newsProvider = new NewsProvider();

		Observer userOne = new NewsObserver(newsProvider);
		Observer userTwo = new NewsObserver(newsProvider);

		newsProvider.setNews(new News("New economy", "Barry Lindon", "Many new changes happened."));
	}
}
{% endcodeblock %}
