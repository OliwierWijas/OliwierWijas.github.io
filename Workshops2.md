# Workshops 2

<p>Hello everyone at the second session of Workshops!</p>
<p>Today we'll be focusing on MVVM pattern combined with the State and Observer patterns.</p>
<p>Today you will have one big exercise, which is about making a task management application.</p>
<p>Let's get started.</p>

## Exercise

<p>Below is a UML of the classes needed. Please note the UML diagram may not be complete, and you're welcome to add to it as is needed.</p>

![image](https://github.com/OliwierWijas/OliwierWijas.github.io/assets/119060666/1c7a581f-9b38-40dd-83e9-34ccdbbe0962)

##### Before starting remember to keep track of the module-info.java file in order to include important libraries and to export proper packages if needed!

##### Also remember that even though we give you full implementation of the needed classes, you can still ask questions if something is unclear to you, especially about the new topics including: the Observer pattern, the State pattern and the MVVM pattern.

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
<summary>Display solution for the Task class</summary>
  
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

<p>Here besides a basic class, we will combine it with the use of the Observer pattern. The Observer pattern will be used in order to send notifications to the View, informing that a change has happened, and to send an updated object to the upper layers of our architecture, so that the object with new data will be displayed.</p>

<p>Firstly, implement the Model interface with the methods given in the UML diagram. Remember that in interfaces our methods do not need access modifiers (e.g. public, protected, private).</p>

<p>After you have done the Model interface, make a ModelManager class, which implements the Model interface and the methods it has.</p>

<p>In order to make our Observer pattern to be working, we need a PropertyChangeSupport support object, that is responsible for informing observers about occuring changed, and two methods: addPropertyChangeListener() and removerPropertyChangeListener(), that are used in order to add or remove observers of the ModelManager class.</p>

###### Think about when firePropertyChange() method should be used and where it would be necessary to inform upper layers about possible changes (tip: it is usually necessary when we add, modify or remove data stored)!

<blockquote>
<details>
<summary>Display solution for the ModelManager interface and class</summary>

```java
public interface Model {
    void startTask(Task task);
    void finishTask(Task task);
    void addTask(Task task);
    void addPropertyChangeListener(PropertyChangeListener listener);
    void removePropertyChangeListener(PropertyChangeListener listener);
}
```
  
```java
public class ModelManager implements Model {
    private ArrayList<Task> tasks;
    private final PropertyChangeSupport support;

    public ModelManager() {
        this.tasks = new ArrayList<>();
        this.support = new PropertyChangeSupport(tasks);
    }

    @Override
    public synchronized void startTask(Task task) {
        try {
            task.startTask();
            support.firePropertyChange("List", null, tasks);

        } catch (Exception e) {
            throw new RuntimeException(e.getMessage());
        }
    }

    @Override
    public synchronized void finishTask(Task task) {
        try {
            task.finishTask();
            support.firePropertyChange("List", null, tasks);
        } catch (Exception e) {
            throw new RuntimeException(e.getMessage());
        }
    }

    @Override
    public synchronized void addTask(Task task) {
        tasks.add(task);
        support.firePropertyChange("List", null, tasks);

    }

    @Override
    public void addPropertyChangeListener(PropertyChangeListener listener) {
        support.addPropertyChangeListener(listener);
    }

    @Override
    public void removePropertyChangeListener(PropertyChangeListener listener) {
        support.removePropertyChangeListener(listener);
    }
}
```

</details>
</blockquote>

#### Step 5 - the StartViewModel class

<p>Right now we are moving to the first upper layer, which is the ViewModel layer. The ViewModel layer is a layer, which is responsible for connecting two other layers : Model and View, and that is why it combines both objects that can be usually seen in View and the ones from our Model.</p>

<p>We will start with the StartViewModel class. Maybe in order to better understand what is going on, you can at first open the StartView.fxml file given by us to visualize the thing we will be working on.</p>

<p>This class is responsible for taking the user username and storing it in the system, so that we can identify who enters new tasks. I would recommend to start with a class that is not on the diagram, but is pretty simple. It will be the User class, which we will use only to store the username of the current user. This class should only contain statis String called name.</p>

<blockquote>
<details>
<summary>Display solution for the Userclass</summary>

```java
public class User {
    public static String name;
}
```
</details>
</blockquote>

<p>Now, when we have a place where to store the username of the current user, we can progress into making a viewcontroller for the StartView.fxml. Therefore, we would need a name StringProperty. I don't know if these Properties are new for you, but you need to get to know with them, since they are widely used in SDJ2 course and I am sure you will need them in your SEP2 project. Those are basically objects that are used in View and allow us to convert user input into our own classes/objects.</p>

<p>Once the StringProperty name is added, we need to have another StringProperty message, which will be used to display any errors to the user. Lastly, we need a reference to the Model interface.</p>

<p>Make a simple constructor, where you intialize the Model object with the one given as a parameter and where you intialize StringProperty references with SimpleStringProperty empty objects.</p>

<p>When that is done, we are moving to the add method. Here you should check in try/catch, if the name property is empty, and if not then you should set the current user username with the name given. It should look like this: User.name = name.get();. Here since the name in the User class is static, we can initialize it without having a User object. Additionally, when using Properties you should always call method get() in order to get the actual value of the user input (without it you will get some object references, but it's a bit tricky and you don't need that to be happy.)</p>

<p>LASTLY, finally here, we make bind methods, which will allow us to bind the values of the properties with the information displayed to the user (thanks to that we ensure consistency between those values). Use bindBidirectional() method.</p>

<blockquote>
<details>
<summary>Display solution for the StartViewModel class</summary>

```java
public class StartViewModel {
    private StringProperty name;
    private StringProperty message;

    private Model model;

    public StartViewModel(Model model) {
        this.model = model;
        this.name = new SimpleStringProperty("");
        this.message = new SimpleStringProperty("");
    }

    public void add() {
        try {

            if (!name.get().isEmpty() && !name.get().equals("")) {
                Person person = new Person(name.get());
                User.name = person.getName();
            }
            else {
                message.set("Fill all of the fields!");
            }

        } catch (IllegalArgumentException e) {
            message.set("Wrong input!");
        }
    }

    public void bindName(StringProperty property) {
        property.bindBidirectional(name);
    }
    public void bindMessage(StringProperty property) {
        property.bindBidirectional(message);
    }
}
```
</details>
</blockquote>

#### Step 6 - the AddTaskViewModel class

<p>Here I won't talk that much here, since this class is very similar to the previous viewmodel (atleast the properties and the binding methods). As before, open the AddTaskView.fxml file to get to know what is expected to be done.</p>

<p>I will only focus on the add() method. In this method, once again, you should make a try/catch clause and within it, you should check if the title and the description properties are empty or null. If not, then create a Person object taking name for it from the User class (you can use just User.name). After that, you create a Task object with the newly created person object and the information taken from the properties. Lastly, you call a proper ModelManager method in order to add a task to its arrayList. In the end, set the message property to a proper information message for the user.</p>

<blockquote>
<details>
<summary>Display solution for the StartViewModel class</summary>

```java
public class AddTaskViewModel {
    private StringProperty title;
    private StringProperty description;
    private StringProperty message;

    private Model model;

    public AddTaskViewModel(Model model) {
        this.model = model;
        this.title = new SimpleStringProperty("");
        this.description = new SimpleStringProperty("");
        this.message = new SimpleStringProperty("");
    }

    public void add() {
        try {

            if (!title.get().isEmpty() && !title.get().equals("") && !description.get().isEmpty() && !description.get().equals("")){
                Person person = new Person(User.name);
                Task task = new Task(title.get(), description.get(), person);
                model.addTask(task);
                message.set("You added new Task: \n" + task);
            }
            else{
                message.set("Fill all of the fields!");
            }

        } catch (IllegalArgumentException e) {
            message.set("Wrong input!");
        }
    }
    public void bindTitle(StringProperty property) {
        property.bindBidirectional(title);
    }
    public void bindDescription(StringProperty property) {
        property.bindBidirectional(description);
    }
    public void bindMessage(StringProperty property) {
        property.bindBidirectional(message);
    }
}
```
</details>
</blockquote>

#### Step 7 - the ManageTasksViewModel class

<p>This viewmodel will be responsible for displaying our tasks as well as all other tasks in the system (look at ManageTasksView.fxml to understand better). Since, here the displayed data can change and we want our view to be always up to date, we need to continue the implementation of the observer pattern. Before the ModelManager class was the class that other ones listen to and now the ManageTasksViewModel will need to be a listener for that class, therefore make sure that it implements PropertyChangeListener interface and the propertyChange method.</p>

<p>Here also, I won't talk much about declaring the properties, just follow the UML diagram, it is very similar to the previous viewmodels.</p>

<p>Make a constructor, where you initalize all properties and remember to add this class as a listener to the model object in order to indicate that this class should listen to the fired events from the layer below.</p>

<p>For start and finish methods, once again in try/catch clause, you need to check if the properties are not null or empty (use get() function on them to get the value as I said before), and then call a proper method from the ModelManager class using the model object.</p>

<p>In binding, now do not use bidirectional binding, here we would want to unidirectionally bind our tasks lists to the given properties and for the bindSelected method, you need to bind the property to the task property declared in the beginning.</p>

<p>Here bindSelected() method might be something new for you. It does not need to be called like this, but I want to highlight how we will use it. Basically, the method is responsible for binding the object that we selected in the list in the view, so that the program will now on which object to run a method.</p>

<p>In the end, in the propertyChange() method, if the event name is "List" (as we declared in the Model), convert the object to an arrayList of tasks and add all tasks to the tasks ListProperty, additionally, add all tasks of the current user to the ownTasks ListProperty.</p>

###### Remember to use Platform.runLater(() -> {}) method in the propertyChange method in order to avoid concurrency (synchronization) issues between the application and the JavaFX thread. You don't need to fully understand it for now. Just use the given method and ask us, if you encounter any problems, since lambda expressions might be new for you. :)

<blockquote>
<details>
<summary>Display solution for the ManageTasksViewModel class</summary>

```java
public class ManageTasksViewModel implements PropertyChangeListener {
    private StringProperty message;
    private final ListProperty<Task> tasks;
    private final ListProperty<Task> ownTasks;
    private ObjectProperty<Task> task;
    private Model model;


    public ManageTasksViewModel(Model model) {
        this.model = model;
        this.message = new SimpleStringProperty("");
        this.tasks = new SimpleListProperty<>(FXCollections.observableArrayList());
        this.ownTasks = new SimpleListProperty<>(FXCollections.observableArrayList());
        this.task = new SimpleObjectProperty<>();
        model.addPropertyChangeListener(this);
    }

    public void startTask() {
        try {
            if (task.get() != null)
                model.startTask(task.get());
        } catch (Error e) {
            message.set(e.getMessage());
        }
    }
    public void finishTask(){
        try {
            if (task.get() != null)
                model.finishTask(task.get());
        } catch (Error e) {
            message.set(e.getMessage());
        }
    }

    public void bindTasks(ObjectProperty<ObservableList<Task>> property) {
        property.bind(tasks);
    }
    public void bindSelected(ReadOnlyObjectProperty<Task> property) {
        task.bind(property);
    }

    public void bindOwnTasks(ObjectProperty<ObservableList<Task>> property) {
        property.bind(ownTasks);
    }

    @Override
    public void propertyChange(PropertyChangeEvent evt) {
        Platform.runLater(() -> { if (evt.getPropertyName().equals("List"))
        {
            ArrayList<Task> temp = ((ArrayList<Task>)evt.getNewValue());
            tasks.clear();
            temp.forEach(task -> tasks.add(task));
            ownTasks.clear();
            temp.forEach(task -> {
                if(task.getPerson().getName().equals(User.name))
                    ownTasks.add(task);
            });
        }
        });
    }
}
```
</details>
</blockquote>

#### Step 8 - the ViewModelFactory class

<p>This class is responsible for making sure there is only one ViewModel of each type in the system. You need to have a reference to each of the viewmodels created, initialize them in the constructor and provide getter methods that we will use in order to access those ViewModel objects.</p>

<blockquote>
<details>
<summary>Display solution for the ViewModelFactory class</summary>

```java

```
</details>
</blockquote>

#### Step 9 - the ViewFactory class

<p>Now we are moving on to the View layer. We will start with the ViewFactory class.</p>

<p>Here also you do not need to fully understand this class. I remember I was struggling with it on the second semester. You can try to write it on your own looking at Ole's examples and by trying to figure out which views will be used, if not you can simply copy the class from down below.</p>

###### You can have an error, because the ViewHandler as well as controllers class are not yet created, we will fix it in the next steps.

<blockquote>
<details>
<summary>Display solution for the ViewFactory class</summary>

```java
public class ViewFactory {
    public static final String ADD = "add";
    public static final String MANAGE = "manage";
    public static final String START = "start";

    private final ViewHandler viewHandler;
    private final ViewModelFactory viewModelFactory;
    private StartViewController startViewController;
    private ManageTasksViewController manageTasksViewController;
    private AddTaskViewController addTaskViewController;

    public ViewFactory(ViewHandler viewHandler, ViewModelFactory viewModelFactory) {
        this.viewHandler = viewHandler;
        this.viewModelFactory = viewModelFactory;
        this.startViewController = null;
        this.manageTasksViewController = null;
        this.addTaskViewController = null;
    }
    public Region loadStartView() {
        if (startViewController == null) {
            FXMLLoader loader = new FXMLLoader();
            loader.setLocation(getClass().getResource("/com/example/workshops2session2/StartView.fxml"));
            try {
                Region root = loader.load();
                startViewController = loader.getController();
                startViewController.init(viewHandler, viewModelFactory.getStartViewModel(), root);
            } catch (IOException e) {
                throw new IOError(e);
            }
        }
        startViewController.reset();
        return startViewController.getRoot();
    }
    public Region loadAddTaskView() {
        if (addTaskViewController == null) {
            FXMLLoader loader = new FXMLLoader();
            loader.setLocation(getClass().getResource("/com/example/workshops2session2/AddTaskView.fxml"));
            try {
                Region root = loader.load();
                addTaskViewController = loader.getController();
                addTaskViewController.init(viewHandler, viewModelFactory.getAddTaskViewModel(), root);
            } catch (IOException e) {
                throw new IOError(e);
            }
        }
        addTaskViewController.reset();
        return addTaskViewController.getRoot();
    }
    public Region loadManageVinylsView() {
        if (manageTasksViewController == null) {
            FXMLLoader loader = new FXMLLoader();
            loader.setLocation(getClass().getResource("/com/example/workshops2session2/ManageTasksView.fxml"));
            try {
                Region root = loader.load();
                manageTasksViewController = loader.getController();
                manageTasksViewController.init(viewHandler, viewModelFactory.getManageTasksViewModel(), root);
            } catch (IOException e) {
                throw new IOError(e);
            }
        }
        manageTasksViewController.reset();
        return manageTasksViewController.getRoot();
    }


    public Region loadView(String id) {
        return switch (id) {
            case START -> loadStartView();
            case ADD -> loadAddTaskView();
            case MANAGE -> loadManageVinylsView();
            default -> throw new IllegalArgumentException("Unknown view: " + id);
        };
    }
}
```
</details>
</blockquote>

#### Step 10 - the ViewHandler class

<p>The same as in the ViewFactory case, you do not need to fully understand the code, just copy it and don't worry about it. This class is responsible for changing scenes for example when a button is clicked etc.</p>

<blockquote>
<details>
<summary>Display solution for the ViewFactory class</summary>

```java
public class ViewHandler {
    private final Scene currentScene;
    private Stage primaryStage;
    private final ViewFactory viewFactory;

    public ViewHandler(ViewModelFactory viewModelFactory) {
        this.viewFactory = new ViewFactory(this, viewModelFactory);
        this.currentScene = new Scene(new Region());
    }

    public void start(Stage primaryStage) {
        this.primaryStage = primaryStage;
        openView(ViewFactory.START);
    }

    public void openView(String id) {
        Region root = viewFactory.loadView(id);
        currentScene.setRoot(root);
        if (root.getUserData() == null) {
            primaryStage.setTitle("");
        } else {
            primaryStage.setTitle(root.getUserData().toString());
        }
        primaryStage.setScene(currentScene);
        primaryStage.sizeToScene();
        primaryStage.show();
    }

    public void closeView() {
        primaryStage.close();
    }
}
```
</details>
</blockquote>

#### Step 11 - the StartViewController class

<p>Here we need to fields with FXML annotation (@FXML before accessor annotation), those are TextField name and Label message. Additionally, as in every controller, we need a Region object called root, a ViewHandler and a StartViewModel objects.</p>

<p>We need an init() method, which works kinda like a constructor (it initalizes view, once the application is started). Inside of it, you should initalize all the objects and bind the FXML objects with the ones in viewModel by using the bind methods that we implemented priorly. (Tip: do not bind objects, but their values by using textProperty() method on them.)</p>

<p>After this we need a onOk() method (also with FXML annotation, since we will use it on a button). This method should call a method add() from the viewModel and change the view to AddTaskView.</p>

<p>The reset method should set the FXML fields to an empty string.</p>

<p>The getRoot() method should return the root.</p>

<blockquote>
<details>
<summary>Display solution for the ViewFactory class</summary>

```java
public class StartViewController {
    @FXML
    public TextField name;
    @FXML public Label message;
    private Region root;
    private ViewHandler viewHandler;
    private StartViewModel viewModel;

    public void init(ViewHandler viewHandler, StartViewModel viewModel, Region root) {
        this.root = root;
        this.viewHandler = viewHandler;
        this.viewModel =viewModel;
        message.setText("");
        name.setText("");

        viewModel.bindName(name.textProperty());
        viewModel.bindMessage(message.textProperty());

    }

    @FXML
    public void onOK() {
        viewModel.add();
        viewHandler.openView(ViewFactory.ADD);

    };

    public void reset() {
        message.setText("");
        name.setText("");
    }

    public Region getRoot() {
        return root;
    }
}
```
</details>
</blockquote>
