Durin
=====

XDS plugin for reading HDF5 files following the NeXuS format or those written by Dectris Eiger detectors.

See:
* https://www.nexusformat.org
* https://www.dectris.com/features/features-eiger-x/hdf5-and-nexus
* https://strucbio.biologie.uni-konstanz.de/xdswiki

## Get Durin
A pre-built durin plugin can be downloaded from the github releases attached to this repository.
Latest release at https://github.com/DiamondLightSource/durin/releases/latest

## Usage
In your XDS.INP add:
```
LIB=[path to durin-plugin.so]
NAME_TEMPLATE_OF_DATA_FRAMES=[data_path]/data_images_??????.h5
```
XDS will instruct the plugin to load `[data_path]/data_images_master.h5` and this must be the
Eiger master file or the NeXus file for the data collection.

It is generally assumed that the files `[data_path]/data_images_data_xxxxxx.h5` contain the actual
datasets and the master file contains HDF5 external links to these, but that is not required, so long as
the master file contains an `NXdata` or `NXdetector` group with either a dataset named `data` or a
series of datasets named `data_000001`, `data_000002`, etc.. If both are present, the dataset named
`data` will be prioritised.


## Building the Durin Plugin

If you'd like to build the plugin yourself, follow the instructions below.

### Requirements
- cmake version >= 3.19 is required to build Durin.
- No manual HDF5 installation is required by default. The CMake build will fetch and build a
	compatible HDF5 as a subproject so users do not need to download or install HDF5 themselves.
- If you prefer to build against a system-installed HDF5, see the "Use system HDF5" section below.

### Build
Durin is now built using CMake. By default the CMake configuration will download and build a
compatible HDF5 dependency automatically, and will also fetch and build the `bitshuffle` library
required for data filtering in virtual datasets. This means you can build the plugin without 
installing HDF5 or bitshuffle yourself.

Example build commands (from the root directory of this repository):
```
mkdir build
cd build
cmake ../
cmake --build .
```
After a successful build the plugin will be available at `build/durin-plugin.so`.

### Use system HDF5 (optional)

If you want to build against a preinstalled HDF5 instead of the bundled subproject, disable
the in-tree HDF5 build and point CMake at your HDF5 installation. Set `BUILD_HDF5` to `OFF`:

```
cmake .. -DBUILD_HDF5=OFF
cmake --build . 
```

When using a system HDF5, it must have been built with the options required by the plugin:
thread-safety, the high-level library, and deprecated/compatibility symbols. For source builds
these typically correspond to the configure flags shown below (or the equivalent for your
distribution/build system):

```
export CFLAGS=-fPIC
../configure --enable-threadsafe --enable-deprecated-symbols --enable-hl --enable-unsupported
make
make check
make install
```

The CMake option to use system HDF5 is provided for advanced users; the default bundled build
should be sufficient for most workflows and avoids the need to manage HDF5 installation manually.


## Example XDS.INP
```
DETECTOR=PILATUS MINIMUM_VALID_PIXEL_VALUE=0 OVERLOAD=4096
LIB=/opt/durin/build/durin-plugin.so
SENSOR_THICKNESS= 0.450
!SENSOR_MATERIAL / THICKNESS Si 0.450
!SILICON= 3.953379
DIRECTION_OF_DETECTOR_X-AXIS= 1.00000 0.00000 0.00000
DIRECTION_OF_DETECTOR_Y-AXIS= 0.00000 1.00000 0.00000
DETECTOR_DISTANCE= 194.633000
ORGX= 1041.30 ORGY= 1160.90
ROTATION_AXIS= 0.00000 -1.00000 -0.00000
STARTING_ANGLE= -30.000
OSCILLATION_RANGE= 0.100
X-RAY_WAVELENGTH= 0.97891
INCIDENT_BEAM_DIRECTION= -0.000 -0.000 1.022
FRACTION_OF_POLARIZATION= 0.999
POLARIZATION_PLANE_NORMAL= 0.000 1.000 0.000
NAME_TEMPLATE_OF_DATA_FRAMES= ../image_9264_??????.h5
TRUSTED_REGION= 0.0 1.41
DATA_RANGE= 1 600
JOB=XYCORR INIT COLSPOT IDXREF DEFPIX INTEGRATE CORRECT
```

N.B. the master file is needed, not the .nxs one which follows the
standard.
