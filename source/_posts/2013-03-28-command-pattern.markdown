---
layout: post
title: "Command Pattern"
date: 2013-03-28 18:26
comments: false
categories: [java, design patterns]
---
{% blockquote Bates and Sierra, http://shop.oreilly.com/product/9780596007126.do Head First Design Patterns %}
The Command Pattern encapsulates a request as an object, thereby letting you parameterize other objects with different requests, queue or log requests, and support undoable operations.
{% endblockquote %}
<!-- more -->

##Class diagram
{% img /images/posts/design-patterns/command_pattern.png %}

* `Client` creates the `ConcreteCommand` and sets its `Receiver`.
* `Receiver` knows how to perform the work to carry out the request.
* `ConcreteCommand` defines a binding between an action and a `Receiver`.
* `Invoker` makes a request calling _execute()_ method, the `ConcreteCommand` carries the request out calling one or more actions on the `Receiver`.

##Key Points

##Command in Action
__Scenario.__ The idea is to implement a remote to control different devices in a house such as the lights, the gate, the door of the garage and so on. Imagine to have a remote having different buttons and to be able to connect to each button a device.

Each device could work in a different way from the others, a gate opens and closes, the light turn on and off. Idea is to abstract from the specific action to be performed and having a common interface.

###Command interface
This interface must be implemented by a command object which _wraps_ the `Receiver` and collect some action over it in the `execute()` method.
{% codeblock lang:java %}
public interface Command {
	void execute();
}
{% endcodeblock %}

###Receiver
The light is the `Receiver` of the _request_, it is the device produced by a _specific vendor_. It has a specific interface which describes, through the methods, the possible behavior of the device.
{% codeblock lang:java %}
public class Light {

	private boolean on;

	public void on() {
		on = true;
	}

	public void off() {
		on = false;
	}
}
{% endcodeblock %}

###Concrete Command
The `LightCommand` is a _wrapper_ for the `Receiver`, the device the request has to be delegated to. The method `execute()` group a set of action which will be invoked over the `Receiver`.

In general a command represent a specific action on a device such as turn on or turn off the light. Each command is then assign to a button on the remote.

The _command object_ encapsulate a request of a device, it is used to make requests, each request will be delegated to the wrapped `Receiver`.
This class is a _command_ and could be implemented by the vendor with specific actions for the controlled device.
{% codeblock lang:java %}
public class LightOnCommand implements Command{

	Light light;

	public LightCommand(Light light) {
		this.light = light;
	}

	@Override
	public void execute() {
		light.on();
	}
}
{% endcodeblock %}

Another command to be assigned to another button.
{% codeblock lang:java %}
public class LightOffCommand implements Command{

	// same code

	@Override
	public void execute() {
		light.off();
	}
}
{% endcodeblock %}

###Invoker
The `RemoteControl` class is the `Invoker` which, in this case, has only one button as stated by the `slot` variable, able to _hold a device to control_. When the `Client` presses the button, the method `buttonPressed()` is invoked.

The `Invoker` manages the command objects, one per button. In this case there is only one button.

The client is decoupled from the specific device interface, he doesn't have to know the details of the device that is how to turn on or of the light, he has just to press a button.
{% codeblock lang:java %}
public class RemoteControl {
	Command slot;

	public void setCommand(Command command) {
		slot = command;
	}

	public void buttonPressed() {
		slot.execute();
	}
}
{% endcodeblock %}

###Client
The client prepare the remote to be used loading command objects in the specific slots. Each command object encapsulate a request of a device.
doesn't use the device, that is the `Receviver`, directly but through the `Invoker` which is the remote control.
{% codeblock lang:java %}
public class Client {
	public static void main(String[] args) {
		// invoker
		RemoteControl remoteControl = new RemoteControl();

		// receiver
		Light light = new Light();

		// create a command and pass the receiver
		Command command = new LightCommand(light);

		// pass the command to the invoker
		remoteControl.setCommand(command);

		// press the button
		remoteControl.buttonPressed();
	}
}
{% endcodeblock %}
