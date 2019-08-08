---
title: "Miniworldmakerv1"
date: 2019-08-07T15:50:28+02:00
draft: true
type: post
author: asbl
draft: false
categories:
  - Python
  - miniworldmaker
---

The Miniworldmaker now has version 1.0! (strictly speaking already 1.0.3)

That doesn't mean that it's bug-free (I've actually fixed a few minor bugs) or you won't encounter unexpected problems - I'll have a lot of work to do next year. 
But now the API looks exactly the way I want it to.

### Physics

Objects of a class are now physically simulated if the method setup_physics() is implemented in the class.

In the method you can define the different physical properties of the object. The method setuo_physics is called automatically before the method on_setup. 

Look at the following code from the game [Angry Birds](https://github.com/asbl/miniworldmaker/tree/master/examples/games/angry)

```python
class Bird(Actor):

    def on_setup(self):
        self.add_image("images/fly.png")
        self.orientation = 180
        self.flip_x()
        self.size = (40, 40)
        self.direction = self.board.arrow.direction + 180
        self.physics.impulse_in_direction(5000)

    def setup_physics(self):
        self.physics.mass = 6
        self.physics.riction = 1
        self.physics.size = 0.7, 0.7
        self.physics.gravity = True
        self.physics.shape_type = "circle"
        self.physics.stable = False
```

The setup_physics method determines the properties of the bird in an Angry-Bird game:

  * The mass is 6, the friction 1.
  * The size of the physical object is 0.7 times the size of the original object. This makes collisions appear slightly smoother.
  * The object follows the rules for gravity.
  * The bird is physically simulated as a circle. It is not stable, i.e. the torque of the bird is also calculated.

Since the physics engine was initialized before the setup method, the object can be manipulated in the setup method. Here an impulse is generated on the bird.

### Collision Handling

The on_sensing methods can now be called directly. See the bird (this time from the game [Flappy Bird](https://github.com/asbl/miniworldmaker/tree/master/examples/games/flappy)):

```python
class Bird(Token):

    def on_setup(self):
        self.add_image("images/fly.png")
        self.size = (60, 60)
        self.costume.orientation = 180
        self.flip_x()

    def setup_physics(self):
        pass

    def on_sensing_borders(self, borders):
        if "bottom" in borders or "top" in borders:
            self.board.is_running = False

    def on_sensing_pipe(self, other):
        self.board.is_running = False
        self.board.reset()
```

If the method on_sensing_borders(borders) is implemented, the main loop is checked in each pass whether the bird touches one of the borders.

The on_sensing_pipe(other) method refers to the Pipe class. If the bird "detects" an object of the class Bird in its place, then the MEthode is called.

### Collision Handling with Physics

The method above for collision detection does not work with the physics engine because objects do not overlap during collisions. Here you can use the methods on_sensing_collision_with_CLASS and on_sensing_separation_with_CLASS, where CLASS should be the class name of the objects used to check a collision.

This looks like the following for Pong:

```python
class Ball(Circle):

    def on_setup(self):
        self.direction = 30
        self.physics.impulse_in_direction(300)

    def setup_physics(self):
        self.physics.mass = 1
        self.physics.elasticity = 1
        self.physics.friction = 0
        self.physics.shape_type = "circle"
        self.physics.gravity = False
        self.physics.stable = False

    def on_sensing_collision_with_border(self, line, collision):
        if line == self.board.left:
            self.board.points_right.inc()
        if line == self.board.right:
            self.board.points_left.inc()
```


Translated with www.DeepL.com/Translator