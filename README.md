## SuperTuxKart Bots race project
#Test branche
[![PyPI
version](https://badge.fury.io/py/pystk2-gymnasium.svg)](https://badge.fury.io/py/pystk2-gymnasium)

Read the [Changelog](./CHANGELOG.md)

## Install

We assume you are on linux

Read this uv tutorial: https://www.datacamp.com/tutorial/python-uv

- Fork this project onto your disk: https://docs.github.com/fr/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo
- install uv: `sudo snap install astral-uv`
- create your project: `uv init "myprojectname"`
- install the requirements: `uv pip install -r` requirements.txt
- install the project: `uv pip install -e .`

You are done.


### Run project

To run a race on a single track with graphical display:

- cd src/main
- uv run single_track_race_display.py

or

- uv run multi_track_race_display.py

To run many races on multiple tracks without graphical display and get the ranked performance of agents:

- cd src/main
- uv run multirace-nodisplay.py

### Create your own agent

All agents are in src/agents/

Yours is in src/agents/teamX/agentX.py, where X is the number of your team. The initial version plays random actions. Your goal is to improve your agent all along the project and win the races.

You should not touch anything else than your agent.

### Submit your agent

You have to perform a pull request on the source project: https://docs.github.com/fr/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests

The project is here: https://github.com/osigaud/pystk_project

### Watch the current results

The current race results are visible [here](https://osigaud.github.io/pystk_project), they will be updated each week.
  
## AgentSpec

Each controlled kart is parametrized by `pystk2_gymnasium.AgentSpec`:

- `name` defines the name of the player 
- `rank_start` defines the starting position (None for random, which is the default)
- `use_ai` flag (False by default) to ignore actions (when calling `step`,  a SuperTuxKart bot is used instead of using the action)
- `camera_mode` can be set to `AUTO` (camera on for non STK bots), `ON` (camera on) or `OFF` (no camera).


## Current limitations

-  the agent cannot use graphics information (i.e. pixmap)


## Environments

After importing `pystk2_gymnasium`, `supertuxkart/full-v0` is the environment containing complete observations. The observation and action spaces are both dictionaries with continuous or discrete variables (see below). The exact structure can be found using `env.observation_space` and `env.action_space`. The following options can be used to modify the environment:
    - `agent` is an `AgentSpec (see above)`
    - `render_mode` can be None or `human`
    - `track` defines the SuperTuxKart track to use (None for random). The full list can be found in `STKRaceEnv.TRACKS` after initialization with
      `initialize.initialize(with_graphics: bool)` has been called.
    - `num_kart` defines the number of karts on the track (3 by default)
    - `max_paths` the maximum number of the (nearest) paths (a track is made of paths) to consider in the observation state
    - `laps` is the number of laps (1 by default)
    - `difficulty` is the difficulty of the AI bots (lowest 0 to highest 2, default to 2)

## Action and observation space

All the 3D vectors are within the kart referential (`z` front, `x` left, `y`, `up`):
- `distance_down_track`: The distance from the start
- `energy`: remaining collected energy
- `front`: front of the kart (3D vector)
- `attachment`: the item attached to the kart (bonus box, banana, nitro/big,
  nitro/small, bubble gum, easter egg)
- `attachment_time_left`: how much time the attachment will be kept
- `items_position`: position of the items (3D vectors)
- `items_type`: type of the item
- `jumping`: is the kart jumping
- `karts_position`: position of other karts, beginning with the ones in front
- `max_steer_angle` the max angle of the steering (given the current speed)
- `center_path_distance`: distance to the center of the path
- `center_path`: vector to the center of the path
- `paths_start`, `paths_end`, `paths_width`: 3D vectors to the paths start and
  end, and vector of their widths (scalar). The paths are sorted so that the
  first element of the array is the current one.
- `paths_distance`: the distance of the paths starts and ends (vector of dimension 2)
- `powerup`: collected power-up
- `shield_time`
- `skeed_factor`
- `velocity`: velocity vector

## Wrappers

Wrappers can be used to modify the environment.

### Constant-size observation

`pystk2_gymnasium.ConstantSizedObservations( env, state_items=5, state_karts=5, state_paths=5 )` ensures that the number of observed items, karts and paths is constant. By default, the number of observations per category is 5.

### Polar observations

`pystk2_gymnasium.PolarObservations(env)` changes Cartesian coordinates to polar ones (angle in the horizontal plane, angle in the vertical plan, and distance) of all 3D vectors.

### Discrete actions

`pystk2_gymnasium.DiscreteActionsWrapper(env, acceleration_steps=5, steer_steps=7)` discretizes acceleration and steer actions (5 and 7 values respectively).

### Flattener (actions and observations)

This wrapper groups all continuous and discrete spaces together.

`pystk2_gymnasium.FlattenerWrapper(env)` flattens **actions and observations**. The base environment should be a dictionary of observation spaces. The transformed environment is a dictionary made with two entries,
`discrete` and `continuous` (if both continuous and discrete observations/actions are present in the initial environment, otherwise it is either the type of `discrete` or `continuous`). `discrete` is `MultiDiscrete`
space that combines all the discrete (and multi-discrete) observations, while `continuous` is a `Box` space.

### Flatten multi-discrete actions

`pystk2_gymnasium.FlattenMultiDiscreteActions(env)` flattens a multi-discrete action space into a discrete one, with one action per possible unique choice of
actions. For instance, if the initial space is $\{0, 1\} \times \{0, 1, 2\}$, the action space becomes $\{0, 1, \ldots, 6\}$.


## Multi-agent environment

`supertuxkart/multi-full-v0` can be used to control multiple karts. It takes an `agents` parameter that is a list of `AgentSpec`. Observations and actions are a
dictionary of single-kart ones where **string** keys that range from `0` to `n-1` with `n` the number of karts.

To use different gymnasium wrappers, one can use a `MonoAgentWrapperAdapter`.

Let's look at an example to illustrate this:

```py

from pystk_gymnasium import AgentSpec

agents = [
    AgentSpec(use_ai=True, name="Yin Team", camera_mode=CameraMode.ON),
    AgentSpec(use_ai=True, name="Yang Team", camera_mode=CameraMode.ON),
    AgentSpec(use_ai=True, name="Zen Team", camera_mode=CameraMode.ON)
]

wrappers = [
    partial(MonoAgentWrapperAdapter, wrapper_factories={
        "0": lambda env: ConstantSizedObservations(env),
        "1": lambda env: PolarObservations(ConstantSizedObservations(env)),
        "2": lambda env: PolarObservations(ConstantSizedObservations(env))
    }),
]

make_stkenv = partial(
    make_env,
    "supertuxkart/multi-full-v0",
    render_mode="human",
    num_kart=5,
    agents=agents,
    wrappers=wrappers
)
```
