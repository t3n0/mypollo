# Extension modules with `Cython`

Python offers the capability to import C/C++ [**extension modules**](https://setuptools.pypa.io/en/latest/userguide/ext_modules.html).
This modules will have a `.c` extension and can be built using **setuptools**, provided that the system has a **C/C++ compiler** installed.

We already covered the instructions on how to build, distribute and install pure python packages on [this notes](python-packaging.md).\
To build the extension modules, we only need a **few modification** in our project:
1. we need to write the actual `.c` modules;
2. we need a `setup.py` file along with the usual `pyproject.toml`.

## 1. Writing the `.c` modules using Cython

From the Cython official [documentation](https://cython.readthedocs.io/en/latest/src/quickstart/overview.html) we read that
"Cython is a programming language that makes **writing C extensions** for the Python language **as easy as Python itself**".

There are [two syntax variants](https://cython.readthedocs.io/en/latest/src/quickstart/cythonize.html) that can be used to write a module:
1. the **cython** variant: the module is a `.pyx` file and uses the `cdef` keyword, there is *no need* to `import cython`;
2. the **pure python** variant: the module is a standard `.py` file, we must `import cython` and declare variables following PEP-484 type hints and PEP 526 variable annotations.

In the following we use the first variant (because I like it more).

Create a `fastmodule.pyx` file containing the following
```python
def f(double x):
    return x ** 2 - x


def integrate_f(double a, double b, int N):
    cdef int i
    cdef double s
    cdef double dx
    s = 0
    dx = (b - a) / N
    for i in range(N):
        s += f(a + i * dx)
    return s * dx
```
To translate this into the extension module `fastmodule.c` simply type
```
cythonize -i -a fastmodule.pyx
```
Now our `fastmodule.c` can be imported and used from the python interpreter with `import fastmodule`. Done!

Notes:
- `-i` means inplace, i.e. the `.c` files are created in the same folder of the `.pyx` files;
- `-a` means annotation, an html file containing info about the cython to C conversion is also created (this is very important to further optimize the code).
- the speed up mostly comes from **declaring the types** of the variables inside the for loop. Cython uses the `cdef` keyword to do that;
- also, the above code is not fully optimized. The `f(x)` funcntion has no declared type. Declaring the type of `f(x)` can further boost the speed up, see [here](https://cython.readthedocs.io/en/latest/src/quickstart/cythonize.html);
- finally, I find it convenient to have a `cythonize.sh` script where I list all the modules I need to cythonize.

## 2. The `setup.py` file

Once we have all our `.c` modules in place, we just have to tell setuptools where they are in order to build the distribution.\
This is done by writing a very specific `setup.py` file in the base folder, close to the usual `pyproject.toml`.\
Supposed we have the following folder structure:

```
base_folder/
├── README.md
├── LICENSE
├── pyproject.toml
├── setup.py           <--- we need a very basic setup.py file
├── carwash/
│   ├── __init__.py
│   ├── washing.py
│   ├── fastmodule.c   <--- the .c module we created previously
│   ├── slowmodule.py  <--- not necessary, just for benchmark
│   └── drying.py
└── tools/
    └── spray.py
```

The content of the `setup.py` file is

```python
from setuptools import setup, Extension

myextensions = [
    Extension(name = "carwash.fastmodule", sources = ["carwash/fastmodule.c"])
]

setup( ext_modules = myextensions )
```

As [usual](python-packaging.md), to **build the above** we type `python -m build`.\
This creates the source and built distributions (`.tar.gz` and `.whl`) ready to be installed or shared.

Notes:
- the `name` is the extension is how we call our module (i.e. `import carwash.fastmodule as fm`), it must match the file name;
- the `ext_modules` argument wants a `list` of extensions objects.

## 3. Workflow and testing the speed-up

We can now put everything together and try to build, install and test our speed-up package.\
Download the [folder zip](https://github.com/t3n0/notes/raw/main/python/cython.zip) with the project.

1. Extract it at your favorite location and dove to the base folder;
1. Cythonize the `.pyx` modules (i.e. create the `.c` modules);
```bash
./cythonize.sh
```
3. Build the package (i.e. create the `.tar.gz` and `.whl` distributions);
```bash
python -m build
```
4. Optional, distribuite it (with `twine` or via a github release);
5. Install it
```bash
pip install ./dist/carwash-0.1.tar.gz
```

We can test wheter it works from a python interpreter. On my machine I see
```bash
Python 3.11.5 (main, Sep 11 2023, 13:23:44) [GCC 11.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import carwash.washing as w
Hello from the washing module
>>> import carwash.drying as d
Hello from the drying module
>>> import carwash.fastmodule as fm
>>> import carwash.slowmodule as sm
>>> fm.integrate_f(1,4,10000)
13.498200045
>>> import timeit
>>> timeit.timeit('sm.integrate_f(1,4,10000)', number=1000, setup='import carwash.slowmodule as sm')
2.933193664997816
>>> timeit.timeit('fm.integrate_f(1,4,10000)', number=1000, setup='import carwash.fastmodule as fm')
0.9179619419737719
>>> 
```

We got more than a **3x** speed-up! And we can get more than **100x** if we declare the type of function `f(x)`!
