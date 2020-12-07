## installing parallel h5py on ubuntu 20.04

will install to $HOME/opt/<br>
will link binaries to $HOME/bin/

Sources:<br>
Download MPICH here: https://www.mpich.org/downloads/<br>
Download HDF5 here: https://www.hdfgroup.org/downloads/hdf5/source-code/

A few settings upfront, set accordingly: 
```shell
export mpichfile=mpich-3.3.2.tar.gz
export mpichsrc=mpich-3.3.2
export hdf5file=hdf5-1.12.0.tar.gz
export hdf5src=hdf5-1.12.0
```

### Install MPICH
```shell
export CC=gcc
cd $HOME/Downloads
tar xzfv $mpichfile
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
cd $HOME/Downloads
tar xzfv $hdf5file
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

### Install h5py (will install mpi4py automatically)
```shell
export CC=mpicc
export HDF5_MPI=ON
export HDF5_DIR=$HOME/opt/$hdf5src
pip install --no-binary=h5py h5py
```
