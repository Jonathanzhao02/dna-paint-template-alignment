# Software for DNA origami cryptography in 2D and 3D space.

This software implements the 2D unsupervised clustering and template alignment method employed in the paper
"High-speed 3D DNA-PAINT and unsupervised clustering for unlocking 3D DNA origami cryptography". The
prior knowledge of the origami template allows fast and accurate analysis of individual particles.

For full results, see [this link](https://www.dropbox.com/sh/48cj5v1qc5ny246/AABSLdFF__FomtCsfQaXNi2Xa?dl=0).

Notes:

* The NSF patterns were run using the "rough" alignment method
* The ASU patterns were run using the "differential_evolution" alignment method

# Setup Instructions

For environment management, we use Miniconda or Anaconda.

To install the conda environment, run `conda env create --file <path to dna_paint.yml>`.
This will create a new conda environment called "dna_paint" with all the required dependencies.

If the yml installation does not work, the following commands can be run instead:
1. `conda create -n dna_paint python==3.7.1`
2. `conda install numpy scikit-learn scipy matplotlib h5py tqdm numba`

This setup should take 30-60 minutes to run.

The software follows this pipeline:
* Processed Picasso data is converted into a compatible (+ labeled) format using the code in `preprocessing`
* Individual readouts are conducted using `process_particles.py`
* Individual readout analysis is conducted using `process_reads.py`
* Superparticle analysis is conducted using `process_superparticles.py`

Some scripts depend on results generated by [smlm_classification2d](https://github.com/Jonathanzhao02/smlm_classification2d).

# Demo

To run the code with the provided NSF dataset used by Figure 3, follow the below instructions:

1. `python3 process_particles.py dataset -t nsf -c kmeans -a rough`
2. `python3 process_reads.py dataset -t nsf`

The first step will perform K-Means clustering on the particle, then align the NSF template for
readout. The second step will then print information and generate figures for the readouts.

These steps should take 30-60 minutes to run.

# Picasso Data Formatting

The data should be processed by Picasso into the .hdf5 format.
Picks should already have been created.

In order to format the data for use in clustering and readout analysis, utilize the scripts in the `preprocessing` folder.

## `convert.py`
This script converts the .hdf5 into a .mat format.

All other scripts utilize the data in a .mat format, so this step must be done for all .hdf5 files.

To use this script, run: `python3 picasso_conversion/convert.py <path to .hdf5 file> [-o <output folder name>] [-t <label name>] [-p <# of picks>]`. Arguments in square brackets are optional, arguments in angled brackets are required.

The arguments work as follows:
* -o / --output: Name of the output folder to save the .mat to. This folder is saved into a folder called `data` in the parent directory of picasso_conversion.
* -t / --tag: Name of the label to associate with each pick in the data. For example, for NSF, the tag is 'N' for N, 'S' for S, and 'F' for F. This enables processing the readouts with respect to their ideal readouts.
* -p / --picks: The number of picks to select from the file. <= 0 means select all picks.

The output is a folder containing a file named `subParticles.mat`.

## `merge.py`
This script combines multiple .mat files into a single file.

Only a single .mat file is accepted by the other scripts, so this script allows the mixture of multiple classes into a single file.

To use this script, run: `python3 picasso_conversion/merge.py <path to folder of .mat files> <output folder name> [-p <# of picks>]`. Arguments in square brackets are optional, arguments in angled brackets are required.

The arguments work as follows:
* -p / --picks: The number of picks to select from each file. <= 0 means select all picks.

The output is a folder containing a file named `subParticles.mat`.

# Individual Particle Readout
## Valid methods/alignments/templates

For the template, alignment, and cluster CLI arguments below, these are the following values:

* VALID_CLUSTER_METHODS = ['kmeans', 'dbscan', 'meanshift', 'mle']
* VALID_ALIGNMENT_METHODS = ['differential_evolution', 'shgo', 'dual_annealing', 'rough', 'finetune']
* VALID_TEMPLATE_NAMES = ['nsf', 'asu_2', 'asu_3']

## Description

The below scripts perform particle readout and analysis over individual picks.

Within both files are various boolean parameters at the top of the file which control interactive visualizations. Turning them on/off is mainly a matter of speed.

## `process_particles.py`
This script performs and saves readouts over each individual pick.
The dataset folder should contain "subParticles.mat" generated by preprocessing.

To use this script, run: `python3 process_particles.py <path to dataset folder> [-o <output file name>] [-t <template name>] [-c <clustering method>] [-a <alignment method>] [-j <path to config file>]`. Arguments in square brackets are optional, arguments in angled brackets are required.

The arguments work as follows:
* -o / --output: The name of the output .mat file which contains all readouts.
* -t / --template: The name of the template to use. Templates are in the `templates` folder.
* -c / --cluster: The name of the clustering method to use, or MLE.
* -a / --alignment: The name of the alignment optimization method to use. Unused for MLE.
* -j / --config: The path to the config file for the methods.

The output is a single file containing the readout results, including centroid positions, cost function values, aligned grid positions, binary readout, correctness, etc. The name defaults to "final.mat".

## `process_particles_spectral.py`
This script performs and saves readouts over each individual pick.
It assumes spectral clustering has already been performed and processes spectral clusters separately.
The dataset folder should contain "subParticles.mat" generated by preprocessing and "clusters.mat" generated by spectral clustering in `smlm_classification2d`.

To use this script, run: `python3 process_particles_spectral.py <path to dataset folder> [-o <output file name>] [-t <template name>] [-c <clustering method>] [-a <alignment method>] [-j <path to config file>]`. Arguments in square brackets are optional, arguments in angled brackets are required.

The arguments work as follows:
* -o / --output: The name of the output .mat file which contains all readouts.
* -t / --template: The name of the template to use. Templates are in the `templates` folder.
* -c / --cluster: The name of the clustering method to use, or MLE.
* -a / --alignment: The name of the alignment optimization method to use. Unused for MLE.
* -j / --config: The path to the config file for the methods.

The output is a single file containing the readout results, including centroid positions, cost function values, aligned grid positions, binary readout, correctness, etc. The name defaults to "final.mat".

## `process_particles_kmeans.py`
This script performs and saves K-means clustering results over each individual pick.
The dataset folder should contain "subParticles.mat" generated by preprocessing.

To use this script, run: `python3 process_particles_kmeans.py <path to dataset folder> [-o <output file name>] [-t <template name>] [-j <path to config file>]`. Arguments in square brackets are optional, arguments in angled brackets are required.

The arguments work as follows:
* -o / --output: The name of the output .mat file which contains all readouts.
* -t / --template: The name of the template to use. Templates are in the `templates` folder.
* -j / --config: The path to the config file for the methods.

The output is a single file containing the cluster results for increasing numbers of centroids, as well as the inertia curve. The name defaults to "clusters.mat".

## `process_clusters_kmeans.py`
This script performs and saves readouts over each individual pick, assuming K-means clustering has already been performed.
The K-means clustering is performed using `process_particles_kmeans.py`.
The dataset folder should contain "subParticles.mat" generated by preprocessing and "clusters.mat" generated by `process_particles_kmeans.py`.

To use this script, run: `python3 process_clusters_kmeans.py <path to dataset folder> [-o <output file name>] [-t <template name>] [-a <alignment method>] [-j <path to config file>] [-i <path to clusters file>] [-of <path to output folder>]`. Arguments in square brackets are optional, arguments in angled brackets are required.

The arguments work as follows:
* -o / --output: The name of the output .mat file which contains all readouts.
* -t / --template: The name of the template to use. Templates are in the `templates` folder.
* -a / --alignment: The name of the alignment optimization method to use. Unused for MLE.
* -j / --config: The path to the config file for the methods.
* -i / --infile: The name of the input .mat file which contains all clusters. Default "clusters.mat"
* -of / --outfolder: The name of the output folder to write CSVs.

The output is a single file containing the readout results, including centroid positions, cost function values, aligned grid positions, binary readout, correctness, etc. The name defaults to "final.mat".

This script additionally outputs "clusters.csv" and "filt_clusters.csv", which contain information about the K-means clusters.

## `process_reads.py`
This script analyzes the readout results from `process_particles.py` or any other script that generates "final.mat".

To use this script, run: `python3 process_reads.py <path to dataset folder> [-i <readout file>] [-o <output file name>] [-t <template name>]`. Arguments in square brackets are optional, arguments in angled brackets are required.

The arguments work as follows:
* -i / --infile: The name of the file containing individual readouts located within the dataset folder.
* -o / --output: The name of the file to save analysis results to. These consist currently only of misclassifications.
* -t / --template: The name of the template to use. Templates are in the `templates` folder.

The output is a folder containing the misclassifications visualizations. Various figures and standard output consisting of analysis results are also displayed.

# Superparticle Readout
## Valid methods/alignments/templates

For the template, alignment, and cluster CLI arguments below, these are the following values:

* VALID_CLUSTER_METHODS = ['kmeans', 'dbscan', 'meanshift', 'mle']
* VALID_ALIGNMENT_METHODS = ['differential_evolution', 'shgo', 'dual_annealing', 'rough']
* VALID_TEMPLATE_NAMES = ['nsf', 'asu_2', 'asu_3']

## Description

The below script performs particle readout over the superparticles of each cluster.
It assumes spectral clustering has already been performed.

## `process_superparticles.py`
This script performs readout over the superparticle results.
Within this file are various boolean parameters at the top of the file, which control interactive visualizations. Turning them on/off is mainly a matter of convenience.

To use this script, run: `python3 process_superparticles.py <path to dataset folder> [-t <template name>] [-c <clustering method>] [-a <alignment method>] [-j <path to config file>]`

The arguments work as follows:
* -t / --template: The name of the template to use. Templates are in the `templates` folder.
* -c / --cluster: The name of the clustering method to use, or MLE.
* -a / --alignment: The name of the alignment optimization method to use. Unused for MLE.
* -j / --config: The path to the config file for the methods.

Various figures and standard output consisting of analysis results are displayed.
