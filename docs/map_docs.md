# Simulation Environment Documentation (`map.py`)

This file sets up and runs the self-driving car simulation environment using the Kivy framework. It defines the car, its sensors, the game logic, user interaction for drawing obstacles, and the main application class.

## Global Variables and Initialization

-   `last_x`, `last_y`, `n_points`, `length`: Used for drawing sand/obstacles on the map.
-   `brain`: An instance of the `Dqn` class from `ai.py`, representing the car's AI. Initialized with `Dqn(5, 3, 0.9)` (5 inputs, 3 actions, gamma=0.9).
-   `action2rotation`: A list mapping action indices (0, 1, 2) to rotation angles `[0, 20, -20]` degrees.
-   `last_reward`: Stores the reward from the previous step.
-   `scores`: A list to keep track of the agent's scores over time.
-   `first_update`: A boolean flag to ensure `init()` is called only once.
-   `sand`: A NumPy array representing the map, where 1 indicates sand (obstacle) and 0 indicates a clear path.
-   `goal_x`, `goal_y`: Coordinates of the current target goal for the car.
-   `last_distance`: The car's distance to the goal in the previous step.
-   `longueur`, `largeur`: Width and height of the game window (updated dynamically).

-   **`init()` function**:
    -   Initializes/resets the `sand` array to all zeros.
    -   Sets the initial `goal_x` and `goal_y` (near the top-left of the window initially).
    -   Sets `first_update` to `False`.

## Kivy Configuration

-   `Config.set('input', 'mouse', 'mouse,multitouch_on_demand')`: Configures Kivy to handle mouse input and allow multitouch on demand, preventing the default right-click red dot.

## Classes

### `Car(Widget)`

Represents the self-driving car.

-   **Inherits from**: `kivy.uix.widget.Widget`
-   **Properties (Kivy NumericProperty and ReferenceListProperty)**:
    -   `angle`: Current angle of the car.
    -   `rotation`: Change in angle to be applied.
    -   `velocity_x`, `velocity_y`, `velocity`: Components and vector of the car's velocity.
    -   `sensor1_x`, `sensor1_y`, `sensor1`: Position of the first sensor (center front).
    -   `sensor2_x`, `sensor2_y`, `sensor2`: Position of the second sensor (front-left, +30 deg).
    -   `sensor3_x`, `sensor3_y`, `sensor3`: Position of the third sensor (front-right, -30 deg).
    -   `signal1`, `signal2`, `signal3`: Density of sand detected by each sensor (0 to 1).

-   **`move(self, rotation)`**:
    -   Updates the car's position based on its `velocity`.
    -   Sets the car's `rotation` and updates its `angle`.
    -   Calculates the new positions of the three sensors based on the car's current position and angle.
    -   Calculates `signal1`, `signal2`, `signal3`:
        -   For each sensor, it sums the values in a 20x20 area of the `sand` array around the sensor's tip.
        -   Divides the sum by 400 (20*20) to get an average density (0 to 1).
        -   If a sensor goes off the map boundaries (less than 10 pixels from an edge), its corresponding signal is set to `1.` (maximum obstacle signal).

### `Ball1(Widget)`, `Ball2(Widget)`, `Ball3(Widget)`

Simple Kivy widgets used to visually represent the car's sensors on the map. Their appearance is defined in `car.kv`.

### `Game(Widget)`

Manages the game state, updates, and interactions between the car and the AI.

-   **Inherits from**: `kivy.uix.widget.Widget`
-   **Properties (Kivy ObjectProperty)**:
    -   `car`: Reference to the `Car` widget instance.
    -   `ball1`, `ball2`, `ball3`: References to the sensor `Ball` widget instances.

-   **`serve_car(self)`**:
    -   Initializes the car's position to the center of the game window.
    -   Sets the car's initial `velocity` to `Vector(6, 0)` (moving right).

-   **`update(self, dt)`**:
    -   This method is scheduled to be called repeatedly by Kivy's `Clock` (typically 60 times per second).
    -   `dt` (float): Time elapsed since the last update (delta time).
    -   Updates global `longueur` (width) and `largeur` (height) from `self.width` and `self.height`.
    -   Calls `init()` if `first_update` is true.
    -   **State Preparation for AI**:
        -   `xx`, `yy`: Vector components from the car to the goal.
        -   `orientation`: Angle between the car's velocity vector and the vector to the goal, normalized to [-1, 1]. (`Vector(*self.car.velocity).angle((xx,yy))/180.`)
        -   `last_signal`: A list containing `[self.car.signal1, self.car.signal2, self.car.signal3, orientation, -orientation]`. This is the state input for the DQN.
    -   **AI Interaction**:
        -   `action = brain.update(last_reward, last_signal)`: Sends the current `last_reward` and `last_signal` (new state) to the DQN agent. The agent learns and returns a new `action`.
        -   `scores.append(brain.score())`: Appends the current average score of the AI to the `scores` list.
        -   `rotation = action2rotation[action]`: Converts the AI's action index to a rotation angle.
    -   **Car Movement and Sensor Update**:
        -   `self.car.move(rotation)`: Moves the car according to the chosen rotation.
        -   `distance = np.sqrt((self.car.x - goal_x)**2 + (self.car.y - goal_y)**2)`: Calculates the new distance to the goal.
        -   Updates the positions of `ball1`, `ball2`, `ball3` to match the car's sensor positions.
    -   **Reward Calculation (`last_reward`)**:
        -   If the car is on sand (`sand[int(self.car.x),int(self.car.y)] > 0`):
            -   Reduces car velocity: `self.car.velocity = Vector(1, 0).rotate(self.car.angle)`.
            -   `last_reward = -1` (penalty for hitting sand).
        -   Else (car is on a clear path):
            -   Maintains normal velocity: `self.car.velocity = Vector(6, 0).rotate(self.car.angle)`.
            -   `last_reward = -0.2` (small penalty for not necessarily progressing or to encourage shorter paths).
            -   If `distance < last_distance` (car moved closer to the goal):
                -   `last_reward = 0.1` (positive reward).
    -   **Boundary Collision Handling**:
        -   If the car hits any of the four window boundaries (x < 10, x > width-10, y < 10, y > height-10):
            -   The car's position is reset to be just inside the boundary.
            -   `last_reward = -1` (penalty for hitting a wall).
    -   **Goal Reaching and Reset**:
        -   If `distance < 100` (car is close to the goal):
            -   `goal_x = self.width - goal_x`
            -   `goal_y = self.height - goal_y`
            -   The goal is effectively moved to the opposite side of the map relative to its previous position.
    -   `last_distance = distance`: Updates `last_distance` for the next iteration.

### `MyPaintWidget(Widget)`

Allows the user to draw obstacles (sand) on the map.

-   **Inherits from**: `kivy.uix.widget.Widget`

-   **`on_touch_down(self, touch)`**:
    -   Called when the mouse button is pressed down on the widget.
    -   Sets the drawing color to yellow-ish (`Color(0.8,0.7,0)`).
    -   Starts a new `Line` on the canvas at the touch position with a width of 10.
    -   Updates `last_x`, `last_y` to the touch coordinates.
    -   Resets `n_points` and `length` (used for line density calculation).
    -   Marks the touched point on the `sand` array as an obstacle (`sand[int(touch.x),int(touch.y)] = 1`).

-   **`on_touch_move(self, touch)`**:
    -   Called when the mouse is dragged while the button is pressed.
    -   Only processes if `touch.button == 'left'`.
    -   Appends the current touch coordinates to the `points` of the current line (`touch.ud['line']`).
    -   Calculates `length` (accumulated distance of the drawn line segment) and `n_points` (number of points in the segment).
    -   Calculates `density = n_points / length`.
    -   Adjusts the line width: `touch.ud['line'].width = int(20 * density + 1)` (thicker lines for faster drawing).
    -   Marks a 20x20 area around the current touch position in the `sand` array as an obstacle (`sand[int(touch.x) - 10 : int(touch.x) + 10, int(touch.y) - 10 : int(touch.y) + 10] = 1`).
    -   Updates `last_x`, `last_y`.

### `CarApp(App)`

The main Kivy application class.

-   **Inherits from**: `kivy.app.App`

-   **`build(self)`**:
    -   This method is called by Kivy to construct the UI of the application.
    -   Creates an instance of `Game` as the parent widget.
    -   Calls `parent.serve_car()` to initialize the car.
    -   Schedules the `parent.update` method to be called at 60 FPS using `Clock.schedule_interval(parent.update, 1.0/60.0)`.
    -   Creates an instance of `MyPaintWidget` for drawing.
    -   Creates three `Button` widgets: `clearbtn`, `savebtn`, `loadbtn`.
    -   Binds their `on_release` events to `self.clear_canvas`, `self.save`, and `self.load` methods respectively.
    -   Adds the painter and buttons as children to the `parent` (Game) widget.
    -   Returns the `parent` widget as the root of the application.

-   **`clear_canvas(self, obj)`**:
    -   Clears all drawings from `self.painter.canvas`.
    -   Resets the `sand` array to zeros using `sand = np.zeros((longueur,largeur))`.

-   **`save(self, obj)`**:
    -   Prints "saving brain...".
    -   Calls `brain.save()` to save the DQN model.
    -   Uses `matplotlib.pyplot` to plot the collected `scores` and shows the plot.

-   **`load(self, obj)`**:
    -   Prints "loading last saved brain...".
    -   Calls `brain.load()` to load the DQN model from `last_brain.pth`.

## Running the Application

-   The script checks `if __name__ == '__main__':` to ensure `CarApp().run()` is called only when the script is executed directly.
-   `CarApp().run()` starts the Kivy event loop and runs the application. 