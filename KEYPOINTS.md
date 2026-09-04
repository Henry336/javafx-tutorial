# JavaFX Tutorial Key Points

## Part 1: Getting Started

### The scene graph

JavaFX arranges visible objects as a tree called the **scene graph**:

```text
Stage (the window)
└── Scene (the window's content)
    └── Root Node
        └── Other Nodes, such as labels and buttons
```

- A `Stage` is a top-level window.
- A `Scene` contains the UI shown in that window.
- A `Node` is any item in the scene graph, such as a `Label`, button, or
  layout pane.

The smallest useful JavaFX setup is:

```java
Label message = new Label("Hello World!");
Scene scene = new Scene(message);
stage.setScene(scene);
stage.show();
```

### Application lifecycle

The GUI class extends `Application`. JavaFX creates it and calls its
`start(Stage)` method:

```java
public class Main extends Application {
    @Override
    public void start(Stage stage) {
        // Build and show the interface here.
    }
}
```

JavaFX needs to construct this class without arguments. If you later add a
constructor with parameters, also provide a no-argument constructor.

### Why `Launcher` exists

Keep an ordinary Java class as the Gradle entry point:

```java
public static void main(String[] args) {
    Application.launch(Main.class, args);
}
```

Point `application.mainClass` at `Launcher`. This avoids a JavaFX classpath
launching issue while keeping `Main` focused on the GUI.

### Dependencies

JavaFX is not bundled with modern JDKs. Gradle downloads it from Maven
Central. The tutorial includes Windows, macOS, and Linux variants so the
same repository can run on each operating system.

## Part 2: Creating the GUI

### Controls and layout panes have different jobs

- Controls are interactive or visible widgets: `Button`, `TextField`,
  `Label`, `ImageView`, and `ScrollPane`.
- Layout panes arrange child nodes. `VBox` stacks nodes vertically, `HBox`
  places them horizontally, and `AnchorPane` pins nodes to its edges.

Example hierarchy used by the chatbot:

```text
AnchorPane
├── ScrollPane
│   └── VBox
│       └── DialogBox (HBox)
│           ├── Label
│           └── ImageView
├── TextField
└── Button
```

`ScrollPane` accepts one content node, so a `VBox` is used as that one node
and holds any number of dialog boxes.

### Reusable custom controls

`DialogBox extends HBox` packages a message and avatar into one reusable
node:

```java
public class DialogBox extends HBox {
    public DialogBox(String message, Image image) {
        Label text = new Label(message);
        ImageView picture = new ImageView(image);
        getChildren().addAll(text, picture);
    }
}
```

Inheritance is useful here because a `DialogBox` genuinely behaves like an
`HBox`; it can be inserted anywhere that accepts a JavaFX node.

### Resources and sizing

Files under `src/main/resources` are loaded from the classpath. A leading
slash makes the path relative to that resources root:

```java
new Image(getClass().getResourceAsStream("/images/DaUser.png"));
```

Preferred sizes guide JavaFX's layout calculation. Anchors position nodes
relative to an `AnchorPane` edge. They are layout instructions, not fixed
screen coordinates.

## Part 3: Interacting with the User

### JavaFX is event-driven

The program does not repeatedly ask whether a button was clicked. Instead,
you register a handler, and JavaFX calls it when the event happens:

```java
sendButton.setOnMouseClicked(event -> handleUserInput());
userInput.setOnAction(event -> handleUserInput());
```

For a `TextField`, an action occurs when the user presses Enter. Both events
call one method so the behavior stays consistent and is not duplicated.

### Keep application logic separate from GUI logic

`Main` collects input and displays output. `Duke` decides what response to
return:

```java
String userText = userInput.getText();
String dukeText = duke.getResponse(userText);
```

This boundary matters in the iP: your GUI should call the existing chatbot
logic instead of copying command parsing or task operations into UI classes.

### Properties can be observed

Many JavaFX values are observable properties. A listener runs whenever the
observed value changes:

```java
dialogContainer.heightProperty().addListener(
        observable -> scrollPane.setVvalue(1.0));
```

Adding a message changes the `VBox` height, so the listener scrolls to the
bottom. This is reactive behavior: describe what should happen *when a value
changes* instead of manually checking it continuously.

### Lambdas are short handler implementations

In `event -> handleUserInput()`, the value before `->` is the event supplied
by JavaFX, and the expression after it is the code to execute. Use a block
when more than one statement is needed:

```java
event -> {
    handleUserInput();
    userInput.requestFocus();
}
```

### Factory methods express intent

These calls are clearer than creating a generic dialog box and remembering
whether to flip it:

```java
DialogBox.getUserDialog(userText, userImage);
DialogBox.getDukeDialog(dukeText, dukeImage);
```

The public factory methods choose the correct arrangement. The private
`flip()` method hides the implementation detail.

The tutorial uses `var db = new DialogBox(...)`. `var` asks Java to infer the
local variable's type from the right-hand side; `db` is still statically a
`DialogBox`. It works only for local variables with an initializer, not for
fields or method parameters.
