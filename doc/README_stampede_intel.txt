Setup instructions for TACC Stampede using Intel-18.0.2

module purge
#
module load libfabric/1.7.0
module load git/2.9.0
module load autotools/1.1
module load xalt/2.7.9
module load TACC
#
module load python2/2.7.15
module load intel/18.0.2
module load cmake/3.16.1
module load impi/18.0.2
module load pnetcdf/1.11.0
module load netcdf/4.6.2
module li

> Currently Loaded Modules:
>  1) pnetcdf/1.11.0   2) netcdf/4.6.2   3) libfabric/1.7.0   4) intel/18.0.2   5) impi/18.0.2   6) git/2.9.0   7) autotools/1.1   8) python2/2.7.15   9) cmake/3.16.1  10) xalt/2.7.9  11) TACC

export CC=icc
export FC=ifort
export CXX=icpc
export NETCDF=${TACC_NETCDF_DIR}
export HDF5_ROOT=/opt/apps/intel18/hdf5/1.10.4/x86_64

mkdir -p $WORK/NCEPLIBS-ufs-v1.0.0/src
# for this particular user, this corresponds to /work/06146/tg854455/stampede2/NCEPLIBS-ufs-v1.0.0/src

cd $WORK/NCEPLIBS-ufs-v1.0.0/src
git clone -b ufs-v1.0.0 --recursive https://github.com/NOAA-EMC/NCEPLIBS-external
cd NCEPLIBS-external
mkdir build && cd build
# If netCDF is not built, also don't build PNG, because netCDF uses the default (OS) zlib in the search path
cmake -DBUILD_MPI=OFF -DBUILD_PNG=OFF -DBUILD_NETCDF=OFF -DCMAKE_INSTALL_PREFIX=$WORK/NCEPLIBS-ufs-v1.0.0 .. 2>&1 | tee log.cmake
make VERBOSE=1 -j8 2>&1 | tee log.make

cd $WORK/NCEPLIBS-ufs-v1.0.0/src
git clone -b ufs-v1.0.0 --recursive https://github.com/NOAA-EMC/NCEPLIBS
cd NCEPLIBS
mkdir build && cd build
cmake -DEXTERNAL_LIBS_DIR=$WORK/NCEPLIBS-ufs-v1.0.0 -DCMAKE_INSTALL_PREFIX=$WORK/NCEPLIBS-ufs-v1.0.0 .. 2>&1 | tee log.cmake
make VERBOSE=1 -j8 2>&1 | tee log.make
make install 2>&1 | tee log.install


- END OF THE SETUP INSTRUCTIONS -


The following instructions are for building the ufs-weather-model (standalone;
not the ufs-mrweather app - for the latter, the model is built by the workflow)
with those libraries installed.

This is separate from NCEPLIBS-external and NCEPLIBS, and details on how to get
the code are provided here: https://github.com/ufs-community/ufs-weather-model/wiki

After checking out the code and changing to the top-level directory of ufs-weather-model,
the following commands should suffice to build the model.


module purge
#
module load libfabric/1.7.0
module load git/2.9.0
module load autotools/1.1
module load xalt/2.7.9
module load TACC
#
module load python2/2.7.15
module load intel/18.0.2
module load cmake/3.16.1
module load impi/18.0.2
module load pnetcdf/1.11.0
module load netcdf/4.6.2

export CC=icc
export FC=ifort
export CXX=icpc
export NETCDF=${TACC_NETCDF_DIR}

. $WORK/NCEPLIBS-ufs-v1.0.0/bin/setenv_nceplibs.sh
export CMAKE_Platform=stampede.intel
./build.sh 2>&1 | tee build.log
