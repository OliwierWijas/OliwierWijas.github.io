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

#### Step 5 - the Connector Remote Interface

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
3. Initialize a new <code>Remote</code> object by calling the <code>UnicastRemoteObject.exportObject()</code> method with the previously created <code>RemoteConnector</code> object and 0 port number given as parameters.
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

### Step 4 - Implementing the Client Class

<p>Our <code>Client</code> interface that we implemented some weeks ago can stay the same, since we still need the same methods, only their implementation will change.</p>

<p>Our <code>ClientImpl</code> class needs to extend the <code>UnicastRemoteObject</code> class and implement both the <code>Client</code> and <code>RemotePropertyChangeListener</code> interfaces (the second one enables the <code>ClientImpl</code> class to get notified about fired events from the server over the network).</p>

<p>Now our client class takes the <code>Connector</code> server interface as a parameter, so that it can use it as a local object, even though they can be located on the other sides of the world.</p>

<p>Implement the methods that the <code>ClientImpl</code> class needs to have. It is very simple, since in most cases you only call appropriate methods on the remote <code>Connector</code> object as if they were local.</p>

<p>Remember to appropriately react to fired events and make sure to inform the upper layers of the client side about them. Don't forget to use <code>Platform.runLater()</code> method, which is needed for making observing adaptable with JavaFX.</p>

<blockquote>
<details>
<summary>Display solution for the Connector interface</summary>
  
```java
public class ClientImpl extends UnicastRemoteObject implements Client , RemotePropertyChangeListener
{
  private final Connector connector;
  private final PropertyChangeSupport support;


    public ModelManager(Client client) throws RemoteException {
        this.client = client;
        this.client.addPropertyChangeListener(this);
        this.support = new PropertyChangeSupport(this);
    }

  @Override public ArrayList<Task> getTasks() throws RemoteException
  {
    try{
      return connector.getTasks();
    }catch (Exception e){
      throw new IllegalArgumentException(e);
    }
  }

  @Override public void startTask(Task task) throws RemoteException
  {
    try{
      connector.startTask(task);
    }catch (Exception e){
      throw new IllegalArgumentException(e);
    }
  }

  @Override public void finishTask(Task task) throws RemoteException
  {
    try{
      connector.finishTask(task);
    }catch (Exception e){
      throw new IllegalArgumentException(e);
    }
  }

  @Override public void addTask(Task task) throws RemoteException
  {
    try{
      connector.addTask(task);
    }catch (Exception e){
      throw new IllegalArgumentException(e);
    }
  }

  @Override public void addPropertyChangeListener (
      PropertyChangeListener listener)
  {
    this.support.addPropertyChangeListener(listener);
  }

  @Override
  public void propertyChange(RemotePropertyChangeEvent event) throws RemoteException {
    Platform.runLater(()->{
      if (event.getPropertyName().equals("List"))
        this.support.firePropertyChange("List", null, event.getNewValue());
    });
  }
}
```

</details>
</blockquote>

### Step 5 - Modifying the StartApplication Class

<p>To start the client instead of what we did previously, we need to:</p>

1. Get a <code>Registry</code> from the specified on the server side port number.
2. Look up for a remote object in the registry with the bound name and cast it to a <code>Connector</code>.
3. Create a <code>Client</code> object with the created <code>Connector</code> object given as a parameter.
4. Initialize the rest of the application as before, but with the new client given as a parameter.

<blockquote>
<details>
<summary>Display solution for the Connector interface</summary>
  
```java
public class StartApplication extends Application {
    @Override
    public void start(Stage stage) throws IOException, NotBoundException {
        Registry registry = LocateRegistry.getRegistry(8080);
        Connector connector = (Connector) registry.lookup("rmiServer");
        Client client = new ClientImpl(connector);
        Model model = new ModelManager(client);
        ViewModelFactory viewModelFactory = new ViewModelFactory(model);
        ViewHandler viewHandler = new ViewHandler(viewModelFactory);
        viewHandler.start(stage);
    }

    public static void main(String[] args) {
        launch();
    }
}
```

</details>
</blockquote>
