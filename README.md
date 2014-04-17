# Exploring Java 1.8

## Lambdas

Lambdas allow code to be treated as data. That is, a piece of executable code can be passed around just like data, and called/executed at a later point.

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
      List<User> users,
      Color color
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
      List<User> users,
      Tester tester
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

We can re-implement our `printUsers` method using the `java.util.function.Predicate` interface and eliminate our Tester interface altogether, shortening our code and making it more generalized 

If our application needed to print lists of users depending on different criteria at different times, our code will get bulky, and repetitive, which isn't very DRY.

