# pypy-python-structure
Proposal/demo of how to incorporate pypy and python in existing conda-land

##Problem
Pypy is compatible with many pure python packages, so it is desirable to create a 
pypy package that can somehow be used with those packages.  One way to achieve this is to 
name the pypy package "python" and add a build string to differentiate from the python 
packages that are implicitly CPython.  Unfortunately, this precludes many mutex schemes 
and is otherwise unwieldy.

##Proposal
1. the "python" package should become a metapackage.  It will be versioned as
it always has been.  
2. The new python metapackage will have a dependency on either cpython (which current "python" 
recipe must be renamed to) or pypy.  This metapackage provides the abstraction over 
python implementations.
3. Default preference for implementations will be handled using "track_features" on the 
non-default metapackage.  This de-prioritizes it to the solver.  Here, this would be 
attached to the python-X.Y-pypy_0 metapackage, so that cpython remains the default.

This assumes that PyPy version numbers mean equivalent things to cpython version numbers.
If that doesn't hold, then the metapackage versions are meaningless and this scheme falls apart.

##Implementation overview
1. rename the existing "python-feedstock" recipe to "cpython-feedstock" for all 
currently maintained branches.  Rename the package that they produce to cpython.  Note 
that no recipes should directly depend on cpython.  If you think a longer, less
desirable name might make this more clear, then go with what you think best.
2. Change the Pypy recipe to create a package named pypy (again, no one should directly
install this)
3. Expand/correct the recipe in the python-selector-metapackage folder as needed
4. For things that link against C APIs, we'll need to line up the correct implementation
at runtime.  There's a python-devel metapackage that should be close to what you need
5. One fly in this ointment is the special handling that conda-build does for python
version.  It may be necessary to overhaul conda-build somewhat so that python-devel
uses the python version loops appropriately. 

##Usage examples at runtime
* using cpython will not change.  The new metapackages will fold in with the old "fat" 
packages, and aside from people noticing that the "python" package suddenly got really
small, nothing will change.  There will now be one additional package installed - the
cpython implementation package (whatever it is called).
* To choose pypy instead, users would specify a spec like `python=*=pypy`.  Again, 
this should be thought of as a strictly run-time choice.  If people hard-code python
implementations in their recipes, they will obviously not span across the languages.  

##Related schemes that may be helpful examples
* https://github.com/anacondarecipes/tensorflow_recipes - the tensorflow folder is the 
metpackage that unifies the CPU and GPU implementations
* https://github.com/conda-forge/mpi-feedstock/blob/master/recipe/meta.yaml - the mpi
metapackage at conda-forge unifies mpich and openmpi 