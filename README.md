# Self-Driving Car Simulation

This project implements a self-driving car simulation using Python, Kivy for the graphical interface, and a Deep Q-Network (DQN) for the AI.

## Project Structure

- `ai.py`: Contains the implementation of the Neural Network and the DQN algorithm.
- `map.py`: Defines the simulation environment, car physics, sensors, and handles the Kivy application setup.
- `car.kv`: Kivy language file that defines the visual layout of the car and its sensors.
- `requirements.txt`: Lists the Python dependencies required to run the project.
- `last_brain.pth`: A pre-trained model for the DQN agent.
- `.gitignore`: Specifies intentionally untracked files that Git should ignore.
- `map_commented.py`: A commented version of `map.py` (likely for educational purposes).

## Core Components

### Artificial Intelligence (`ai.py`)

The AI for the self-driving car is based on a Deep Q-Network (DQN).

- **`Network` Class**: A feedforward neural network with one hidden layer (30 neurons). It takes the car's sensor inputs and outputs Q-values for each possible action.
  - Input size: 5 (3 sensor signals + orientation to goal + negative orientation to goal)
  - Output size: 3 (actions: go straight, turn left, turn right)
- **`ReplayMemory` Class**: Implements experience replay. It stores transitions (state, action, reward, next_state) observed by the agent, allowing for more stable and efficient learning. The capacity of the memory is 100,000 transitions.
- **`Dqn` Class**: Implements the DQN algorithm.
  - **Initialization**: Sets up the model, replay memory, optimizer (Adam), and learning parameters (gamma = 0.9).
  - **`select_action(state)`**: Selects an action based on the current state using an epsilon-greedy strategy (implicitly, via softmax with temperature T=100 on Q-values).
  - **`learn(batch_state, batch_next_state, batch_reward, batch_action)`**: Updates the network's weights by minimizing the Temporal Difference (TD) error using a batch of experiences sampled from the replay memory. It uses the Smooth L1 loss function.
  - **`update(reward, new_signal)`**: This is the main update step called in each simulation frame. It stores the latest transition in the replay memory, selects an action for the new state, and triggers the learning process if enough samples are in the memory.
  - **`score()`**: Calculates the average reward over the last 1000 steps.
  - **`save()` / `load()`**: Methods to save the trained model's state dictionary and optimizer state to `last_brain.pth` and load them back.

### Simulation Environment (`map.py` and `car.kv`)

The simulation is built using the Kivy framework.

- **`Car` Class (in `map.py`)**:
  - Represents the self-driving car.
  - Properties: `angle`, `rotation`, `velocity`, `pos`.
  - Sensors: Three sensors (`sensor1`, `sensor2`, `sensor3`) are placed at the front of the car (center, +30 degrees, -30 degrees).
  - `move(rotation)`: Updates the car's position and angle based on the chosen rotation. It also calculates the signals from the sensors.
  - Sensor signals (`signal1`, `signal2`, `signal3`): These values represent the density of "sand" (obstacles) detected by each sensor. The values are scaled between 0 and 1. If a sensor is off-map, the signal is set to 1.

- **`Game` Class (in `map.py`)**:
  - Manages the overall simulation state and updates.
  - `serve_car()`: Initializes the car's position and velocity.
  - `update(dt)`: This function is called at each time step (1/60th of a second).
    - Calculates the car's orientation towards the goal.
    - Creates the `last_signal` array (sensor signals + orientation) for the DQN agent.
    - Calls `brain.update()` to get the next action and update the DQN.
    - Moves the car based on the selected action.
    - Calculates the reward based on whether the car is on sand, its proximity to the goal, and whether it hits a boundary.
    - Updates the goal position when the car gets close to the current goal.

- **`MyPaintWidget` Class (in `map.py`)**:
  - Allows the user to draw "sand" (obstacles) on the map using mouse clicks and drags.
  - `on_touch_down` and `on_touch_move` methods handle drawing and updating the `sand` array (a NumPy array representing the map).

- **`CarApp` Class (in `map.py`)**:
  - The main Kivy application class.
  - `build()`: Sets up the game layout, initializes the car, schedules the `update` function, and adds buttons for:
    - `clear`: Clears the drawn sand from the map.
    - `save`: Saves the current DQN model and plots the scores.
    - `load`: Loads the last saved DQN model (`last_brain.pth`).

- **`car.kv`**:
  - Defines the visual representation of the `Car`, `Ball1` (sensor1), `Ball2` (sensor2), `Ball3` (sensor3), and `Game` widgets.
  - The car is a `Rectangle`.
  - The sensors are `Ellipse` shapes with different colors.

## How It Works

1.  **Initialization**: The Kivy application starts, the game environment is set up, and the DQN agent (`brain`) is initialized. If `last_brain.pth` exists, the pre-trained model is loaded.
2.  **User Interaction**: The user can draw obstacles (sand) on the map.
3.  **Sensing**: The car has three sensors that detect the density of sand in front of it. It also calculates its orientation relative to the current goal.
4.  **State Representation**: The state provided to the DQN agent consists of the three sensor readings and the orientation towards the goal (and its negative).
5.  **Action Selection**: The DQN agent selects an action (go straight, turn left by 20 degrees, or turn right by 20 degrees) based on the current state and its learned policy.
6.  **Movement**: The car moves according to the selected action.
7.  **Reward Calculation**:
    -   `+0.1`: If the car moves closer to the goal.
    -   `-0.2`: If the car moves on a clear path but not necessarily closer to the goal.
    -   `-1`: If the car drives on sand or hits a boundary.
8.  **Learning**: The agent's experience (state, action, reward, next state) is stored in the replay memory. The DQN is trained by sampling batches of experiences from this memory and updating its network weights to better predict future rewards.
9.  **Goal**: The car tries to reach a designated goal position on the map. When it gets close, the goal is moved to a new random location.
10. **Saving/Loading**: The user can save the trained DQN model or load a previously saved one.

## Getting Started

### Prerequisites

- Python 3.x
- Kivy
- PyTorch
- NumPy
- Matplotlib

### Installation

1.  Clone the repository:
    ```bash
    git clone <repository_url>
    cd self-driving-car
    ```
2.  Install the dependencies:
    ```bash
    pip install -r requirements.txt
    ```
    (Alternatively, if you use Pipenv: `pipenv install && pipenv shell`)

### Running the Simulation

```bash
python map.py
```

## Controls

-   **Left Mouse Button**: Click and drag on the map to draw sand (obstacles).
-   **Clear Button**: Clears all drawn sand from the map.
-   **Save Button**: Saves the current state of the AI's "brain" to `last_brain.pth` and displays a plot of the reward scores over time.
-   **Load Button**: Loads the AI "brain" from `last_brain.pth`.

## To-Do / Potential Improvements (Suggestions)

-   Implement more complex environments or maps.
-   Add more sophisticated sensors (e.g., LiDAR-like).
-   Experiment with different neural network architectures.
-   Fine-tune hyperparameters for the DQN agent.
-   Implement other reinforcement learning algorithms (e.g., A2C, PPO).
-   Improve the reward function for more nuanced behaviors.
-   Add more car physics (e.g., acceleration, braking).
-   Package the application for easier distribution. 