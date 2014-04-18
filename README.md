# Exploring Java 1.8

## Scenario

Suppose we have the following User class:

    public class User
    {
      public enum Color
      {
        RED, BLUE, GREEN, YELLOW, ORANGE, BLACK
      }

      String  name;
      String  emailAddress;
      Color   favoriteColor;
      int     age;
      
      public void setName(String name)
      {
        this.name = name;
      }

      public String getName()
      {
        return this.name;
      }

      public void setEmailAddress(String emailAddress)
      {
        this.emailAddress = emailAddress;
      }

      public String getEmailAddress()
      {
        return this.emailAddress;
      }

      public void setFavoriteColor(Color favoriteColor)
      {
          this.favoriteColor = favoriteColor;
      }

      public Color getFavoriteColor()
      {
        return this.favoriteColor;
      }

      public void setAge(int age)
      {
        this.age = age;
      }

      public void getAge()
      {
        return this.age;
      } 
    }

Now let's suppose we have a List of User objects, and we want to print out the users whose favorite color is RED. We could naively implement the functionality like this:

    public static void printUsersThatLikeRed(List<User> users)
    {
      for (User user : users)
      {
        if (user.getFavoriteColor() == Color.RED)
        {
          System.out.println(user.getName());
        }
      }
    }

This works, but it's brittle. If the requirements change from finding users who like red to, say, blue, then the code must be changed, or we must re-implement an almost identical method. Let's fix it...

    public static void printUsersThatLikeColor
    (
      List<User>  users,
      Color       color
    )
    {
      for (User user : users)
      {
        if (user.getFavoriteColor() == color)
        {
          System.out.println(user.getName());
        }
      }
    }

Still brittle! It depends on the User class interface, and if that were to change this code would be broken. It is also not very flexible. Suppose we want to get users whose favorite color is blue, and are also less than 27 years old. Then what?

We could specify an _interface_, say `Tester`, with a boolean method `test(User user)`, and create an implementation of that interface which returns `true` if the User meets the criteria we want. We could then accept an instance of that implementation in our method, and let our method call its `test` method for each User in the list. This can be done more succinctly using  an _anonymous class_...

    interface Tester
    {
      boolean test(User user);
    }

    public void printUsers
    (
      List<User>  users,
      Tester      tester
    )
    {
      for (User user : users)
      {
        if (tester.test(user))
        {
          System.out.println(user.getName());
        }
      }
    }

    printUsers
    (
      users,
      new Tester()
      {
        boolean test(User user)
        {
          return (user.getFavoriteColor() == Color.RED && user.getAge() < 27);
        }
      }
    );

This is much better, but it is still quite a bit of code to write. Using our same method, we can reduce the amount of code to write when calling it using a _lambda expression_...

## Lambdas

Lambdas allow code to be treated as data. That is, a piece of executable code can be passed around just like data, and called/executed at a later point.

    printUsers
    (
      users,
      (User user) -> user.getFavoriteColor() == Color.RED && user.getAge() < 27
    );

## Functional Interfaces

The last example above works because our interface, Tester, is a _functional interface_. A functional interface is any interface with only one _abstract_ method (but may contain one or more _default_ methods or _static_ methods). The Java compiler knows that `Tester` is a functional interface, and it knows that the `printUsers` method's second parameter is an instance of that functional interface, so it allows us to omit the declaration of the `new` interface implementation and the method name within it, and instead use the lambda expression.

## Standard Functional Interfaces

The Java 1.8 JDK provides several standard generic functional interfaces for common methods in the `java.util.function` package. One such standard functional interface is `Predicate<T>`:

    interface Predicate<T>
    {
      boolean test(T t);
    }

We can re-implement our `printUsers` method using the `java.util.function.Predicate` interface and eliminate our Tester interface altogether, shortening our code and making it more generalized...

    public void printUsers
    (
      List<User>      users,
      Predicate<User> tester
    )
    {
      for (User user : users)
      {
        if (tester.test(user))
        {
          System.out.println(user.getName());
        }
      }
    }

And call it like so:

    printUsers
    (
      users,
      user -> user.getFavoriteColor() == Color.RED && user.getAge() < 27
    )

This looks very similar to the previous implementation, but in this case we do not need to define our own interface, and by using a standard interface our method becomes more compatible with other code in general. We've improved quite a bit, but we can do even better! Let's suppose now that our requirements change - sometimes we want to print out all the users, but other times we need to send them an email. We can replace the call to `System.out.println` with a call to an interface method, and then pass a lambda with the desired functionality when we call `printUsers`. It turns out that `java.util.function` provides just the interface we need, `Consumer<T>`, which has the method `accept` that takes one argument, an object, and returns `void`:

    interface Consumer<T>
    {
      void accept(T t);
    }

Since we're generalizing the functionality of our method to do more than print users, we'll rename the method to `processUsers` instead...

    public void processUsers
    (
      List<User>      users,
      Predicate<User> tester,
      Consumer<User>  action
    )
    {
      for (User user : users)
      {
        if (tester.test(user))
        {
          action.accept(user);
        }
      }
    }

And again we call it like so:

    processUsers
    (
      users,
      user -> user.getFavoriteColor() == Color.RED && user.getAge() < 27,
      user -> System.out.println(user.getName());
    );

Or:

    processUsers
    (
      users,
      user -> user.getFavoriteColor() == Color.RED && user.getAge() < 27,
      user -> sendAnEmail(user.getEmailAddress())
    );

We've made the method pretty generalized now: we can use _any_ functionality we want to select the users we want to take action on, and we can take any action we want, all using the same method implementation. But what if we want to use the same iterate-filter-act behavior for a collection of a different type, say, Posts? Let's pretend we want to display a User's posts, but only the ones that are marked public. Using _Generics_ we can generalize our method even further to go beyond just User collections. We'll rename it to `processItems`...

    public <X,Y> void processItems
    (
      Iterable<X>   items,
      Predicate<X>  tester,
      Consumer<X>   action
    )
    {
      for (X item : items)
      {
        if (tester.test(item))
        {
          action.accept(item);
        }
      }
    }

Now we can implement both requirements using this `processItems` method:

    processElements
    (
      users,
      user -> user.getFavoriteColor() == Color.RED && user.getAge() < 27,
      user -> sendAnEmail(user.getEmailAddress())
    );

    processElements
    (
      user.getPosts(),
      post -> post.getVisibility() == "public",
      post -> displayPost(post);
    );

Tada! Now we have a reusable _behavior_ pattern that we can apply to any collection.

## Method References

Sometimes lambas do nothing but call an existing method. In those cases the method can be referenced directly by it's name, using a special `::` syntax instead of the lambda syntax. For instance, in our previous example of displaying public posts, our last argument to `processElements` is the lambda `post -> displayPost(post)`. Assuming `displayPost` is a method of an object called `postViewManager`, we could instead write:

    processElements
    (
      user.getPosts(),
      post -> post.getVisibility() == "public",
      postViewManager::displayPost
    );

This allows our code to be more concise and a bit more direct when an existing method fits the bill of a lambda we want to use. 

## Some History

This approach to programming, where behavior is based on the evaluation of functions instead of mutable state, is known as functional programming. Functional programming has its roots in two theoretical mathematical foundations:  Combinatorial Logic, which was largely introduced by Moses Schonfinkel in the early 1920's and picked up by Haskell Curry in 1927, and Lambda Calculus, which was introduced by Alonzo Church in the 1930's. One of the early functional-flavored programming languages is Lisp, developed by John McCarthy at MIT in the 1950's. Functional programming has been around for over 50 years and its influence is widely present in popular modern languages like Ruby, JavaScript, Haskell, Erlang, Python, and others.

## Aggregate Operations, Pipelines, and Streams

Given fucntional programming's lengthy history, several common and fundamental behaviors and operations have emerged which are provided in most functional-flavored languages. Java 8 provides them as _Aggregate Operations_, largely through the Stream API, which is based on _stream_ objects. A stream is a sequence of elements, similar to an iterator, but NOT a data structure. Streams gets values from a source, such as collection data structure, and pump them into a _pipeline_, which is nothing more than a sequence of aggregate operations. The distinction between between streams and data structures is important, because streams are not limited to sourcing data from data structures; they can also _generate_ data, even infinitely. 

Rather than implementing our own iteration behavior as in the previous examples, we could instead employ aggregate operations to achieve the same behavior:

    users.stream()
         .filter(user -> user.getFavoriteColor() == Color.RED && user.getAge() < 27)
         .forEach(user -> System.out.println(user.getName());

    users.stream()
         .filter(user -> user.getFavoriteColor() == Color.RED && user.getAge() < 27)
         .forEach(user -> sendAnEmail(user.getEmailAddress));

    user.getPosts()
        .stream()
        .filter(post -> post.getVisibility() == "public")
        .forEach(postViewManager::displayPost);

The stream API provides many aggregate operations which can be chained together to perform complex transformations of data with very little code. Let's say we want to get the average age of all users whose favorite color is green:

    double averageAge = users.stream()
                             .filter(user -> user.getFavoriteColor() == Color.GREEN)
                             .mapToInt(User::getAge)
                             .average()
                             .getAsDouble();

## Reduction Operations

The `average` method of the Stream API is a _reduction operation_, which means it returns one value by combining the elements of a stream in some fashion. Reduction operations can also return a collection instead of a single value. The Stream API provides several reduction operations, as well as two general purpose methods, `reduce` and `collect`, which make it easy to implement custom reductions.
