# Computing intermediate data

These scripts are designed to be run from the `mitochondria` directory above this one.
Many of them take some time, but can be parallelized using command line arguments or
by simply splitting the cells/organelles processed into jobs and merging the results.

## Computing basic branch statistics
Here, a branch refers to a connected component of a skeleton that possesses
the same compartment label. This means that branches can also refer to the
soma.

This script computes a data frame with an ID, volume, and surface area
measurement for each branch. The branch ID is the minimum skeleton node index
that is contained within the branch. This can be parallelized by splitting over
the processed (`clean_and_complete_soma_ids_v185.csv` below).

NOTE: this requires downloading the meshes (through downloadNeuronMeshes.sh)
```
$ python scripts/computebasicbranchstats.py \
      data/clean_and_complete_soma_ids_v185.csv \
      data/basicbranchstats.csv
```


## Associating mitochondria to skeleton nodes
More precisely, finding the set of skeleton nodes that is the closest to
some mesh vertex of each mitochondrion. Labels each such mitochondrion
with the set of skeleton node indices that is paired with it in this way.
This can be parallelized by splitting the mitochondria dataframe
(`pni_mito_cellswskel_v185.csv`) by cell or naively splititng the rows.


NOTE: this requires downloading the mitochondria meshes but these are not available yet
```
$ python scripts/mitotoskel.py \
      data/pni_mito_cellswskel_v185.csv \
      data/temp/assoc_
$ python scripts/assemblemitotoskel.py \
      data/pni_mito_cellswskel_v185.csv \
      data/pycids_v185.csv data/mito_to_skel_v185.h5
```


## Computing MCI and assigning mitochondria to compartments
This adds a surface area measurement, mitochondrial complexity index,
and a compartment assignment to each row of the mitochondria stats
data frame. This can be parallelized by splitting the rows of the
mitochondria dataframe (`pni_mito_cellswskel_v185.csv`) naively.

NOTE: this requires downloading the mitochondria meshes (through downloadmitomeshes.py)
```
$ python scripts/computeextramitostats.py \
      data/pni_mito_cellswskel_v185.csv \
      data/mitotoskel.h5 \
      data/pni_mito_cellswskel_v185_fullstats.csv
```


## Computing density statistics
For each compartment branch, we count the number of synapses associated
with each branch and compute its total mitochondrial volume. We then
compute synapse and mitochondrial density metrics, and write out
two dataframes as a result - one describing a branch per row, and
another describing a cell per row, where each branch of a compartment
is summed within each cell. This can be parallelized by splitting over
cells (`clean_and_complete_soma_ids_v185.csv` below).
```
$ python scripts/computedensitystats.py \
      data/pni_mito_cellswskel_v185_fullstats.csv \
      data/clean_and_complete_soma_ids_v185.csv \
      data/neuron_received_synapses_v185.csv \
      data/basicbranchstats.csv \
      data/pni_mito_cell_overlaps_v185.csv \
      data/pni_nucleus_segments.csv \
      data/clean_and_complete_nucleus_lookup.csv \
      data/fullbranchstats.csv \
      data/cellwisestats.csv
```
