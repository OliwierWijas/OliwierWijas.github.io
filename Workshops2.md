# Workshops 2

<p>Hello everyone at the second session of Workshops!</p>
<p>Today we'll be focusing on MVVM pattern combined with the State and Observer patterns.</p>
<p>Today you will have one big exercise, which is about making a task management application.</p>
<p>Let's get started.</p>

## Exercise

<p>Below is a UML of the classes needed. Please note the UML diagram may not be complete, and you're welcome to add to it as is needed.</p>

![image](https://github.com/OliwierWijas/OliwierWijas.github.io/assets/119060666/1c7a581f-9b38-40dd-83e9-34ccdbbe0962)

##### Before starting remember to keep track of the module-info.java file in order to include important libraries and to export proper packages if needed.

#### Step 1 - the Person class

<p>Implement the Person class from the class diagram, I think you have done it many times and do not need additional information about this. :)</p>

<blockquote>
<details>
<summary>Display solution for the Person class</summary>
  
```java
public class Person {
    private final String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public boolean equals(Object obj) {
        if(obj == null || obj.getClass()!= getClass()) return false;
        Person p = (Person) obj;
        return p.name.equals(this.getName());
    }

    public String toString() {
        return name;
    }
}
```

</details>
</blockquote>

#### Step 2 - the State pattern

<p>After the Person class has been implemented, we can move on to the implementation of the State Pattern.</p>

<p>You should start with the State interface. It should contain two methods: startTask() and finishTask() and those methods should behave differently depending on the current state.</p>

<p>In the NotStarted class, the start method should change the state to InProgress, but should throw and exception if the finish method is called (we cannot finish unstarted task). While a task being InProgress, it should be allowed to change the state to Done, when a task has been fulfilled, but there should be an exception thrown whenever we try to call the start method. The last state - Done - should throw an exception whenever any of the two methods is called (cannot undo a task or restart it).</p>

<p>You can add toString() method in all of the state concrete implementations, since we will need them in the view.</p>

<blockquote>
<details>
<summary>Display solution for the State Pattern</summary>
  
```java
public interface State {
    void startTask(Task task);
    void finishTask(Task task);
}
```

```java
public class NotStarted implements State {
    @Override
    public void startTask(Task task) {
        task.setState(new InProgress());
    }

    @Override
    public void finishTask(Task task){
        throw new RuntimeException("You can not finish not started task!!!");
    }

    public String toString(){
        return "Not Started";
    }
}
```

```java
public class InProgress implements State {
    @Override
    public void startTask(Task task) {
        task.setState(this);
        throw new RuntimeException("Task is already started!!!");
    }

    @Override
    public void finishTask(Task task) {
        task.setState(new Done());
    }
    
    public String toString() {
        return "In Progress";
    }
}
```

```java
public class Done implements State {
    @Override
    public void startTask(Task task) {
        throw new RuntimeException("You can not start finished task!!!");
    }

    @Override
    public void finishTask(Task task) {
        task.setState(this);
        throw new RuntimeException("Task is already finished!!!");
    }

    public String toString() {
        return "Done";
    }
}
```

</details>
</blockquote>

#### Step 3 - the Task class

<p>The same as in the Person class, I believe you do not need any additional assistance. You have done more complecated stuff in SDJ1. :)</p>

<p>The only things you can encounter problems on are the start and finish methods. Basically you should call a corresponding method on the state object you have a reference to (those are the methods that we created in the State pattern).</p>

<blockquote>
<details>
<summary>Display solution for the Person class</summary>
  
```java
public class Task {
    private final String title;
    private final String description;
    private State state;
    private Person person;

    public Task(String title, String description, Person creator) {
        this.title = title;
        this.description = description;
        this.state = new NotStarted();
        this.person = creator;
    }

    public State getState() {
        return state;
    }

    public void setState(State state) {
        this.state = state;
    }

    public String getTitle() {
        return title;
    }

    public String getDescription() {
        return description;
    }

    public Person getPerson() {
        return person;
    }

    public void startTask() {
        state.startTask(this);
    }
    
    public void finishTask() {
        state.finishTask(this);
    }
    
    public boolean equals(Object obj) {
        if(obj == null || obj.getClass()!= getClass()) return false;
        Task p = (Task) obj;
        return p.getTitle().equals(this.getTitle()) && p.getDescription().equals(this.getDescription()) && p.getState().equals(this.getState()) && p.getPerson().equals(this.getPerson());
    }

    public String toString() {
        return title+": " +description;
    }
}
```

</details>
</blockquote>

#### Step 4 - the ModelManager interface and implementation

