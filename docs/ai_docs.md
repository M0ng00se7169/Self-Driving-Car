# AI Documentation (`ai.py`)

This file implements the artificial intelligence for the self-driving car, utilizing a Deep Q-Network (DQN).

## Classes

### `Network(nn.Module)`

Defines the architecture of the neural network used by the DQN agent.

-   **Inherits from**: `torch.nn.Module`

-   **`__init__(self, input_size, nb_action)`**:
    -   Initializes the neural network layers.
    -   `input_size` (int): The number of input features for the network. In this project, it's 5 (3 sensor signals, orientation to the goal, and negative orientation to the goal).
    -   `nb_action` (int): The number of possible actions the car can take. In this project, it's 3 (go straight, turn left, turn right).
    -   Layers:
        -   `self.fc1`: A fully connected linear layer from `input_size` to 30 neurons.
        -   `self.fc2`: A fully connected linear layer from 30 neurons to `nb_action` neurons (outputting Q-values for each action).

-   **`forward(self, state)`**:
    -   Defines the forward pass of the network.
    -   `state` (torch.Tensor): The input state (sensor readings and orientation).
    -   Applies a ReLU activation function to the output of `self.fc1`.
    -   Returns `q_values` (torch.Tensor): The Q-values for each action, produced by `self.fc2`.

### `ReplayMemory(object)`

Implements the experience replay mechanism, which stores transitions observed by the agent to be used for training.

-   **`__init__(self, capacity)`**:
    -   Initializes the replay memory.
    -   `capacity` (int): The maximum number of transitions to store in the memory. Set to 100,000.
    -   `self.memory` (list): A list to store the experiences (transitions).

-   **`push(self, event)`**:
    -   Adds a new experience (transition) to the memory.
    -   `event` (tuple): A tuple representing the transition: `(last_state, new_state, last_action, last_reward)`.
    -   If the memory reaches its capacity, the oldest experience is removed.

-   **`sample(self, batch_size)`**:
    -   Randomly samples a batch of experiences from the memory.
    -   `batch_size` (int): The number of experiences to sample. Set to 100.
    -   Returns a batch of experiences, where each element of the batch (states, next_states, actions, rewards) is concatenated into a PyTorch Variable.

### `Dqn(object)`

Implements the Deep Q-Learning algorithm.

-   **`__init__(self, input_size, nb_action, gamma)`**:
    -   Initializes the DQN agent.
    -   `input_size` (int): The size of the input state (passed to the `Network`).
    -   `nb_action` (int): The number of possible actions (passed to the `Network`).
    -   `gamma` (float): The discount factor for future rewards. Set to 0.9.
    -   `self.reward_window` (list): Stores the recent rewards to calculate an average score.
    -   `self.model` (Network): An instance of the `Network` class.
    -   `self.memory` (ReplayMemory): An instance of the `ReplayMemory` class.
    -   `self.optimizer` (torch.optim.Adam): Adam optimizer for training the model, with a learning rate of 0.001.
    -   `self.last_state` (torch.Tensor): The last observed state, initialized as a tensor of zeros.
    -   `self.last_action` (int): The last action taken.
    -   `self.last_reward` (float): The last reward received.

-   **`select_action(self, state)`**:
    -   Selects an action based on the current state using the policy learned by the model.
    -   `state` (torch.Tensor): The current input state.
    -   Calculates Q-values using `self.model(Variable(state, volatile=True))`.
    -   Applies a softmax function with a temperature parameter (T=100) to the Q-values to get action probabilities. `volatile=True` was used in older PyTorch versions to indicate inference mode; modern PyTorch uses `with torch.no_grad():`.
    -   Samples an action from the probability distribution using `probs.multinomial(1)`.
    -   Returns `action.data[0,0]` (int): The selected action.

-   **`learn(self, batch_state, batch_next_state, batch_reward, batch_action)`**:
    -   Updates the model's weights based on a batch of experiences.
    -   `batch_state`, `batch_next_state` (torch.Tensor): Batches of current states and next states.
    -   `batch_reward` (torch.Tensor): Batch of rewards.
    -   `batch_action` (torch.Tensor): Batch of actions taken.
    -   Calculates the Q-values for the `batch_state` and gathers the Q-values corresponding to the `batch_action` taken (`outputs`).
    -   Calculates the Q-values for the `batch_next_state` using the target network (in this implementation, the same model is used, but `detach()` is called to prevent gradients from flowing back from the target Q-value calculation). It takes the maximum Q-value for each next state (`next_outputs`).
    -   Calculates the target Q-values: `target = self.gamma * next_outputs + batch_reward`.
    -   Computes the Temporal Difference (TD) loss using `F.smooth_l1_loss(outputs, target)`.
    -   Performs backpropagation: `self.optimizer.zero_grad()`, `td_loss.backward()`, `self.optimizer.step()`.

-   **`update(self, reward, new_signal)`**:
    -   This is the main method called at each step of the simulation to update the agent.
    -   `reward` (float): The reward received from the environment after the last action.
    -   `new_signal` (list): The new state information (sensor readings and orientation) from the environment.
    -   Converts `new_signal` to a PyTorch tensor `new_state`.
    -   Pushes the transition `(self.last_state, new_state, torch.LongTensor([int(self.last_action)]), torch.Tensor([self.last_reward]))` to the `self.memory`.
    -   Selects an `action` for the `new_state` using `self.select_action()`.
    -   If the replay memory has more than 100 samples, it samples a batch of 100 experiences and calls `self.learn()`.
    -   Updates `self.last_action`, `self.last_state`, and `self.last_reward`.
    -   Appends the current `reward` to `self.reward_window` (and maintains its size at 1000).
    -   Returns the selected `action`.

-   **`score(self)`**:
    -   Calculates the average reward over the rewards stored in `self.reward_window`.
    -   Returns the mean score.

-   **`save(self)`**:
    -   Saves the model's `state_dict` and the optimizer's `state_dict` to a file named `last_brain.pth`.

-   **`load(self)`**:
    -   Loads the model and optimizer state dictionaries from `last_brain.pth` if the file exists.
    -   Prints a message indicating whether the checkpoint was loaded or not found.

## Key Concepts

-   **Deep Q-Network (DQN)**: A reinforcement learning algorithm that uses a deep neural network to approximate the Q-function (action-value function).
-   **Experience Replay**: A technique where the agent's experiences (transitions) are stored in a replay memory and randomly sampled to train the neural network. This helps to break correlations in sequential experiences and improves learning stability.
-   **Target Network**: While not explicitly a separate target network is used in this code for Q-value calculation in the `learn` method (i.e. `self.model(batch_next_state).detach().max(1)[0]`), the `.detach()` serves a similar purpose by preventing gradients from the target Q-values from affecting the main network's gradients during the Bellman update. In more standard DQN, a separate network with periodically updated weights from the main network is often used.
-   **Softmax Action Selection**: Actions are chosen based on probabilities derived from the Q-values using a softmax function. The temperature parameter controls the exploration/exploitation trade-off. 