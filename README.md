# SemiInfiniteOptimization

### Kris Hauser

### 10/30/2018

### kris.hauser@duke.edu


This package contains code accompanying the paper
"[Semi-Infinite Programming for Trajectory Optimization with Nonconvex Obstacles](http://motion.pratt.duke.edu/papers/WAFR2018-Hauser-SemiInfinite.pdf)"
by K. Hauser, in Workshop on the Algorithmic Foundations of Robotics (WAFR), 2018.


![Animation of live optimization of a trajectory with a tree obstacle](http://motion.pratt.duke.edu/videos/wafr2018/trajectory_tree.gif) ![Animation of live optimization of a trajectory with a chair obstacle](http://motion.pratt.duke.edu/videos/wafr2018/trajectory_chair.gif) ![Real-time optimization of a robot pose in the presence of a chair obstacle](http://motion.pratt.duke.edu/videos/wafr2018/moving_chair.gif)


## File structure

```
├── data                      World, robot, and object files for running the example code
|   └─── ...
├── geomopt.py                A geometry - geometry collision optimization example program
├── README.md                 This file
├── resources                 Path files for running the trajectory optimization example code
|   └─── ...
├── robotposeopt.py           A robot pose - geometry collision optimization example program
├── robottrajopt.py           A robot pose - geometry collision optimization example program
└── semiinfinite/             The core Python module
    ├── geometryopt.py        SIP code for collision-free constraints between geometries, for objects, robot poses, and robot trajectories.
    ├── __init__.py           Tells Python that this is a module
    └── sip.py                Generic semi-infinite programming code
```


## Dependencies

This package requires

1. Numpy/Scipy

2. [OSQP](http://osqp.org) for quadratic program (QP) solving.  OSQP can be
   installed using

> pip install osqp

   Other solvers might be supported in the future.

3. The [Klampt](https://klampt.org) 0.8.x Python API (https://klampt.org) to be installed.  `pip install klampt` may work.


## Basic usage:

Copy the semiinfinite folder to your desired project, or create a setup.py to install this into your Python
site-packages, if you prefer.

```python
from klampt import *
from semiinfinite.geometryopt import PenetrationDepthGeometry,optimizeCollFree
from semiinfinite.sip import SemiInfiniteOptimizationSettings

# ... TODO: setup Klamp't Geometry3D or RigidObjectModel objects obj1 and obj2 here ...
# For example,
# obj1 = Geometry3D()
# obj1.loadFile("data/cube.off")
# obj2 = Geometry3D()
# obj2.loadFile("data/m797.off")

# Define a volumetric grid resolution
gridres = 0.05
# Define a point cloud resolution
pcres = 0.02

geom1 = PenetrationDepthGeometry(obj1,gridres,pcres)
geom2 = PenetrationDepthGeometry(obj2,gridres,pcres)

# ... TODO: setup appropriate object transforms

geom1.setTransform(obj1.getTransform())        
geom2.setTransform(obj2.getTransform())

# The optimizer will optimize the transform of object 1 with object 2 fixed.
Tinit = obj1.getTransform()   # Initial transform
Tdes = None   # Desired transform.  This can be None, in which case the optimizer assumes Tdes=Tinit

# Run the optimizer with default settings.
# You can pass SemiInfiniteOptimizationSettings object into the settings argument if you want to
# configure the solver.
Tcollfree,trace,cps = optimizeCollFree(geom1,geom2,Tinit,Tdes,verbose=0,settings=None)

# Tcollfree is the solved rigid transform, in klampt.math.se3 format.
# You may want to test the soluiton for collision.
geom1.setTransform(Tcollfree)
if geom1.distance(geom2) < 0:
    print "Couldn't solve for a collision free configuration"

# If you want to update the Klamp't object, call this...
obj1.setTransform(*Tcollfree)

```



## Running demos

Basic geometry - geometry collision testing:

> python geomopt.py data/cube.off

> python geomopt.py data/cube.off data/m797.off

> python geomopt.py data/cube.off data/scene2_1.pcd


Robot pose optimization testing:

> python robotposeopt.py data/tx90_geom_test.xml

> python robotposeopt.py data/tx90_geom_test2.xml

> python robotposeopt.py data/tx90_geom_test3.xml


Trajectory optimization testing

> python robottrajopt.py data/tx90_geom_test3.xml

The example files in data/*.xml assume the Klampt folder is two levels up from this folder.  If your Klampt
folder is somewhere else, change the paths accordingly.

In each of these demos, you can drag around the object transform and observe the results of the optimization.

