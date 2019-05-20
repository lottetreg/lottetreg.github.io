---
layout: post
title:  "The Builder Pattern"
date:   2019-05-20
---

If you're familiar with the concept of [default arguments](https://en.wikipedia.org/wiki/Default_argument) provided in some languages, you know how useful they can be. Default arguments allow us to instantiate new objects without having to explicitly pass in every requirement. Java does not provide default arguments, but there are a number of ways to simulate this feature ourselves. One such way is with the builder pattern.

Let's imagine we're working on a project that has an `Employee` class, and every employee is instantiated with an email address and a desk.

{% highlight java %}
public class Employee {
  private String emailAddress;
  private Desk desk;

  Employee(String emailAddress, Desk desk) {
    this.emailAddress = emailAddress;
    this.desk = desk;
  }
}

// This is how we create new employees
Employee employee = new Employee("pickles@gmail.com", new Desk());
{% endhighlight %}

We expect new employees to be given a new desk, so we would like to simplify the creation of employees and avoid passing in a desk argument when we don't have to; however, we've also decided that it's important to be able to inject these `Desk` dependencies. It sounds like we would benefit from using default arguments, so we're going to implement the builder pattern.

{% highlight java %}
public class Employee {
  private String emailAddress;
  private Desk desk;

  private Employee(Builder builder) {
    this.emailAddress = builder.emailAddress;
    this.desk = builder.desk;
  }

  public class Builder {
    private String emailAddress;
    private Desk desk;

    Builder(String emailAddress) {
      this.emailAddress = emailAddress;
      this.desk = new Desk();
    }

    public Builder setDesk(Desk desk) {
      this.desk = desk;
      return this;
    }

    public Employee build() {
      return new Employee(this);
    }
  }
}
{% endhighlight %}

We've placed the `Builder` class directly inside `Employee`, and as long as it's public we can use it to create new employees.

Since we've also decided that all employees must be created with an email address, we pass `emailAddress` into the builder's constructor. You can see that we also use this constructor to set the `desk` field to the default `new Desk()`.

After setting the desk, the `setDesk` method returns the current instance of `Builder`. This allows us to chain `setDesk` with `build`. If we added more setter methods, and they all returned a `Builder`, we would be able to chain those together, too. This is known as a [fluent interface](https://en.wikipedia.org/wiki/Fluent_interface), and it can make our code more readable.

With the builder pattern in place, we can now use it to create new employees with or without passing in a `Desk`.

{% highlight java %}
// Using the default desk
Employee employee = new Employee.Builder("pickles@gmail.com").build();

// Manually setting the desk
Employee anotherEmployee = new Employee.Builder("pickles@gmail.com")
            .setDesk(new Desk())
            .build();
{% endhighlight %}
