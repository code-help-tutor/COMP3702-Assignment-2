# COMP3702 Assignment 2

## COMP3702 Artificial Intelligence (Semester 2, 2024)

**Assignment 2: BeeBot MDP**

**Key information**:
- **Due**: 1pm, Friday 20 September 2024
- This assignment assesses your skills in developing discrete search techniques for challenging problems.
- Assignment 2 contributes 20% to your final grade.
- This assignment consists of two parts: (1) programming and (2) a report.
- This is an individual assignment.
- Both code and report are to be submitted via Gradescope (https://www.gradescope.com/). You can find a link to the COMP3702 Gradescope site on Blackboard.
- Your code (Part 1) will be graded using the Gradescope code autograder, using the testcases in the support code provided at https://github.com/comp3702/2024-Assignment-2-Support-Code.
- Your report (Part 2) should fit the template provided, be in.pdf format and named according to the format a2 - COMP3702 - [SID].pdf. Reports will be graded by the teaching team.

**The BeeBot AI Environment**

You have been tasked with developing a planning algorithm for automatically controlling BeeBot, a Bee which operates in a hexagonal environment, and has the capability to push, pull and rotate honey ‘Widgets’ in order to reposition them to target honeycomb locations. To aid you in this task, we have provided support code for the BeeBot environment which you will interface with to develop your solution. To optimally solve a level, your AI agent must efficiently find a sequence of actions so that every Target cell is occupied by part of a Widget, while incurring the minimum possible action cost.

For A2, the BeeBot environment has been extended to model non - deterministic outcomes of actions. Cost and action validity are now replaced by a reward function where action costs are represented by negative received rewards, with additional penalties (i.e., negative rewards) being incurred when a collision occurs (between the Bee or a honey Widget and an obstacle, or between Widgets). Updates to the game environment are indicated in pink font.

**Levels in BeeBot are composed of a Hexagonal grid of cells, where each cell contains a character representing the cell type. An example game level is shown in Figure 1.**



**Figure 1: Example game level of BeeBot, showing character - based and GUI visualiser representations**

**Environment representation**

**Hexagonal Grid**

The environment is represented by a hexagonal grid. Each cell of the hex grid is indexed by (row, column) coordinates. The hex grid is indexed top to bottom, left to right. That is, the top left corner has coordinates (0, 0) and the bottom right corner has coordinates (nrows - 1, ncols - 1). Even numbered columns (starting from zero) are in the top half of the row, and odd numbered columns are in the bottom half of the row. An example is shown in Figure 2.

```
____ ____
/ \ / \
/row 0 \____/row 0 \____...
\col 0 / \col 2 / \
\____/row 0 \____/row 0 \...
/ \col 1 / \col 3 /
/row 1 \____/row 1 \____/...
\col 0 / \col 2 / \
\____/row 1 \____/row 1 \...
\col 1 / \col 3 /
... \____/... \____/
......
```

**Figure 2: Example hexagonal grid showing the order that rows and columns are indexed**

Two cells in the hex grid are considered adjacent if they share an edge. For each non - border cell, there are 6 adjacent cells.

**The BeeBot Agent and its Actions**

The BeeBot Bee occupies a single cell in the hex grid. In the visualisation, the Bee is represented by the cell marked with the character ‘R’ (or the Bee in the graphical visualiser). The side of the cell marked with ‘*’ (or the arrow in the graphical visualiser) represents the front of the Bee. The state of the Bee is defined by its (row, column) coordinates and its orientation (i.e. the direction its front side is pointing towards).

At each time step, the agent is prompted to select an action. The Bee has 4 available nominal actions:
- **Forward →** move to the adjacent cell in the direction of the front of the Bee (keeping the same orientation)
- **Reverse →** move to the adjacent cell in the opposite direction to the front of the Bee (keeping the same orientation)
- **Spin Left →** rotate left (relative to the Bee’s front, i.e. counterclockwise) by 60 degrees (staying in the same cell)
- **Spin Right →** rotate right (i.e. clockwise) by 60 degrees (staying in the same cell)

Each time the Bee selects an action, there is a fixed probability (given as a parameter of each testcase) for the Bee to ‘drift’ by 60 degrees in a clockwise or counterclockwise direction (separate probabilities for each drift direction) before the selected nominal action is performed. The probability of drift occurring depends on which nominal action is selected, with some actions more likely to result in drift. Drifting CW and CCW are mutually exclusive events.

Additionally, there is a fixed probability (also given as a parameter of each testcase) for the Bee to ‘double move’, i.e. perform the nominal selected action twice. The probability of a double move occurring depends on which action is selected. Double movement may occur simultaneously with drift (CW or CCW).

The reward received after each action is the minimum/most negative out of the rewards received for the nominal action and any additional (drift/double move) actions.

The Bee is equipped with a gripper on its front side which allows it to manipulate honey Widgets. When the Bee is positioned with its front side adjacent to a honey Widget, performing the ‘Forward’ action will result in the Widget being pushed, while performing the ‘Reverse’ action will result in the honey Widget being pulled.

**Action Costs**

Each action has an associated cost, representing the amount of energy used by performing that action.

If the Bee moves without pushing or pulling a honey widget, the cost of the action is given by a base action cost, ACTION_BASE_COST[a] where ‘a’ is the action that was performed.

If the Bee pushes or pulls a honey widget, an additional cost of ACTION_PUSH_COST[a] is added on top, so the total cost is ACTION_BASE_COST[a] + ACTION_PUSH_COST[a].

The costs are detailed in the constants.py file of the support code:
```python
ACTION_BASE_COST = {FORWARD: 1.0, REVERSE: 1.0, SPIN_LEFT: 0.1, SPIN_RIGHT: 0.1}
ACTION_PUSH_COST = {FORWARD: 0.8, REVERSE: 0.5, SPIN_LEFT: 0.0, SPIN_RIGHT: 0.0}
```

**Obstacles**

Some cells in the hex grid are obstacles. In the visualisation, these cells are filled with the character ‘X’ (represented as black cells in the graphical visualiser). Any action which causes the Bee or any part of a honey Widget to enter an obstacle results in collision, causing the agent to receive a negative obstacle collision penalty as reward. This reward replaces the movement cost which the agent would have otherwise incurred. The outside boundary of the hex grid behaves in the same way as an obstacle.

Additionally, the environment now contains an additional obstacle type, called ‘thorns’. Thorns behave in the same way as obstacles, but when collision occurs, a different (larger) penalty is received as the reward. As a result, avoiding collisions with thorns has greater importance than avoiding collisions with obstacles. Thorns are represented by ‘!!!’ in the text - based visualisation and red cells in the graphical visualisation.

**Widgets**

Honey Widgets are objects which occupy multiple cells of the hexagonal grid, and can be rotated and translated by the BeeBot Bee. The state of each honey widget is defined by its centre position (row, column) coordinates and its orientation. Honey Widgets have rotational symmetries - orientations which are rotationally symmetric are considered to be the same.

In the visualisation, each honey Widget in the environment is assigned a unique letter ‘a’, ‘b’, ‘c’, etc. Cells which are occupied by a honey Widget are marked with the letter assigned to that Widget (surrounded by round brackets). The centre position of the honey Widget is marked by the uppercase version of the letter, while all other cells occupied by the widget are marked with the lowercase.

Three honey widget types are possible, called Widget3, Widget4 and Widget5, where the trailing number denotes the number of cells occupied by the honey Widget. The shapes of these three honey Widget types and each of their possible orientations are shown in Figures 3 to 5 below.

**VERTICAL**
```
_____
SLANT_LEFT / \ SLANT_RIGHT
_____ / (a) \ _____
/ \ \ / / \
/ (a) \_____/ (a) \ \_____/ (a) \
\ / \ / \ / \ /
\_____/ (A) \_____ / (A) \ _____/ (A) \_____/
\ / \ \ /
_____/ (A) \_____ \_____/
/ \ / \ / \
/ (a) \_____/ (a) \ / (a) \
\ / \ / \ /
\_____/ \_____/ \_____/
```

**Figure 3: Widget3**

**UP DOWN**
```
_____ _____ _____
/ \ / \ / \
/ (a) \ / (a) \_____/ (a) \
\ / \ / \ /
\_____/ \_____/ (A) \_____/
/ \ \ /
_____/ (A) \_____ \_____/
/ \ / \ / \
/ (a) \_____/ (a) \ / (a) \
\ / \ / \ /
\_____/ \_____/ \_____/
```

**Figure 4: Widget4**

**SLANT_RIGHT SLANT_LEFT**
```
_____ _____
HORIZONTAL / \ / \
_____ _____ / (a) \_____ _____/ (a) \
/ \ / \ \ / \ / \ /
/ (a) \_____/ (a) \ \_____/ (a) \ / (a) \_____/
\ / \ / / \ / \ / \
\_____/ (A) \_____/ _____/ (A) \_____/ \_____/ (A) \_____
/ \ / \ / \ / \ / \
/ (a) \_____/ (a) \ / (a) \_____/ \_____/ (a) \
\ / \ / \ / \ / \ /
\_____/ \_____/ \_____/ (a) \ / (a) \_____/
\ / \ /
\_____/ \_____/
```

**Figure 5: Widget5**

Two types of widget movement are possible – translation (change in centre position) and rotation (change in orientation).

Translation occurs when the Bee is positioned with its front side adjacent to one of the honey widgets’ cells such that the Bee’s orientation is in line with the honey widget’s centre position. Translation results in the centre position of the widget moving in the same direction as the Bee. The orientation of the honey Widget does not change when translation occurs. Translation can occur when either ‘Forward’ or ‘Reverse’ actions are performed. For an action which results in translation to be valid, the new position of all cells of the moved widget must not intersect with the environment boundary, obstacles, the cells of any other honey Widgets or the Bee’s new position.

Rotation occurs when the Bee’s new position intersects one of the cells of a honey Widget but the Bee’s orientation does not point towards the centre of that widget. Rotation results in the honey widget spinning around its centre point, causing the widget to change orientation. The position of the centre point does not change when rotation occurs. Rotation can only occur for the ‘Forward’ action - performing ‘Reverse’ in a situation where ‘Forward’ would result in a widget rotation is considered invalid.

The following diagrams show which moves result in translation or rotation for each honey Widget type, with the arrows indicating directions from which the Bee can push or pull a widget in order to cause a translation or rotation of the widget. Pushing in a direction which is not marked with an arrow is considered invalid.

**Forward Translate**
**Reverse Translate**
**Forward Rotate CW**
**Reverse Rotate CCW**

**Figure 6: Widget3 translations and rotations**

**Forward Translate**
**Reverse Translate**
**Forward Rotate CW**
**Reverse Rotate CCW**

**Figure 7: Widget4 translations and rotations**

**Forward Translate**
**Reverse Translate**
**Forward Rotate CW**
**Reverse Rotate CCW**

**Figure 8: Widget5 translations and rotations**

**Targets**

The hex grid contains a number of ‘target’ honeycomb cells which must be filled with honey. In the visuali - sation, these cells are marked with ‘tgt’ (cells coloured in orange in the graphical visualiser). For a BeeBot environment to be considered solved, each target cell must be occupied by part of a honey Widget. The number of targets in an environment is always less than or equal to the total number of cells occupied by all honey Widgets.

**Interactive mode**

A good way to gain an understanding of the game is to play it. You can play the game to get a feel for how it works by launching an interactive game session from the terminal with the following command for the graphical visualiser:
```
$ python play_game.py.txt
```
or the following command for the command - line ASCII - character based visualiser:
```
$ python play.py.txt
```
where.txt is a valid testcase file (from the support code, with path relative to the current directory), e.g. testcases/ex1.txt.

Depending on your python installation, you should run the code using python, python3 or py.

In interactive mode, type the symbol for your chosen action (and press enter in the command line version) to perform the action: press ’W’ to move the Bee forward, ’S’ to move the Bee in reverse, ’A’ to turn the Bee left (counterclockwise) and ’D’ to turn the Bee right (clockwise). Use ’Q’ to quit the simulation, and ’R’ to reset the environment to the initial configuration.

**BeeBot as an MDP**

In this assignment, you will write the components of a program to play BeeBot, with the objective of finding a high - quality solution to the problem using various sequential decision - making algorithms based on the Markov decision process (MDP) framework. This assignment will test your skills in defining a MDP for a practical problem and developing effective exact algorithms for MDPs.

**What is provided to you**

We provide supporting code in Python, in the form of:
1. A class representing BeeBot game map and a number of helper functions
2. A parser method to take an input file (testcase) and convert it into a BeeBot map
3. A tester
4. Testcases to test and evaluate your solution
5. A script to allow you to play BeeBot interactively
6. A solution file template

The support code can be found at: https://github.com/comp3702/2024-Assignment-2-Support-Code.

Autograding of code will be done through Gradescope, so that you can evaluate your submission and continue to improve it based on this feedback — you are strongly encouraged to make use of this feedback. You can also test locally using the supplied tester.py file.

**Your assignment task**

Your task is to develop algorithms for computing policies, and to write a report testing your understanding of and analysing the performance of your algorithms. You will be graded on both your submitted code (Part 1, 60%) and the report (Part 2, 40%). These percentages will be scaled to the 20% course weighting for this assessment item.

The provided support code formulates BeeBot as an MDP, and your task is to submit code implementing the following MDP algorithms:
- **Value Iteration (VI)**
- **Policy Iteration (PI)**

Individual testcases specify which strategy (value iteration/policy iteration) will be applied, but you may modify the strategy specified in your local copy of the test cases for the purpose of comparing the performance of your two algorithms. Note that any local changes you make to the test cases will not modify the test cases on Gradescope (against which your programming component will be graded). The difficulty of higher level testcases will be designed to require a more advanced solution (e.g. linear algebra policy iteration).

Once you have implemented and tested the algorithms above, you are to complete the questions listed in the section “Part 2 - The Report” and submit the report to Gradescope.

More detail of what is required for the programming and report parts are given below.

**Part 1 — The programming task**

Your program will be graded using the Gradescope autograder, using the testcases in the support code provided at https://github.com/comp3702/2024-Assignment-2-Support-Code.

**Interaction with the testcases and autograder**

We now provide you with some details explaining how your code will interact with the testcases and the autograder (with special thanks to Nick Collins and Aryaman Sharma).

Information on the BeeBot implementation is provided in the Assignment 2 Support Code README.md and code comments.

Implement your solution using the supplied solution.py template file. You are required to fill in the following method stubs:
- **init(game_env)**
- **vi_initialise()**
- **vi_is_converged()**
- **vi_iteration()**
- **vi_plan_offline()**
- **vi_get_state_value()**
- **vi_select_action()**
- **pi_initialise()**
- **pi_is
# COMP3702 Assignment 2

# CS Tutor | 计算机编程辅导 | Code Help | Programming Help

# WeChat: cstutorcs

# Email: tutorcs@163.com

# QQ: 749389476

# 非中介, 直接联系程序员本人
