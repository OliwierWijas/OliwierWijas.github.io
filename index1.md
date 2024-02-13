# Workshops 1

<p>Hello everyone at the first session of Workshops!</p>
<p>Today we'll be focusing on threads and how they work within a shared environment.</p>
<p>Let's get started!</p>

## Exercise 1

<p>Below is a UML of the classes needed. Please note the UML diagram may not be complete, and you're welcome to add to it as is needed.</p>

![image](https://github.com/OliwierWijas/OliwierWijas.github.io/assets/119060666/547f93f1-a2e2-4538-af0d-a721a8cf463b)

#### Step 1 - the Counter class

<p>Create a class, Counter, with a single private field variable, count, of type int. Initialized count to 0 in the constructor.</p>

<p>The incrementCount method should add 1 to the count.</p>

<blockquote>
<details>
<summary>Display solution for the Counter class</summary>
{% highlight java %}
public class Counter
{
  private int count;
  
  public Counter()
  {
    this.count = 0;
  }
  
  public void incrementCount()
  {
    count++;
  }
  
  public int getCount()
  {
    return count;
  }
}
{% endhighlight %}

</details>
</blockquote>

#### Step 2 - the Runnable class

<p>Create a Runnable class (a class implementing the Runnable interface), CountIncrementer, which takes a reference to your class with the counter (an argument for the constructor of CountIncrementer).</p>

<p>In the run() method of CounterIncrementer, call the update method 1,000,000 times in a for-loop, so that the ‘count’ variable in Counter is incremented. When the for-loop is done, i.e. after and outside of the for-loop, get the count variable, and print it out.</p>

<blockquote>
<details>
<summary>Display solution for the CounterIncrementer class</summary>
  
```java
public class CounterIncrementer implements Runnable
{
  private Counter counter;
  
  public CounterIncrementer(Counter counter)
  {
    this.counter = counter;
  }
  
  @Override public void run()
  {
    for (int i = 0; i < 1_000_000; i++)
    {
      counter.incrementCount();
    }
    System.out.println(counter.getCount());
  }
}
```

</details>
</blockquote>

#### Step 3 - the Start class

<p>Now create a Start class with a main method. Instantiate the Counter class, instantiate one instance of CountIncrementer, and start the Thread.

<blockquote>
<details>
<summary>Display solution for the Start class</summary>
  
```java
public class Start
{
  public static void main(String[] args)
  {
    Counter counter = new Counter();
    CounterIncrementer counterIncrementer1 = new CounterIncrementer(counter);
    Thread t1 = new Thread(counterIncrementer1);
    t1.start();
  }
}
```

</details>
</blockquote>

<p>Verify that the number printed out is 1,000,000.</p>

#### Step 4 - add another Thread

<p>Change the code in your main method so that two CounterIncrementer threads are created and started but use the same counter.</p>

<blockquote>
<details>
<summary>Display solution for the Start class</summary>
  
```java
public class Start
{
  public static void main(String[] args)
  {
    Counter counter = new Counter();
    CounterIncrementer counterIncrementer1 = new CounterIncrementer(counter);
    CounterIncrementer counterIncrementer2 = new CounterIncrementer(counter);
    Thread t1 = new Thread(counterIncrementer1);
    Thread t2 = new Thread(counterIncrementer2);
    t1.start();
    t2.start();
  }
}
```

</details>
</blockquote>

<p>Two threads will now both increment the count variable 1,000,000 times, which means that the counter is incremented 2,000,000 times. Run it a few times. Is the value always 2,000,000?</p>

<p>We’ll fix this problem next time.</p>
