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

#### Step 1 - the Connector Remote Interface

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

### Step 2 - The RemoteConnector Class

<p>Right now we need to implement the previously created interface and the methods it includes. Do that based on the class diagram.</p>
<p>Remember to use the RemotePropertyChangeSupport object in order to fire events to notify clients about necessary data updates.</p>

<blockquote>
<details>
<summary>Display solution for the Connector interface</summary>
  
```java
public class RemoteConnector implements Connector {
    private final ArrayList<Task> tasks;
    private final RemotePropertyChangeSupport support;

    public RemoteConnector(){
        this.tasks = new ArrayList<>();
        this.support = new RemotePropertyChangeSupport();
    }
    @Override
    public ArrayList<Task> getTasks() throws RemoteException {
        return tasks;
    }

    @Override
    public void startTask(Task task) throws RemoteException {
        for (int i = 0; i < tasks.size(); i++)
        {
            if (tasks.get(i).equals(task)) {
                tasks.get(i).startTask();
                break;
            }
        }
        this.support.firePropertyChange("List", null, tasks);
    }

    @Override
    public void finishTask(Task task) throws RemoteException {
        for (int i = 0; i < tasks.size(); i++)
        {
            if (tasks.get(i).equals(task)) {
                tasks.get(i).finishTask();
                break;
            }
        }
        this.support.firePropertyChange("List", null, tasks);
    }

    @Override
    public void addTask(Task task) throws RemoteException {
        tasks.add(task);
        this.support.firePropertyChange("List", null, tasks);
    }

    @Override
    public void addRemotePropertyChangeListener(RemotePropertyChangeListener listener) throws RemoteException {
        this.support.addPropertyChangeListener(listener);
    }
}
```

</details>
</blockquote>

### Step 3 - Implementing the ServerStart Class

<p>We have implemented all the essential functionalities within the server domain. Now, let's implement the class responsible for starting the server - <code>ServerStart</code>.</p>

<p>To start the RMI server, we need to:</p>

1. Create a <code>Registry</code> at a specified port number.
2. Make a <code>RemoteConnector</code> object.
3. Initialize a new <code>Remote</code> object by calling the <code>UnicastRemoteObject.exportObject()</code> method with the previously created <code>RemoteConnector</code> object and 0 port number as parameters.
4. Bind the <code>Remote</code> object with a given name in the registry.
5. (Optionally) Print out that the server is running.

<blockquote>
<details>
<summary>Display solution for the Connector interface</summary>
  
```java
public class ServerStart
{
    public static void main(String[] args) throws RemoteException, AlreadyBoundException
    {
        Registry registry = LocateRegistry.createRegistry(8080);
        RemoteConnector remoteConnector = new RemoteConnector();
        Remote remote = UnicastRemoteObject.exportObject(remoteConnector, 0);
        registry.bind("rmiServer", remote);
        System.out.println("Server running");
    }
}
```

</details>
</blockquote>
