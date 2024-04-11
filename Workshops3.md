# Workshops - Learning Path 3

<p>Hello everyone at the third learning path of Workshops!</p>

<p>Today we'll be focusing on recreating the previous learning path so that it uses Remote Method Invocation.</p>

<p>The idea behind the system is to allow users to set their username, add a task, and mark it as in-progress or done. It's very simple, but it will help you better understand basic concepts from SDJ2 so far.</p>

[Full solution on Dominika's github](https://github.com/DominikaJanczyszyn/TaskApplication-Remote)

<p>The exercise consists of 15 steps. Let's get started.</p>

## Exercise

<p>Below is a UML of the classes needed. Please note the UML diagram may not be complete, and you're welcome to add to it as is needed.</p>

![image](https://github.com/OliwierWijas/OliwierWijas.github.io/assets/119060666/13dd760a-e702-4126-8f9a-7a7bf416047a)

##### Before starting remember to keep track of the module-info.java file in order to include important libraries and to export proper packages if needed!

##### Also remember that even though we give you full implementation of the needed classes, you can still ask questions if something is unclear to you, especially about the new topic: Remote Method Invocation.

#### Step 1 - the Connector remote interface

<p>In order to start working on a server we need to make an interface of it, in this case we call it - <code>Connector</code></p>
<p>Implement the <code>Connector</code> class from the class diagram. Besides basic methods that are needed for the logic of the system, we need the <code>addRemotePropertyChangeListener</code> method, which was done by Ole and allows for using the Observer pattern remotely. In order to be able to use the package that includes the RemoteObserver, you need to take it from your SDJ2 course and add to the project.</p>

<blockquote>
<details>
<summary>Display solution for the Connector interface</summary>
  
```java
public interface Connector extends Remote {
    ArrayList<Task> getTasks() throws RemoteException;
    void startTask(Task task) throws RemoteException;
    void finishTask(Task task) throws RemoteException;
    void addTask(Task task) throws RemoteException;
    void addRemotePropertyChangeListener(RemotePropertyChangeListener listener) throws RemoteException;
}
```

</details>
</blockquote>
