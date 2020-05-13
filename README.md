ommsystems
==============================
[//]: # (Badges)
[![Travis Build Status](https://travis-ci.com/REPLACE_WITH_OWNER_ACCOUNT/ProjectName.svg?branch=master)](https://travis-ci.com/REPLACE_WITH_OWNER_ACCOUNT/ProjectName)
[![AppVeyor Build status](https://ci.appveyor.com/api/projects/status/REPLACE_WITH_APPVEYOR_LINK/branch/master?svg=true)](https://ci.appveyor.com/project/REPLACE_WITH_OWNER_ACCOUNT/ProjectName/branch/master)
[![codecov](https://codecov.io/gh/REPLACE_WITH_OWNER_ACCOUNT/ProjectName/branch/master/graph/badge.svg)](https://codecov.io/gh/REPLACE_WITH_OWNER_ACCOUNT/ProjectName/branch/master)


### --- THIS IS ONLY A PREVIEW --- THE REPO CONTAINS NO FUNCTIONAL CODE, YET ---

A collection of OpenMM systems and data generated for those systems.

This package is an extension of the `openmmtools.testsystems` module with a twofold aim:
1) It contains a collection of OpenMM systems (with initial positions and topologies).
2) It provides infrastructure for fetching and sharing data (coordinates, forces, energies, velocities)
generated for these systems.

### Quickstart: Install
Make sure that you are in a Python/conda environment that has OpenMM installed.

Clone the code
```
git clone git@github.com:noegroup/ommsystems.git
```

And install
```
cd ommsystems
python setup.py install
```


### Quickstart: Example
```python
from ommsystems import *

# get data and OpenMMSystem instance
bpti_data = get_data("bpti_implicit_mini_example")
bpti = bpti_data.create_openmm_system()
print(bpti_data.info())

# download data
bpti_data.download()
bpti_data.read()
print(bpti_data.coordinates)
trajectory = bpti_data.to_mdtraj()

# evaluate energies
from simtk.openmm import LangevinIntegrator
energy_bridge = OpenMMEnergyBridge(bpti.system, LangevinIntegrator(300,1,0.0002))
energies, force = energy_bridge.evaluate(bpti_data.coordinates)
```

### Using Systems from this Repository

Each `OpenMMSystem` has a system object, a topology, a set of initial coordinates, and a unique identifier (the class 
or directory name).

Subclasses of `OpenMMSystem` can be created via their constructor:
```python
from ommsystems import BPTIImplicit
bpti = BPTIImplicit()
```

All available systems can also be created via their identifier:

```python
from ommsystems import get_system
ala = get_system("AlanineDipeptideImplicit")  # this is a system defined in openmmtools_testsystems
print(ala.info())
```

Identifiers can be:
1) Class names in `ommsystems.systems` (subclasses of `ommsystems.OpenMMSystem`)
2) Directory names in `ommsystems/systems` (each subdirectory defines a system, as specified below)
3) Class names in `ommsystems/openmmtools_testsystems` (subclasses of `openmmtools_testsystems.TestSystem`)

`OpenMMSystem` instances may have keyword arguments, for example:

```python
from simtk import unit
get_system(
    "HarmonicOscillator", 
    K=100.0*unit.kilocalories_per_mole / unit.angstroms**2, 
    mass=39.948*unit.amu
)
```


### Using Data from this Repository

The samples created for the different systems do not live in the repository 
but in some path (webserver, compute cluster, local network, local machine).
This means that not all samples are accessible for all users; they may require access to specific clusters, etc.

You can list all registered data (sets of coordinates, forces, energies, velocities) for a system:

```python
from ommsystems import list_all_data
list_all_data("AlanineDipeptideImplicit")
```

You can also list only the data sets that are accessible for you from your machine:

```python
from ommsystems import list_accessible_data
list_accessible_data("AlanineDipeptideImplicit")
```

or data that is available for a given set of constructor arguments

```python
from ommsystems import list_accessible_data
ala_samples = list_accessible_data("HarmonicOscillator", mass=39.948*unit.amu)
```

Samples also have unique identifiers (the name of a yaml file, as specified below) and can be read as follows 
```python
bpti_data = get_data("bpti_implicit_mini_example")
print(bpti_data.info())
bpti_data.download()  # gets stored in $HOME/ommsystems_data/samples
bpti_data.read() # load data into memory
bpti_data.randomize() # random permutation

# Now, randomly reordered positions and forces are accessible as numpy arrays
bpti_data.coordinates
bpti_data.forces
```

For each data set, you can also get the system
```python
bpti_data = bpti_data.create_openmm_system()
```


### Adding Systems to the Repository

The preferred way to add systems is by adding a subclass of `OpenMMSystem` to the `ommsystems.systems` module, 
see examples therein. 

### Adding Samples to the Repository

To register data for a system, add a yaml file to the ommsystems/samples directory.
The name of the yaml file serves as the identifier for the data set. The yaml file has the following format:

```
info:
    description: A short description (which method was used for sampling).
    system: The system identifier (class or directory name).
    arguments: Keyword arguments to the system constructor.
    temperature: Temperature at which samples were created (in Kelvin).
    author: Who created the data?
    date: When was the data created?
    ommsystems_version: (optional) Which version of ommsystems was used?
    openmm_version: Which version of openmm was used for sampling?
    num_frames: Number of frames contained in the data set.
    size: (approximate) size of the data [specify number in KB, MB, GB]

location:
    server: A server or hostname from which the data is accessible.
    protocol: Protocol (ftp or scp)
    root_directory: (optional) The root directory for the data files

datafiles:  
    # The data files (in format .npy or any mdtraj-readable format)
    # The number of files must be the same (or zero) for each type of data.
    # If a file (for example an HDF5 trajectory) contains multiple types of data, it can appear in multiple lines.
    coordinates: a list of filenames containing coordinates (and - for trajectory files - box dimensions)
    velocities: (optional) a list of filenames containing velocities
    forces: (optional) a list of filenames containing forces
    energies: (optional) a list of filenames containing energies

# (optional)
integrator: Serialized openmm integrator in XML format.
output_interval: Spacing between trajectory frames.
```

The integrator and output interval may be used later down the road to extend data sets.

To facilitate writing these yml files, a terminal command `ommsystems register` and a python function
`ommsystems.register` are provided.


### Customization

Instead of accessing samples from the ommsystems repository, you can also keep a local data set.
Link your local samples directory by creating a file `.ommsystems.yaml` in your home directory and specify:

```
samples_paths:
- local_directory_1
- local_directory_2
- ...
```

### Evaluating Energies

TODO: The BGTorch openmm wrapper would have a natural place in this repository.

### Creating Samples

TODO: simple API to extend a data set.

### Copyright

Copyright (c) 2020, Andreas Krämer


#### Acknowledgements
 
Project based on the 
[Computational Molecular Science Python Cookiecutter](https://github.com/molssi/cookiecutter-cms) version 1.2.
