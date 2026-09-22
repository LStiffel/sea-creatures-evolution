# Sea Creatures Evolution

A 2D soft-body physics engine and fluid simulation, both written from scratch,
used as the environment for a genetic algorithm that evolves the body shape
and movement pattern of simple aquatic creatures from nothing.

## How it works

- **`physics.py` — the engine.** Point masses (`Point`) connected by rigid
  `Link`s, integrated with Verlet integration and constraint solving (link
  length + point collision), no physics library involved.
- **`fluid.py` — custom hydrodynamics.** Two hand-derived force laws:
  perpendicular drag on every body segment, and a "wedge" resistance at each
  joint that resists fast angle changes — together they're what make flicking
  a limb actually push the creature through water instead of just flailing.
- **`creature.py`** — a creature is a tree of points and links built up one
  segment at a time, with joint torques applied around any three connected
  points (`apply_joint_force`) so a genome can actuate it.
- **`genome.py`** — encodes both the *body* (how many points, how they're
  connected, at what angles) and the *brain* (a cycle of poses/target angles)
  as flat gene arrays, with `random_genome`, `decode_genome`, `mutate` and
  `crossover` to turn genes into a simulate-able `Creature`.
- **`brain.py`** — the decoded "brain" is a simple timed pose cycle: hold a
  set of joint target angles for a duration, then switch to the next pose,
  looping — evolution is what shapes which poses and timings actually work.
- **`simulation.py`** — the fitness function: distance travelled by the
  creature's center of mass over a fixed simulated duration, penalized for
  energy use and for the body self-intersecting.
- **`evolution.py`** — the genetic algorithm: a population of 1000 creatures,
  parallelized fitness evaluation across CPU cores (`multiprocessing.Pool`),
  elitism (top 500 kept each generation) plus crossover/mutation for the
  rest. Every generation is logged (`Gen NNNN | best / avg / worst`) and
  checkpointed to `evolution_data/gen_NNNN.json`.
- **`main.py`** — a live Pygame viewer for a single random creature (handy
  for iterating on the physics/genome encoding).
- **`replay.py`** — loads any saved generation and replays one creature from
  it in Pygame, so you can watch what a given generation actually evolved.

`evolution_data/` in this repo already has 100 saved generations from a full
run.

## Running it

```bash
pip install pygame numpy
python main.py                          # watch one random creature live
python evolution.py                     # run the GA (headless, saves every generation)
python replay.py --gen 100 --index 0    # replay the fittest creature of a saved generation
```

## Notes

This started as the Python prototype; [`sea-creatures-evolution-js`](https://github.com/LStiffel/sea-creatures-evolution-js)
is a JavaScript port of the same idea for the browser.
