# Kivy Layout Documentation (`car.kv`)

This file uses the Kivy language to define the visual appearance and structure of the widgets used in the self-driving car simulation. The Kivy language provides a declarative way to build user interfaces.

```kivy
#:kivy 1.0.9
# ref: https://kivy.org/docs/tutorials/pong.html
```

-   `#:kivy 1.0.9`: Specifies the version of Kivy this file is intended for.
-   The comment `# ref: ...` suggests that some concepts might have been adapted from the Kivy Pong tutorial.

## Widget Styling Rules

### `<Car>:`

This rule styles the `Car` widget defined in `map.py`.

```kivy
<Car>:
    size: 20, 10
    canvas:
        PushMatrix
        Rotate:
            angle: self.angle
            origin: self.center
        Rectangle:
            pos: self.pos
            size: self.size
        PopMatrix
```

-   `size: 20, 10`: Sets the default size of the car widget to 20 pixels wide and 10 pixels high.
-   `canvas:`: Defines custom drawing instructions for the widget.
    -   `PushMatrix`: Saves the current graphics transformation matrix. This is important so that the rotation applied next doesn't affect other elements on the canvas.
    -   `Rotate:`: Applies a rotation transformation.
        -   `angle: self.angle`: The angle of rotation is bound to the `angle` property of the `Car` widget instance. When `car.angle` changes in the Python code, the visual representation will update.
        -   `origin: self.center`: The rotation is performed around the center of the `Car` widget.
    -   `Rectangle:`: Draws a rectangle.
        -   `pos: self.pos`: The position of the rectangle is bound to the `pos` property of the `Car` widget.
        -   `size: self.size`: The size of the rectangle is bound to the `size` property of the `Car` widget (which is 20x10 as defined above, but could be overridden).
    -   `PopMatrix`: Restores the previously saved graphics transformation matrix, undoing the rotation for subsequent drawing operations.

### `<Ball1>:, <Ball2>:, <Ball3>:`

These rules style the `Ball1`, `Ball2`, and `Ball3` widgets, which represent the car's sensors.

**`<Ball1>:` (Center Sensor - Red)**
```kivy
<Ball1>:
    size: 10,10
    canvas:
        Color:
            rgba: 1,0,0,1  # Red color (R,G,B,A)
        Ellipse:
            pos: self.pos
            size: self.size
```

**`<Ball2>:` (Front-Left Sensor - Cyan)**
```kivy
<Ball2>:
    size: 10,10
    canvas:
        Color:
            rgba: 0,1,1,1  # Cyan color (R,G,B,A)
        Ellipse:
            pos: self.pos
            size: self.size
```

**`<Ball3>:` (Front-Right Sensor - Yellow)**
```kivy
<Ball3>:
    size: 10,10
    canvas:
        Color:
            rgba: 1,1,0,1  # Yellow color (R,G,B,A)
        Ellipse:
            pos: self.pos
            size: self.size
```

-   `size: 10,10`: Sets the default size of each ball (sensor) widget to 10x10 pixels.
-   `canvas:`: Defines custom drawing instructions.
    -   `Color:`: Sets the drawing color.
        -   `rgba: 1,0,0,1`: Red for `Ball1`.
        -   `rgba: 0,1,1,1`: Cyan for `Ball2`.
        -   `rgba: 1,1,0,1`: Yellow for `Ball3`.
        (The four values are Red, Green, Blue, and Alpha/Opacity).
    -   `Ellipse:`: Draws an ellipse (which will be a circle since width and height are equal).
        -   `pos: self.pos`: The position is bound to the `pos` property of the respective Ball widget.
        -   `size: self.size`: The size is bound to the `size` property of the respective Ball widget.

### `<Game>:`

This rule defines properties and the layout for the `Game` widget, which is the root widget for the simulation visuals within the app window (excluding the paint widget and buttons which are added directly in Python).

```kivy
<Game>:
    car: game_car
    ball1: game_ball1
    ball2: game_ball2
    ball3: game_ball3

    Car:
        id: game_car
        center: self.parent.center
    Ball1:
        id: game_ball1
        center: self.parent.center
    Ball2:
        id: game_ball2
        center: self.parent.center
    Ball3:
        id: game_ball3
        center: self.parent.center
```

-   Property Assignments:
    -   `car: game_car`: Links the `car` ObjectProperty of the `Game` instance (defined in `map.py`) to the `Car` widget instance that has `id: game_car`.
    -   `ball1: game_ball1`, `ball2: game_ball2`, `ball3: game_ball3`: Similarly links the `ball1`, `ball2`, and `ball3` ObjectProperties of the `Game` instance to their respective `Ball` widget instances declared below.

-   Child Widgets:
    -   `Car:`: Declares an instance of the `Car` widget as a child of the `Game` widget.
        -   `id: game_car`: Assigns an ID to this `Car` instance, allowing it to be referenced (e.g., by `Game.car` property).
        -   `center: self.parent.center`: Initially centers the car within its parent (`Game`) widget.
    -   `Ball1:`, `Ball2:`, `Ball3:`: Declare instances of `Ball1`, `Ball2`, and `Ball3` widgets as children of the `Game` widget.
        -   `id: game_ballX`: Assigns IDs to these ball instances.
        -   `center: self.parent.center`: Initially centers these balls within their parent (`Game`) widget. Their positions will be dynamically updated by the `Game.update` method in `map.py` to follow the car's sensors.

## Summary

The `car.kv` file is crucial for defining how the `Car` and its `Sensor` (Ball) widgets are drawn and how they are initially structured within the `Game` widget. It leverages Kivy's binding capabilities to ensure that changes to widget properties in Python code are automatically reflected in their visual representation. 