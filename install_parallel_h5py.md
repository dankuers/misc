## installing parallel h5py on ubuntu 20.04

will install to $HOME/opt/<br>
will link binaries to $HOME/bin/

Sources:<br>
Download MPICH here: https://www.mpich.org/downloads/<br>
Download mpi4py here: https://bitbucket.org/mpi4py/mpi4py/downloads/
Download HDF5 here: https://www.hdfgroup.org/downloads/hdf5/source-code/

Name of the source folders, set accordingly: 
```shell
export mpichsrc=mpich-3.3.2
export mpi4pysrc=mpi4py-3.0.3
export hdf5src=hdf5-1.12.0
```

Note: Each install starts at your download folder with you having extracted the files.

### Install MPICH
```shell
export CC=gcc
cd $mpichsrc
mkdir -p $HOME/opt/$mpichsrc
# ./configure --enable-shared --disable-fortran --prefix=$HOME/opt/$mpichsrc --with-device=ch3:sock
./configure --enable-shared --disable-fortran --prefix=$HOME/opt/$mpichsrc
make
# make check
make install
cd $HOME
mkdir bin
cp -rs $HOME/opt/$mpichsrc/bin .
```

### Install HDF5 with parallel support
```shell
export CC=mpicc
cd $hdf5src
mkdir -p $HOME/opt/$hdf5src
./configure --enable-shared --enable-parallel --prefix=$HOME/opt/$hdf5src
make
# make check
make install
cd $HOME
mkdir bin
cp -rs $HOME/opt/$hdf5src/bin .
```

Note: Switch to your desired virtual enviornment.

### Install mpi4py
```shell
cd $mpi4pysrc
python setup.py build --mpicc=$HOME/opt/$mpichsrc/bin/mpicc
python setup.py install
```

Test it:
```shell
mpiexec -n 4 python -m mpi4py.bench helloworld
```
Output should be:
```
Hello, World! I am process 0 of 4 on [yourmachine].
Hello, World! I am process 1 of 4 on [yourmachine].
Hello, World! I am process 2 of 4 on [yourmachine].
Hello, World! I am process 3 of 4 on [yourmachine].
```

### Install h5py
```shell
export CC=$HOME/opt/$mpichsrc/bin/mpicc
export HDF5_MPI="ON"
export HDF5_DIR=$HOME/opt/$hdf5src
pip install --no-binary=h5py h5py
```

Test it:
```shell
mpiexec python -n 4 python -c "exec(\"from mpi4py import MPI\nimport h5py\nrank = MPI.COMM_WORLD.rank\nf = h5py.File('parallel_test.hdf5', 'w', driver='mpio', comm=MPI.COMM_WORLD)\ndset = f.create_dataset('test', (4,), dtype='i')\ndset[rank] = rank\nf.close()\")" && h5dump parallel_test.hdf5
```
Output should be:
```
HDF5 "parallel_test.hdf5" {
GROUP "/" {
   DATASET "test" {
      DATATYPE  H5T_STD_I32LE
      DATASPACE  SIMPLE { ( 4 ) / ( 4 ) }
      DATA {
      (0): 0, 1, 2, 3
      }
   }
}
}
```

## Troubleshooting
- Some 8.x.x version of gcc didn't work for the HDF5 compilation, so instead I used 5.4.0 and then it worked.
- When different versions of MPI were used to build HDF5 and mpi4py then some error came up. But this shouldn't be the case if you installed all of the above yourself (and therefore used the same MPI).
