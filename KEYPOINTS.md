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

## Part 4: Using FXML

### Separate the view from its controller

FXML is XML that declares the scene graph. Java controller classes hold the
behavior. This separation makes layout changes easier to find and keeps
`Main` from becoming responsible for everything.

```xml
<TextField fx:id="userInput"
           onAction="#handleUserInput" />
```

The same information previously written as Java setters can be expressed as
attributes. `fx:id` links the node to a controller field, while `#` means
"call this controller method".

### `@FXML` preserves encapsulation

FXMLLoader needs access to fields and methods named by the FXML document.
Annotating them lets them remain private:

```java
@FXML
private TextField userInput;

@FXML
private void handleUserInput() {
    String input = userInput.getText();
}
```

Without `@FXML`, those members would need wider visibility. The annotation
therefore connects the files without exposing implementation details to the
rest of the program.

### The loading sequence

`FXMLLoader` performs several jobs:

1. Read the FXML resource.
2. Construct the declared JavaFX nodes.
3. Create the controller named by `fx:controller`.
4. Inject matching `fx:id` nodes into `@FXML` fields.
5. Call the controller's `initialize()` method.

```java
FXMLLoader loader = new FXMLLoader(
        Main.class.getResource("/view/MainWindow.fxml"));
AnchorPane root = loader.load();
MainWindow controller = loader.getController();
controller.setDuke(duke);
```

`setDuke` is dependency injection in a simple form: `Main` supplies the
chatbot object that the newly created controller needs.

### Two ways to connect FXML and controllers

`MainWindow.fxml` names its controller using `fx:controller`. FXMLLoader
constructs both the nodes and controller.

`DialogBox.fxml` instead uses `fx:root`. The `DialogBox` constructor already
has the Java root/controller instance, so it supplies both explicitly:

```java
fxmlLoader.setController(this);
fxmlLoader.setRoot(this);
fxmlLoader.load();
```

Use `fx:root` for a reusable custom component whose Java class extends its
root node type.

### Binding versus listening

Part 3 listened for height changes and assigned a scroll value. Part 4 binds
the values instead:

```java
scrollPane.vvalueProperty().bind(dialogContainer.heightProperty());
```

A binding keeps one property derived from another automatically. A listener
runs arbitrary code when a value changes. Use binding for a continuing value
relationship and listeners for side effects.

### Scene Builder

Scene Builder is a visual editor for FXML, not a different UI system. Inspect
its generated FXML rather than treating it as magic. If it changes the
`xmlns` JavaFX version, restore it to the runtime version used by this
tutorial (`17`) to avoid version warnings.

## Part 5: Tweaking the GUI

### Make layouts responsive with constraints

Preferred sizes alone describe an initial or desired size. Anchor constraints
describe how a child should respond when its parent changes size:

```xml
<TextField AnchorPane.leftAnchor="0.0"
           AnchorPane.rightAnchor="76.0"
           AnchorPane.bottomAnchor="1.0" />
```

Anchoring both horizontal edges lets the text field grow horizontally while
leaving 76 pixels for the button. The scroll pane is anchored on all four
edges, so it grows in both directions. `fitToWidth="true"` also resizes its
content to the viewport width.

Minimum stage dimensions prevent resizing the interface until controls become
unusable:

```java
stage.setMinHeight(220);
stage.setMinWidth(417);
```

### Keep appearance in CSS

FXML defines structure, controller classes define behavior, and CSS defines
appearance. Link a stylesheet from FXML using a resource-relative URL:

```xml
<AnchorPane stylesheets="@../css/main.css">
```

JavaFX CSS resembles browser CSS but its properties usually begin with
`-fx-`. The most useful selectors are:

```css
.button { }                 /* Every node with class "button". */
#displayPicture { }         /* The node with this ID. */
.button:hover { }           /* A class in a temporary state. */
.scroll-pane .viewport { }  /* A descendant inside another node. */
```

Built-in controls already have style classes such as `.button`, `.label`, and
`.text-field`. `fx:id="displayPicture"` also provides the `#displayPicture`
CSS ID used in this tutorial.

### Reuse colours and represent interaction states

A looked-up colour is a named CSS value inherited through the scene graph:

```css
.root {
    main-color: rgb(237, 255, 242);
    -fx-background-color: main-color;
}
```

Pseudo-classes style temporary states without Java event handlers:

```css
.button:hover {
    -fx-background-color: cyan;
}

.button:pressed {
    -fx-background-color: orange;
}
```

Use CSS for visual feedback and Java handlers for actual behavior.

### Understand spacing and borders

- Padding is space between content and its border.
- Margin is space outside the border.
- JavaFX CSS has no direct margin property. Matching background and border
  insets can simulate that outside spacing.

```css
.label {
    -fx-padding: 6px;
    -fx-border-insets: 0 7px 0 7px;
    -fx-background-insets: 0 7px 0 7px;
}
```

Four values are read clockwise as top, right, bottom, left. Radius values use
the same order for the four corners. Giving Duke's label a separate
`.reply-label` class lets its bubble point in the opposite direction.

### Add and remove style classes dynamically

Controllers can choose a visual state at runtime:

```java
if (isError) {
    dialog.getStyleClass().add("error-label");
}
```

That is the reusable idea behind Part 5's command-specific colours. The
tutorial starter only echoes text and has no command classes, so command-type
styling was intentionally not copied here. In the iP, the parser or command
result should provide the semantic state; the GUI should only select the
matching class.

### Background images are ordinary CSS resources

A background image can be resolved relative to the CSS file:

```css
.root {
    -fx-background-image: url("../images/background.jpg");
    -fx-background-size: cover;
}
```

`cover` preserves the aspect ratio while covering the region; `stretch`
distorts the image to fit. Repetition can be controlled per axis. No
background image was added because Part 5 does not supply one and the choice
is purely cosmetic.

### Implemented scope

This project implements every generally applicable Part 5 tweak: responsive
anchoring, minimum window dimensions, linked stylesheets, colours, borders,
padding/insets, directional bubbles, hover/pressed states, and image shadows.
Only command-specific styling and a user-chosen background image were omitted
because the tutorial starter has neither commands nor a supplied background.
