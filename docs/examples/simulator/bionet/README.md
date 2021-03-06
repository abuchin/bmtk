# BioNet: a Python interface to NEURON for large-scale network simulations

## Getting started
This brief tutorial explains how to run simulations with BioNet using one of the provided example networks. 
The instructions apply for Linux platform, although the package was also tested on MacOS.

BioNet is a component of the Brain Modeling ToolKit package.
Thus, to install BioNet you will need to install the Brain Modeling Toolkit. 

Go to the package's main page: https://github.com/AllenInstitute/bmtk.

Click "Clone or download" button and download the zip file to the desired folder. 

cd to the folder with the downloaded file and unzip the file.:
```bash
$ unzip bmtk-develop
```
You should now have the folder `bmtk-develop` containing the Brain Modeling Toolkit package.

Add a path to the `bmtk-develop` folder to the environment variable PYTHOPATH in your `.bashrc` file.

### Dependencies:

#### Python
BioNet package requires [Python 2.7](www.python.org). 

If you do not have Python already installed, we recommend installing [Anaconda](https://www.anaconda.com/download/) distribution of Python that comes preloaded with many packages and allows an easy installation of additional packages.

#### Python packages
BioNet has the following Python package dependancies:

numpy 1.10

pandas 0.19.2

h5py 2.6

jsonschema

***Kael, what should I write about setup.py?

If you have just installed Python then you will also need to update your environemnt variables:
include a path to Python to your PATH environment variable 
include path to Python's site packages to PYTHONPATH. 

For instance, for Anaconda you will need to add 

`/path/to/python/anaconda2/bin/` to your PATH

`/path/to/python/anaconda2/lib/python2.7/site-packages` to your PYTHOPATH.

where `/path/to/python` corresponds to the folder where Anaconda Python is installed.

At this stage try running python and importing Brain Modeling Toolkit as follows:

```bash
$ python
>>> import bmtk
>>> exit()
```
#### NEURON

Additionally, BioNet requires [NEURON](http://www.neuron.yale.edu/neuron/download/) simulator (version 7.4.rel-1370 or or later). Detailed installation instructions may be found in the Appendix in Hines et al. Frontiers of Neuroinformatics, 2009. 

Since BioNet does not require NEURON's GUI functionality, one may install NEURON without GUI support (i.e., without needing to install InterViews). For convenience we provide the installation instructions of NEURON from the source code on Linux platform without InterViews below:

download and unzip a nrn-7.4.rel-1370.tar.gz version of neuron:
```bash
$ wget http://www.neuron.yale.edu/ftp/neuron/versions/v7.4/v7.4.rel-1370/nrn-7.4.rel-1370.tar.gz
$ tar xf nrn*gz
```
This should have create a `nrn-7.4` directory. 

Other versions of NEURON may be found following the link http://www.neuron.yale.edu/ftp/neuron/versions/` and similarly installed.

Now you will configure and compile binaries.

If you want to run NEURON in parallel, it requires a version of [MPI](http://www.mpich.org/) to be installed. If you have MPI installed you will configure installation as follows: 

change to the newly created `nrn-7.4` directory and configure installation:

```bash
$ ./configure --prefix=`pwd` --with-paranrn --with-nrnpython --without-x have_cython=no BUILD_RX3D=0
```
otherwise remove --with-paranrn from configuration options:

```bash
$ ./configure --prefix=`pwd` --with-nrnpython --without-x have_cython=no BUILD_RX3D=0
```

finally compile the code:

```bash
make -j 6 install
```

Finally, you will need to add a path to: 
`/your/home/neuron/nrn-7.4/x86_64/bin` to your PATH variable and also
`/your/home/neuron/nrn-7.5/lib/python to your PYTHONPATH variables

where `/your/home` is the location where NEURON was installed.

If installed correctly you should be able to do the following:

```bash
$ nrniv -python
>>> import bmtk
>>> import neuron
>>> exit()
```

If everything is installed correctly, you are now ready to run the examples.

### Running examples

To run examples go to the folder `bmkt-develop` package is installed and cd to the examples folder for BioNet 

```bash
cd ./docs/examples/simulator/bionet/
```

There you will find 4 folders with examples of pre-built and ready to run networks:

`14cells`, `14cells_nsyns`: network of 14 cells with more and less detailed specification of connectivity

`450cells`, `450cells_nsyns`: network of 450 cells with more and less detailed specification of connectivity

as well as a `components` folder including files defining models of individual cells, synapses, electrodes etc., used by the above networks.

Prior to running simulations you need to compile NEURON mechanisms located in the `components` folder:

```bash
$ cd  ./components/mechanisms/
$ nrnivmodl modfiles
```

which will create `/x86_64` folder

now cd to the specific example such as `14cells`

inside each example folder you will find:

`run_bionet.py` : main python script which calls BioNet to run simulation

`config.json` : configuration file incuding paths to files describing simulated network as well as run parameters

`./network` : folder including files describing parameters of the nodes and edges making up the network

set_cell_params.py set_syn_params.py set_weights.py : modules which describe how models of cells, synapses and connection weights are created.

To run simulation on a single processor:
```bash
$ nrniv -python run_bionet.py config.json
```
If you have MPI installed you may run simulation in parallel (e.g. on 4 processor) as follows:

```bash
$ mpirun -np 4 nrniv -mpi -python run_bionet.py config.json
```

The code will first check whether the folders listed in the configuration file `config.json` are are present in the file system and will provide an informative error message otherwise. If you receive an error message, check the paths in the configuration file.

Upon succesful validation of configuration BioNet will proceed to load the network, simulate the activity and save the results to files in the output directory.

In the output directory you will find:
`spikes.h5`: hdf5 file containing spike information.
`spikes.txt`: same as above, but using ascci file for users not familiar with hdf5.
`./cellvars`: an folder including hdf5 files with simulation output for individual select cells. 
`config.json`: a copy of configuration file for the record keeping
`log.txt`: run log file including time-stemped information about simulation run progress. 


### Simulating your network models

To run simulations of your network with BioNet, you will first need to provide a pre-built network in the format understood by BioNet. We recommend using BMTK's builder tools to build simuations, but you may also use your own scripts or a third party tool. As a start we suggest to modify the existing network examples as a quick way of customizing network models. When you become familiar with model structure, we suggest to study the example for building a network of e.g. [14cells](https://github.com/AllenInstitute/bmtk/tree/develop/docs/examples/builder/bionet_14cells)

When you have your custom model, you will need to specify in the your configuration file the paths to the network, components as well as simulation run parameters.

Just as in the above examples, your run folder should include Python modules: set_cell_params.py, set_syn_params.py, set_weights.py specifying how models of cells, synapses and connection weights are created as well as a main python script. 

When running different simulations you will rarely need to modify the main Python script. Instead, you will commonly need to modify paths or parameters in the configuration file to instruct BioNet which model to run and how to run it. Thus we included a separate tutorial explaining the structure of the [configuration file](http://stash.corp.alleninstitute.org/projects/MIND/repos/bmtk/browse/docs/examples/simulator/bionet/README_config.md). 









