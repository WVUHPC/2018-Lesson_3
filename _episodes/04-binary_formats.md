---
title: "Binary formats: NetCDF and HDF5"
teaching: 105
exercises: 15
questions:
- "How to store large amounts of numerical data?"
objectives:
- "Learn how to store big numerical arrays in NetCDF and HDF5 formats."
keypoints:
- "Storing large numerical arrays as text is highly inefficient both for humans and machine. NetCDF and HDF5 are de-facto standards for storing numerical data and metadata."
---

## Binary Formats

### NetCDF

Lets use the python version of NetCDF4. There are several implementations of this format, but we will use the official from
[UniData.org](unidata.org)

This is the problem, lets suppose that we need an efficient way of storing the positions and forces of a protein composed by 100000 atoms.

We can store both forces and positions as text files. Doing that will use 7.2MB for each file. Now imagine that we are doing some Molecular Dynamics simulation where the positions and forces are changing during thousands of iterations, we will end up generating thousands of files each of them 7.2 MB. The storage usage scales pretty quickly.

We will present here and alternative, that uses a binary format that fast and uses storage more efficiently than a text format.

This exercise will be executed with IPython

~~~
from netCDF4 import Dataset
import numpy as np
~~~
{: .source}

Lets declare the number of variables as 100000 as we need that number in several places.

~~~
NATOMS=100000
~~~
{: .source}

Here we open the NetCDF file. It is a binary file that internally follows the HDF5 file format a Standard File Format for large numerical data.

~~~
rootgrp = Dataset("MD.nc", "w", format="NETCDF4")
~~~
{: .source}

With NetCDF you can create a complete tree of data structures. We do not need to do this for this case, but this would show how we can create two groups, for positions and forces.

~~~
positions = rootgrp.createGroup("positions")
forces = rootgrp.createGroup("forces")
~~~
{: .source}

Each group can have its own variables and metadata. But before we add the actual contents we need to declare the dimensions of the data.
To simplify the exercise we will just store one iteration of our Molecular Dynamics simulation. Two dimensions are needed, the 3 spatial coordinates is one dimension and the indices of the atoms is the second dimension.

~~~
pxyz=positions.createDimension("XYZ", 3)
patoms=positions.createDimension("Atoms", NATOMS)
~~~
{: .source}

We do exactly the same for forces.

~~~
fxyz=forces.createDimension("XYZ", 3)
fatoms=forces.createDimension("Atoms", NATOMS)
~~~
{: .source}

Now we are ready to create one Variable for each group and we use the dimensions above to define the shape of the arrays that will be stored.
We also mention to NetCDF that we want to store each number a double precision float "f8".

~~~
pvalues=positions.createVariable("Values","f8",(Atoms","XYZ"))
fvalues=forces.createVariable("Values","f8",(Atoms","XYZ"))
~~~
{: .source}

One of the big advantages of Binary formats like NetCDF is the ability to store Metadata, Information about the contents of some data that can be relevant to interpret correctly. If we store just the numbers, other party could not know if the values in different units.

So to clarify that we will use add one attribute saying that our positions are in `Angstrom` and the Forces are in `eV/Angstrom`

~~~
pvalues.units = "Angstrom"
fvalues.units = "eV/Angstrom"
~~~
{: .source}

Here is the point we we introduce the data, we do not have actual positions and forces for 100000 so to mimic that we use random numbers here.

~~~
rpos=np.random.rand(NATOMS,3)
rfor=np.random.rand(NATOMS,3)
~~~
{: .source}

Inserting the values inside the corresponding variables is a easy as assign numpy arrays to the objects.

~~~
pvalues[:,:] = rpos
fvalues[:,:] = rfor
~~~
{: .source}

Finally, all that we have to do is close the Root Group and the file is save.

~~~
rootgrp.close()
~~~
{: .source}
~~~

### HDF5

HDF5 allows to store data and metadata in a binary format, it could be as simple as storing a single table and as complex as an hierarchical multilevel structure
of tables, where values can be either matrices, scalars or vectors.

Here we are presenting a simple example, with one dataset storing a matrix of floating point numbers

~~~

/*
 *  This example illustrates how to create a dataset
 *  array.  It is used in the HDF5 Tutorial.
 */

#include <stdio.h>
#include <stdlib.h>
#include "hdf5.h"

#define H5FILE "dset.h5"
#define NX     100000                      /* dataset dimensions */
#define NY     3


int main() {

  FILE        *fp;
  hid_t       file_id, dataset_id, dataspace_id;  /* identifiers */
  hsize_t     dims[2];
  herr_t      status;
  double      data[NX][NY];          /* data to write */
  int         i, j;

  /*
   * Data  and output buffer initialization.
   */
  for (j = 0; j < NX; j++) {
    for (i = 0; i < NY; i++)
      data[j][i] = drand48();
  }

  /* Create a new file using default properties. */
  file_id = H5Fcreate(H5FILE, H5F_ACC_TRUNC, H5P_DEFAULT, H5P_DEFAULT);

  /* Create the data space for the dataset. */
  dims[0] = NX;
  dims[1] = NY;
  dataspace_id = H5Screate_simple(2, dims, NULL);

  /* Create the dataset. */
  dataset_id = H5Dcreate2(file_id, "/dset", H5T_NATIVE_DOUBLE, dataspace_id,
                          H5P_DEFAULT, H5P_DEFAULT, H5P_DEFAULT);

  /*
   * Write the data to the dataset using default transfer properties.
   */
  status = H5Dwrite(dataset_id, H5T_NATIVE_DOUBLE, H5S_ALL, H5S_ALL,
		    H5P_DEFAULT, data);

  /* End access to the dataset and release resources used by it. */
  status = H5Dclose(dataset_id);

  /* Terminate access to the data space. */
  status = H5Sclose(dataspace_id);

  /* Close the file. */
  status = H5Fclose(file_id);

  fp=fopen("dset.txt","w");
  for (j = 0; j < NX; j++) {
    for (i = 0; i < NY; i++)
      fprintf(fp, "%.16e ", data[j][i]);
    fprintf(fp, "\n");
  }
  fclose(fp);

}
~~~
{: .source}


{% include links.md %}
