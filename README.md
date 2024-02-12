# Workshops 1

<p>Hello everyone at the first session of Workshops!<\p>
<p>Today we'll be focusing on threads and how they work within a shared environment.<\p>
<p>Let's get started!<\p>

## Exercise 1

<p>Below is a UML of the classes needed. Please note the UML diagram may not be complete, and you're welcome to add to it as is needed.<\p>

![image](https://github.com/OliwierWijas/OliwierWijas.github.io/assets/119060666/547f93f1-a2e2-4538-af0d-a721a8cf463b)

<p>Create a class, Counter, with a single private field variable, count, of type int. Initialized count to 0 in the constructor.<\p>

<p>The incrementCount method should add 1 to the count.<\p>

<blockquote>
<details>
<summary>Display solution for Counter Class</summary>
  
```java
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
```

</details>
</blockquote>
