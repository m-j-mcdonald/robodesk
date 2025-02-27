# RoboDesk

THIS IS A FORK OF THE ORIGINAL REPO FROM [HERE](https://github.com/google-research/robodesk)

[![PyPI](https://img.shields.io/pypi/v/robodesk.svg)](https://pypi.python.org/pypi/robodesk/#history)

A Multi-Task Reinforcement Learning Benchmark

![Robodesk Banner](https://i.imgur.com/1qp1SUh.gif)

If you find this open source release useful, please reference in your paper:

```
@misc{kannan2021robodesk,
  author = {Harini Kannan and Danijar Hafner and Chelsea Finn and Dumitru Erhan},
  title = {RoboDesk: A Multi-Task Reinforcement Learning Benchmark},
  year = {2021},
  howpublished = {\url{https://github.com/google-research/robodesk}},
}
```

## Highlights

- **Diversity:** RoboDesk includes 9 diverse tasks that test for a variety of different behaviors within the same environment, making it useful for evaluating transfer, multi-task learning, and global exploration.
- **Complexity:** The high-dimensional image inputs contain objects of different shapes and colors, whose initial positions are randomized to avoid naive memorization and require learning algorithms to generalize.
- **Robustness:** We carefully designed and tested RoboDesk to ensure fast and stable physics simulation. This avoids objects from intersecting, getting stuck, or quickly flying away, a common problem with some existing environments.
- **Lightweight:** RoboDesk comes as a self-contained Python package with few dependencies. The source code is clean and pragmatic, making it a useful blueprint for creating new MuJoCo environments.

## Training Agents

Installation: `pip3 install -U robodesk`

The environment follows the [OpenAI Gym][gym] interface:

```py
import robodesk

env = robodesk.RoboDesk(seed=0)
obs = env.reset()
assert obs.shape == (64, 64, 3)

done = False
while not done:
  action = env.action_space.sample()
  obs, reward, done, info = env.step(action)
```

[gym]: https://github.com/openai/gym


## Tasks

![Robodesk Tasks](https://i.imgur.com/OwTT2pk.gif)

The behaviors above were learned using the [Dreamer](https://github.com/danijar/dreamer) agent. These policies have been learned from scratch and only from pixels, not proprioceptive states.

| Task | Description |
| :-------- | :---------- |
| `open_slide` | Push the sliding door all the way to the right, navigating around the other objects.  |
| `open_drawer` | Pull the dark brown drawer all the way open. |
| `push_green` | Push the green button to turn the green light on. |
| `stack_blocks` | Stack the upright blue block on top of the flat green block. |
| `upright_block_off_table` | Push the blue upright block off the table.  |
| `flat_block_in_bin` | Push the green flat block into the blue bin.  |
| `flat_block_in_shelf` | Push the green flat block into the shelf, navigating around the other blocks.  |
| `lift_upright_block` | Grasp the blue upright block and lift it above the table.  |
| `lift_ball` | Grasp the magenta ball and lift it above the table. |


## Environment Details

### Constructor

```py
robodesk.RoboDesk(task='open_slide', reward='dense', action_repeat=1, episode_length=500, image_size=64)
```

| Parameter | Description |
| :-------- | :---------- |
| `task` | Available tasks are `open_slide`, `open_drawer`, `push_green`, `stack`, `upright_block_off_table`, `flat_block_in_bin`, `flat_block_in_shelf`, `lift_upright_block`, `lift_ball`.  |
| `reward` | Available reward types are `dense`, `sparse`, `success`. Success gives only the first sparse reward during the episode, useful for computing success rates during evaluation. |
| `action_repeat` | Reduces the control frequency by applying each action multiple times. This is faster than using an environment wrapper because only the needed images are rendered. |
| `episode_length` | Time limit for the episode, can be `None`. |
| `image_size` | Size of the image observations in pixels, used for both height and width. |

### Reward

All rewards are bound between 0 and 1. There are three types of rewards available:

- Dense rewards are based on Euclidean distances between the objects and their target positions and can include additional terms, for example to encourage the arm to reach the object. These are the easiest rewards for learning.
- Sparse rewards are either 0 or 1 based on whether the target object is in the target area or not, according to a fixed threshold. Learning from sparse rewards is more challenging.
- Success rewards are equivalent to the sparse rewards, except that only the first reward is given during each episode. As a result, an episode return of 0 means failure and 1 means sucess at the task. This should only be used during evaluation.

### Termination

Episodes end after 500 time steps by default. There are no early terminations.

### Observation Space

Each observation is a dictionary that contains the current image, as well as additional information. For the standard benchmark, only the image should be used for learning. The observation dictionary contains the following keys:

| Key | Space |
| :-- | :---- |
| `image` | `Box(0, 255, (64, 64, 3), np.uint8)` |
| `qpos_robot` | `Box(-np.inf, np.inf, (9,), np.float32)` |
| `qvel_robot` | `Box(-np.inf, np.inf, (9,), np.float32)` |
| `qpos_objects` | `Box(-np.inf, np.inf, (26,), np.float32)` |
| `qvel_objects` | `Box(-np.inf, np.inf, (26,), np.float32)` |
| `end_effector` | `Box(-np.inf, np.inf, (3,), np.float32)` |

### Action Space

RoboDesk uses end effector control with a simple bounded action space:

```
Box(-1, 1, (5,), np.float32)
```

## Acknowledgements
We thank Ben Eysenbach and Debidatta Dwibedi for their helpful feedback.


Our benchmark builds upon previously open-sourced work. We build upon the desk XMLs first introduced in [[1]](https://github.com/google-research/google-research/tree/master/playrooms), the Franka XMLs open-sourced in [[2]](https://github.com/vikashplus/franka_sim), and the Franka meshes open-sourced in [[3]](https://github.com/frankaemika/franka_ros/tree/kinetic-devel/franka_description).

## Questions

Please [open an issue][issues] on Github.

[issues]: https://github.com/google-research/robodesk/issues

Disclaimer: This is not an official Google product.
