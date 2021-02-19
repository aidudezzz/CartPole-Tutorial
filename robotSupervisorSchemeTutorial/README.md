# CartPole Beginner Tutorial

![Solved cartpole demonstration](/robotSupervisorSchemeTutorial/images/cartPoleWorld.gif)

This tutorial shows the creation of the [CartPole](https://gym.openai.com/envs/CartPole-v0/) problem using the updated
version of the [*deepbots framework*](https://github.com/aidudezzz/deepbots), utilizing the 
[robot-supervisor scheme](https://github.com/aidudezzz/deepbots#combined-robot-supervisor-scheme) which combines the
gym environment and the robot controller in one script, forgoing the emitter and receiver communication.

The first parts of the tutorial are identical to the 
[original CartPole tutorial](https://github.com/aidudezzz/deepbots-tutorials/tree/master/cartPoleTutorial) that uses the 
[emitter-receiver scheme](https://github.com/aidudezzz/deepbots#emitter---receiver-scheme), so one can follow either
one, depending on their use-case. Mainly, if you desire to set up a more complicated example that might use multiple
robot or similar, refer to the emitter-receiver tutorial to get started.

Keep in mind that the tutorial is very detailed and many parts can be completed really fast by an 
experienced user. The tutorial assumes no familiarity with the [Webots](https://cyberbotics.com/) simulator.

We will recreate the [CartPole](https://gym.openai.com/envs/CartPole-v0/) problem step-by-step in 
[Webots](https://cyberbotics.com/), and solve it with the 
[Proximal Policy Optimization](https://openai.com/blog/openai-baselines-ppo/) (PPO) 
Reinforcement Learning (RL) algorithm, using [PyTorch](https://pytorch.org/) as our neural network backend library.

We will focus on creating the project, the world and the controller scripts and how to use the *deepbots framework*.
For the purposes of the tutorial, a basic implementation of the PPO algorithm.  For guides on how to construct a 
custom robot, please visit the official Webots [tutorial](https://cyberbotics.com/doc/guide/tutorial-6-4-wheels-robot). 

You can check the complete example [here](/robotSupervisorSchemeTutorial/full_project) with all the scripts and nodes 
used in this tutorial.
The CartPole example is available on the [deepworlds](https://github.com/aidudezzz/deepworlds/) repository.


## Prerequisites

Before starting, several prerequisites should be met. Follow the [installation section on the deepbots framework main 
repository](https://github.com/aidudezzz/deepbots#installation).

For this tutorial you will also need to [install PyTorch](https://pytorch.org/get-started/locally/) 
(no CUDA/GPU support needed for this tutorial).


## CartPole
### Creating the project

Now we are ready to start working on the *CartPole* problem. First of all, we should create a new project.

1. Open Webots and on the menu bar, click *"Wizards -> New Project Directory..."*\
    ![New project menu option](/robotSupervisorSchemeTutorial/images/newProjectMenuScreenshot.png)
2. Select a directory of your choice
3. On world settings **all** boxes should be ticked\
    ![World settings](/robotSupervisorSchemeTutorial/images/worldSettingsScreenshot.png)
4. Give your world a name, e.g. "cartPoleWorld.wbt"
5. Press Finish

You should end up with:\
![Project created](/robotSupervisorSchemeTutorial/images/projectCreatedScreenshot.png)


### Adding a *supervisor robot* node in the world

First of all we will download the *CartPole robot node* definition that is supplied for the purposes of the tutorial.
 
1. Right-click on
TODO Fix link [this link]() 
and click *Save link as...* to download the CartPole robot definition 
2. Save the .wbo file in a directory of your choice, where you can easily find it later.

Now that the project and the starting world are created, we are going to create our *robot*, that also has *supervisor*
privileges. Later, we will add the *controller* script, through which we will be able to handle several 
aspects of the simulation needed for RL (e.g. resetting), but also control the robot with the actions produced by the 
RL agent.
 
1. Click on the *Add a new object or import an object* button\
![Add new object button](/robotSupervisorSchemeTutorial/images/addNewObjectButtonScreenshot.png)
2. Click on *Import...* on the bottom right of the window\
![Add Robot node](/robotSupervisorSchemeTutorial/images/importRobotNodeScreenshot.png)
3. Locate the .wbo file downloaded earlier, select it and click *Open*
4. Now on the left side of the screen, under the *Rectangle Arena* node, you can see the *Robot* node
5. Double click on the *Robot* node to expand it
6. Scroll down to find the *supervisor* field and set it to TRUE\
![Set supervisor to TRUE](/robotSupervisorSchemeTutorial/images/setSupervisorTrueScreenshot.png)
7. Click *Save*\
![Click save button](/robotSupervisorSchemeTutorial/images/clickSaveButtonScreenshot.png)


### Adding the controllers

Now we will create the two basic controller scripts needed to control the *supervisor* and the *robot* nodes.
Then we are going to assign the *supervisor controller* script to the *supervisor robot* node created before.
Note that the *CartPole robot* node is going to be loaded into the world through the *supervisor controller* script
later, but we still need to create its controller.

Creating the *supervisor controller* and *robot controller* scripts:
1. On the *menu bar*, click *"Wizards -> New Robot Controller..."*\
![New robot controller](/robotSupervisorSchemeTutorial/images/newControllerMenuScreenshot.png)
2. On *Language selection*, select *Python*
3. Give it the name "*supervisorController*"*
4. Press *Finish* 
5. Repeat from step 1, but on step 3 give the name "*robotController*"

*If you are using an external IDE:    
1. Un-tick the "open ... in Text Editor" boxes and press *Finish*
2. Navigate to the project directory, inside the *Controllers/controllerName/* directory
3. Open the controller script with your IDE

Two new Python controller scripts should be created and opened in Webots text editor looking like this:\
![New robot controller](/robotSupervisorSchemeTutorial/images/newControllerCreated.png)

Assigning the *supervisorController* to the *supervisor robot* node *controller* field:
1. Expand the *supervisor robot* node created earlier and scroll down to find the *controller* field
2. Click on the *controller* field and press the "*Select...*" button below\
![New robot controller](/robotSupervisorSchemeTutorial/images/assignSupervisorController1Screenshot.png)
3. Find the "*supervisorController*" controller from the list and click it\
![New robot controller](/robotSupervisorSchemeTutorial/images/assignSupervisorController2Screenshot.png)
4. Click *OK*
5. Click *Save*

### Downloading the CartPole robot node

The *CartPole robot node* definition is supplied for the purposes of the tutorial.
 
1. Right-click on 
[this link](https://raw.githubusercontent.com/aidudezzz/deepbots-tutorials/master/robotSupervisorSchemeTutorial/full_project/controllers/supervisorController/CartPoleRobot.wbo) 
and click *Save link as...* to download the CartPole robot definition 
2. Save the .wbo file inside the project directory, under Controllers/supervisorController/


### Code overview

Before delving into writing code, we take a look at the general workflow of the framework. We will create two classes 
that inherit the *deepbots framework* classes and write implementations for several key methods, specific for the 
*CartPole* problem.

We will be implementing the basic methods *get_observations*, *get_reward*, *is_done* and *reset*, used for RL based 
on the [OpenAI Gym](https://gym.openai.com/) framework logic, that will be contained in the *supervisor controller*. 
These methods will compose the *environment* for the RL algorithm. The *supervisor controller* will also contain the 
RL *agent*, that will receive *observations* and output *actions*.

We will also be implementing methods that will be used by the *handle_emitter* and *handle_receiver* methods on the 
*robot controller* to send and receive data between the *robot* and the *supervisor*.

The following diagram loosely defines the general workflow of the framework:\
![deepbots workflow](/robotSupervisorSchemeTutorial/images/workflowDiagram.png)

The *robot controller* will gather data from the *robot's* sensors and send it to the *supervisor controller*. The 
*supervisor controller* will use the data received and extra data to compose the *observation* for the agent. Then, 
using the *observation* the *agent* will perform a forward pass and return an *action*. Then the *supervisor controller* 
will send the *action* back to the *robot controller*, which will perform the *action* on the *robot*. This closes the 
loop, that repeats until a termination condition is met, defined in the *is_done* method. 


### Writing the scripts

Now we are ready to start writing the *robot controller* and *supervisor controller* scripts.
It is recommended to delete the contents of the two scripts that were automatically created. 

### Robot controller script

First, we will write the *robot controller* script. In this script we will import the *RobotEmitterReceiverCSV*
class from the *deepbots framework* and inherit it into our own *CartPoleRobot* class. Then, we are going to
implement the two basic framework methods *create_message* and *use_message_data*. The former gathers data from the 
*Robot*'s sensors and packs it into a string message to be sent to the *supervisor controller* script. The latter 
unpacks messages sent by the *supervisor* that contain the next action, and uses the data to move the *CartPoleRobot* 
forward and backward.

The only import we are going to need is the *RobotEmitterReceiverCSV* class.
```python
from deepbots.robots.controllers.robot_emitter_receiver_csv import RobotEmitterReceiverCSV
```
Then we define our class, inheriting the imported one.
```python
class CartpoleRobot(RobotEmitterReceiverCSV):
    def __init__(self):
        super().__init__()
```
Then we initialize the position sensor that reads the pole's angle, needed for the agent's observation.
```python
        self.positionSensor = self.robot.getPositionSensor("polePosSensor")
        self.positionSensor.enable(self.get_timestep())
```
Finally, we initialize the four motors completing our `__init__()` method.
```python
        self.wheel1 = self.robot.getMotor('wheel1')  # Get the wheel handle
        self.wheel1.setPosition(float('inf'))  # Set starting position
        self.wheel1.setVelocity(0.0)  # Zero out starting velocity
        self.wheel2 = self.robot.getMotor('wheel2')
        self.wheel2.setPosition(float('inf'))
        self.wheel2.setVelocity(0.0)
        self.wheel3 = self.robot.getMotor('wheel3')
        self.wheel3.setPosition(float('inf'))
        self.wheel3.setVelocity(0.0)
        self.wheel4 = self.robot.getMotor('wheel4')
        self.wheel4.setPosition(float('inf'))
        self.wheel4.setVelocity(0.0)
```
After the initialization method is done we move on to the `create_message()` method implementation, used to pack the 
value read by the sensor into a string, so it can be sent to the *supervisor controller*.

(mind the indentation, the following two methods belong to the *CartpoleRobot* class)
```python
    def create_message(self):
        # Read the sensor value, convert to string and save it in a list
        message = [str(self.positionSensor.getValue())]
        return message
```
Finally, we implement the `use_message_data()` method, which unpacks the message received by the 
*supervisor controller*, that contains the next action. Then we implement what the *action* actually means for the
*CartPoleRobot*, i.e. moving forward and backward using its motors.
```python
    def use_message_data(self, message):
        action = int(message[0])  # Convert the string message into an action integer

        if action == 0:
            motorSpeed = 5.0
        elif action == 1:
            motorSpeed = -5.0
        else:
            motorSpeed = 0.0
        
        # Set the motors' velocities based on the action received
        self.wheel1.setVelocity(motorSpeed)
        self.wheel2.setVelocity(motorSpeed)
        self.wheel3.setVelocity(motorSpeed)
        self.wheel4.setVelocity(motorSpeed)
```
That's the *CartpoleRobot* class complete. Now all that's left, is to add (outside the class scope, mind the 
indentation) the code that runs the controller.

```python
# Create the robot controller object and run it
robot_controller = CartpoleRobot()
robot_controller.run()  # Run method is implemented by the framework, just need to call it
```

Now the *robot controller* script is complete! We move on to the *supervisor controller* script.

### Supervisor controller script
Before we start coding, we should add two scripts, one that contains the RL PPO agent, 
and the other containing utility functions that we are going to need.

Save both files inside the project directory, under Controllers/supervisorController/
1. Right-click on [this link](https://raw.githubusercontent.com/aidudezzz/deepbots-tutorials/master/robotSupervisorSchemeTutorial/full_project/controllers/supervisorController/PPOAgent.py) and click *Save link as...* to download the PPO agent
2. Right-click on [this link](https://raw.githubusercontent.com/aidudezzz/deepbots-tutorials/master/robotSupervisorSchemeTutorial/full_project/controllers/supervisorController/utilities.py) and click *Save link as...* to download the utilities script

Now for the imports, we are going to need the numpy library, the deepbots SupervisorCSV class, the PPO agent and the
utilities.

```python
import numpy as np
from deepbots.supervisor.controllers.supervisor_emitter_receiver import SupervisorCSV
from PPOAgent import PPOAgent, Transition
from utilities import normalizeToRange
```

Then we define our class inheriting the imported, also defining the observation and action spaces.
Here, the observation space is basically the number of the neural network's inputs, so its defined simply as an
integer. 

Num | Observation | Min | Max
----|-------------|-----|----
0 | Cart Position z axis | -0.4 | 0.4
1 | Cart Velocity | -Inf | Inf
2 | Pole Angle | -1.3 rad | 1.3 rad
3 | Pole Velocity at Tip | -Inf | Inf

The action space defines the outputs of the neural network, which are 2. One for the forward movement
and one for the backward movement of the robot. 

```python
class CartPoleSupervisor(SupervisorCSV):
    def __init__(self):
        super().__init__()
        self.observationSpace = 4  # The agent has 4 inputs
        self.actionSpace = 2  # The agent can perform 2 actions
```
Then we initialize the `self.robot` variable which will hold a reference to the *CartPole robot* node.

The `respawnRobot()` method is called to use the .wbo file we downloaded earlier, to spawn the robot node
into the world and give a value to the self.robot variable. We will implement this method later.

We also get a reference for the *pole endpoint* node, which is a child node of the *CartPole robot node* and is going
to be useful for getting the pole tip velocity. 
```python
        self.robot = None
        self.respawnRobot()
        self.poleEndpoint = self.supervisor.getFromDef("POLE_ENDPOINT")
        self.messageReceived = None  # Variable to save the messages received from the robot
```
Finally, we initialize several variables used during training. Note that the `self.stepsPerEpisode` is set to `200` 
based on the problem's definition. Feel free to change the `self.episodeLimit` variable.

```python

        self.episodeCount = 0  # Episode counter
        self.episodeLimit = 10000  # Max number of episodes allowed
        self.stepsPerEpisode = 200  # Max number of steps per episode
        self.episodeScore = 0  # Score accumulated during an episode
        self.episodeScoreList = []  # A list to save all the episode scores, used to check if task is solved
```        

Before implementing the base environment methods, we will first implement the `respawnRobot()` method,
which spawns the *CartPole robot* node, resetting it to its initial state, using several Webots methods.
This method also uses the `simulationResetPhysics()` supervisor method to reset the simulation.

(mind the indentation of the following code snippets, the following methods all belong inside the 
*CartpoleSupervisor* class)
```python
    def respawnRobot(self):
        if self.robot is not None:
            # Despawn existing robot
            self.robot.remove()

        # Respawn robot in starting position and state
        rootNode = self.supervisor.getRoot()  # This gets the root of the scene tree
        childrenField = rootNode.getField('children')  # This gets a list of all the children, ie. objects of the scene
        childrenField.importMFNode(-2, "CartPoleRobot.wbo")  # Load robot from file and add to second-to-last position

        # Get the new robot and pole endpoint references
        self.robot = self.supervisor.getFromDef("ROBOT")
        self.poleEndpoint = self.supervisor.getFromDef("POLE_ENDPOINT")
```

Now its time for us to implement the base environment methods that a regular OpenAI Gym environment uses and
most RL algorithm implementations (agents) expect.
These base methods are *get_observations()*, *get_reward()*, *is_done()*, *reset()* and *get_info()*.
 
Let's start with the `get_observations()` method, which builds the agent's observation (i.e. the neural network's input) 
for each step. This method also normalizes the values in the [-1.0, 1.0] range as appropriate, using the 
`normalizeToRange()` utility method.

We will start by getting the *CartPole robot* node position and velocity on the z axis. The z axis is the direction of 
its forward/backward movement. We will also get the pole tip velocity from the *poleEndpoint* node we defined earlier.
```python
    def get_observations(self):
        # Position on z axis, third (2) element of the getPosition vector
        cartPosition = normalizeToRange(self.robot.getPosition()[2], -0.4, 0.4, -1.0, 1.0)
        # Linear velocity on z axis
        cartVelocity = normalizeToRange(self.robot.getVelocity()[2], -0.2, 0.2, -1.0, 1.0, clip=True)
        # Angular velocity x of endpoint
        endpointVelocity = normalizeToRange(self.poleEndpoint.getVelocity()[3], -1.5, 1.5, -1.0, 1.0, clip=True)
```
Now all it's missing is the pole angle off vertical, which will be provided by the robot sensor.
To get it, we will need to call the `handle_receiver()` method to get the message sent by the robot into the
`self.messageReceived` variable. The message received, as defined into the robot's `create_message()` method, is a 
string which, here, gets converted back into a single float value. 

```python
        # Update self.messageReceived received from robot, which contains pole angle
        self.messageReceived = self.handle_receiver()
        if self.messageReceived is not None:
            poleAngle = normalizeToRange(float(self.messageReceived[0]), -0.23, 0.23, -1.0, 1.0, clip=True)
        else:
            # Method is called before self.messageReceived is initialized
            poleAngle = 0.0
```

Finally, we return a list containing all four values we created earlier.

```python
        return [cartPosition, cartVelocity, poleAngle, endpointVelocity]
```

Now for something simpler, we will define the `get_reward()` method, which simply returns
`1` for each step. Usually reward functions are more elaborate, but for this problem it is simply
defined as the agent getting a +1 reward for each step it manages to keep the pole from falling.

```python
    def get_reward(self, action=None):
        return 1
```

Moving on, we define the *is_done()* method, which contains the episode termination conditions:
- Episode terminates if the pole has fallen beyond an angle which can be realistically recovered (+-15 degrees)
- Episode terminates if episode score is over 195
- Episode terminates if the robot hit the walls by moving into them, which is calculated based on its position on z axis

```python
    def is_done(self):
        if self.messageReceived is not None:
            poleAngle = round(float(self.messageReceived[0]), 2)
        else:
            # method is called before self.messageReceived is initialized
            poleAngle = 0.0
        if abs(poleAngle) > 0.261799388:  # more than 15 degrees off vertical
            return True

        if self.episodeScore > 195.0:
            return True

        cartPosition = round(self.robot.getPosition()[2], 2)  # Position on z axis
        if abs(cartPosition) > 0.39:
            return True

        return False
```

We separate the *solved* condition into another method, the `solved()` method, because it requires different handling.
The *solved* condition depends on the agent completing consecutive episodes successfully, consistently. We measure this,
by taking the average episode score of the last 100 episodes and checking if it's over 195.

```python
    def solved(self):
        if len(self.episodeScoreList) > 100:  # Over 100 trials thus far
            if np.mean(self.episodeScoreList[-100:]) > 195.0:  # Last 100 episodes' scores average value
                return True
        return False
```

Now we move on to `reset()`. Reset simply calls the `respawnRobot()` method described earlier to reset the *CartPole 
robot* node to its initial state and then calls the Webots method to reset the simulation physics. Also resets
`self.messageReceived` and then returns a zero vector as starting observation.

```python
    def reset(self):
        self.respawnRobot()
        self.supervisor.simulationResetPhysics()  # Reset the simulation physics to start over
        self.messageReceived = None
        return [0.0 for _ in range(self.observationSpace)]
```

Lastly, we add a dummy implementation of `get_info()` method, because in this example it is not actually used, but
is called by the framework.

```python
    def get_info(self):
        return None
```

This concludes the *CartPoleSupervisor* class, that now contains all required methods to run an RL training loop!

### RL Training Loop

Finally, it all comes together inside the RL training loop. Now we initialize the RL agent and create the 
*CartPoleSupervisor* class object with which it gets trained to solve the problem, maximizing the reward received
by our reward function and achieve the solved condition defined.

First we create a supervisor object and then initialize the PPO agent, providing it with the observation and action
spaces.

```python
supervisor = CartPoleSupervisor()
agent = PPOAgent(supervisor.observationSpace, supervisor.actionSpace)
```

Then we set the `solved` flag to `false`. This flag is used to terminate the training loop.
```python
solved = False
```

Now we define the outer loop which runs the number of episodes defined in the supervisor class
and resets the world to get the starting observation. We also reset the episode score to zero.

(please be mindful of the indentation on the following code, because we are about to define several levels of nested
loops and ifs)
```python
# Run outer loop until the episodes limit is reached or the task is solved
while not solved and supervisor.episodeCount < supervisor.episodeLimit:
    observation = supervisor.reset()  # Reset robot and get starting observation
    supervisor.episodeScore = 0
```

Inside the outer loop defined above we define the inner loop which runs for the course of an episode. This loop
runs for a maximum number of steps defined by the problem. Here, the RL agent - environment loop takes place.

We start by calling the `agent.work()` method, by providing it with the current observation, which for the first step
is the zero vector returned by the `reset()` method. The `work()` method implements the forward pass of the agent's 
actor neural network, providing us with the next action. As the comment suggests the PPO algorithm implements 
exploration by sampling for the probability distribution the agent outputs from its actor's softmax output layer.

```python
    for step in range(supervisor.stepsPerEpisode):
        # In training mode the agent samples from the probability distribution, naturally implementing exploration
        selectedAction, actionProb = agent.work(observation, type_="selectAction")
``` 

The next part contains the call to the `step()` method. This method calls most of the methods we implemented earlier 
(`get_observation()`, `get_reward()`, `is_done()` and `get_info()`), steps the Webots controller and sends the action 
that the agent selected to the robot for execution. Step returns the new observation, the reward for the previous 
action and whether the episode is terminated (info is not implemented in this example).

Then, we create the `Transition`, which is a named tuple that contains, as the name suggests, the transition between
the previous `observation` (/state) to the `newObservation` (/newState). This is needed by the agent for its training 
procedure, so we call the agent's `storeTransition()` method to save it to the buffer. Most RL algorithms require a 
similar procedure and have similar methods to do it.

```python
        # Step the supervisor to get the current selectedAction's reward, the new observation and whether we reached 
        # the done condition
        newObservation, reward, done, info = supervisor.step([selectedAction])

        # Save the current state transition in agent's memory
        trans = Transition(observation, selectedAction, actionProb, reward, newObservation)
        agent.storeTransition(trans)
```

Finally, we check whether the episode is terminated and if it is, we save the episode score, run a training step
for the agent giving the number of steps taken in the episode as batch size, check whether the problem is solved
via the `solved()` method and break.

If not, we add the step reward to the `episodeScore` accumulator, save the `newObservation` as `observation` and loop 
onto the next episode step.

```python
        if done:
            # Save the episode's score
            supervisor.episodeScoreList.append(supervisor.episodeScore)
            agent.trainStep(batchSize=step)
            solved = supervisor.solved()  # Check whether the task is solved
            break

        supervisor.episodeScore += reward  # Accumulate episode reward
        observation = newObservation  # observation for next step is current step's newObservation
```

This is the inner loop complete and now we add a print statement and increment the episode counter to finalize the outer
loop.

(note that the following code snippet is part of the outer loop)
```python
    print("Episode #", supervisor.episodeCount, "score:", supervisor.episodeScore)
    supervisor.episodeCount += 1  # Increment episode counter
```

With the outer loop complete, this completes the training procedure. Now all that's left is the testing loop which is a
barebones, simpler version of the training loop. First we print a message on whether the task is solved or not (i.e. 
reached the episode limit without satisfying the solved condition) and call the `reset()` method. Then, we create a 
`while True` loop that runs the agent's forward method, but this time selecting the action with the max probability
out of the actor's softmax output, eliminating exploration. Finally, the `step()` method is called, but this time
we keep only the observation it returns so as to keep the environment - agent loop running.

```python
if not solved:
    print("Task is not solved, deploying agent for testing...")
elif solved:
    print("Task is solved, deploying agent for testing...")
observation = supervisor.reset()
while True:
    selectedAction, actionProb = agent.work(observation, type_="selectActionMax")
    observation, _, _, _ = supervisor.step([selectedAction])
```

### Conclusion

Now with the coding done you can click on the *Run the simulation* button and watch the training run!
 
![Run the simulation](/robotSupervisorSchemeTutorial/images/clickPlay.png)\
Webots allows to speed up the simulation, even run it without graphics, so the training shouldn't take long, at 
least to see the agent becoming visibly better at moving under the pole to balance it. It takes a while for it to 
achieve the *solved* condition, but when it does it becomes quite good at balancing the pole! You can even apply forces 
in real time by pressing Alt - left-click and drag on the robot or the pole.

That's it for this tutorial! :)

![Solved cartpole demonstration](/robotSupervisorSchemeTutorial/images/cartPoleWorld.gif)
