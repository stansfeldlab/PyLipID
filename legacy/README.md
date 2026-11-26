## Introduction

**pylipid.py** is a toolkit for calculating lipid interactions with membrane proteins.

It calculates:

- lipid interactions with proteins in terms of duration, residence time, occupancy, number of lipids surrounding given residues, and koff;
- lipid binding sites through interaction networks;
- binding-site kinetics, such as lipid residence time and koff, for each binding site;
- lipid-binding-site surface areas using the Shrake-Rupley algorithm (Shrake and Rupley, J. Mol. Biol. 79 (1973) 351-371);
- probability density functions for bound lipids and representative binding poses for each binding site based on the calculated PDFs.

It plots:

- lipid interactions (in terms of duration, residence time, occupancy, and number of surrounding lipids) with the protein as a function of protein residue index;
- calculated lipid koff values for each protein residue;
- calculated lipid koff values for each binding site;
- surface area for each binding site.

It generates:

- protein coordinates in PDB format in which data such as residence time, koff, duration, and occupancy are recorded in the B-factor column;
- representative binding poses for each binding site based on scoring functions that use probability density functions for the bound lipids.

It can also map the calculated binding sites onto a PDB structure provided by the user via `-pdb`. When the `-pdb` flag is provided, `pylipid.py` writes a Python script named `show_binding_site_info.py`. This script opens a PyMOL session in which residues belonging to the same binding site are shown as spheres whose sizes correspond to their calculated residence times.

For a definition of residence time, see:

- García, A. E. and Stiller, L. "Computation of the mean residence time of water in the hydration shells of biomolecules." Journal of Computational Chemistry (1993).
- Duncan, A. L., Corey, R. A., and Sansom, M. S. P. "Defining how multiple lipid species interact with inward rectifier potassium (Kir2) channels." Proceedings of the National Academy of Sciences (2020).

To alleviate the "cage-rattling" phenomenon of bead dynamics in coarse-grained simulations, PyLipID uses a dual-cutoff scheme. The scheme defines the start of a continuous interaction between a lipid molecule and a given object as the point when any atom or bead of the lipid molecule moves within the smaller cutoff. It defines the end of the interaction as the point when all atoms or beads of the lipid molecule move outside the larger cutoff. This scheme can also be applied to atomistic simulations. The recommended dual cutoffs are **0.55 1.0** nm for coarse-grained simulations and **0.35 0.55** nm for atomistic simulations. Users should test these values with their systems. The same value can be used for both cutoffs to obtain a single-cutoff scheme.

## Installation

`pylipid.py` requires the following packages:

- Python
- mdtraj
- numpy
- pandas
- matplotlib
- seaborn
- networkx
- scipy
- python-louvain
- logomaker
- statsmodels
- scikit-learn
- tqdm

To avoid modifying global Python settings, we recommend using [Conda](https://www.anaconda.com/distribution/) to create an isolated environment named PyLipID.

Create the PyLipID environment using the provided `env.yml`, if it is available for your platform:

```bash
conda env create -f env_{OS_PLATFORM}.yml
```

Activate the environment before using the script:

```bash
conda activate PyLipID
```

To return to the default environment:

```bash
conda deactivate
```

To remove the environment:

```bash
conda env remove --name PyLipID
```

## Examples

Information about the flags accepted by **pylipid.py** is available with:

```bash
python pylipid.py -h
```

A standard lipid-interaction analysis suitable for most cases is:

```bash
conda activate PyLipID
python pylipid.py -f ./run_1/md.xtc ./run_2/md.xtc -c ./run_1/protein_lipids.gro ./run_2/protein_lipids.gro \
  -cutoffs 0.55 1.0 -lipids POPC CHOL POP2 -nprot 1 -save_dataset
```

Coarse-grained force fields often make phospholipid tails too flexible because of smoothened energy potentials. This can lead to poor characterization of binding sites. When tail behavior is not the main focus, it is better to focus on headgroup binding. Use the `-lipid_atoms` flag to specify which lipid atoms or beads are used for the calculation. The following example calculates PIP2 binding in MARTINI 2 (named POP2 in this force field) using only the headgroup beads:

```bash
python pylipid.py -f ./run_1/md.xtc ./run_2/md.xtc -c ./run_1/protein_lipids.gro ./run_2/protein_lipids.gro \
  -cutoffs 0.55 1.0 -lipids POP2 -lipid_atoms C1 C2 C3 C4 PO4 P1 P2 -nprot 1 -save_dataset
```

**pylipid.py** uses graph theory and community analysis to calculate lipid binding sites. Binding-site information is written to `BindingSites_info_{LIPID}.txt` in the `Binding_Sites_{LIPID}` directory. For each binding site, `pylipid.py` can write out highly ranked binding poses sampled from the simulations. It scores bound lipid poses for each binding site using a function based on the probability density of bound lipids at that site. The written coordinates include the bound lipid and the receptor conformation to which it was bound. By default, `pylipid.py` writes the top five ranked lipid poses for each binding site in Gromacs `gro` format. Use `-save_pose_format` to choose any coordinate format supported by [MDTraj](http://mdtraj.org). For phospholipids, it is recommended to give higher weights to lipid headgroups in the scoring function. Use `-score_weights` to change the weights. The `-n_binding_poses` flag specifies how many lipid poses are generated for each binding site.

The following example generates 10 highly ranked poses for each binding site, saves them in `gro` format, and gives higher weights to the PIP2 headgroup beads in the MARTINI force field:

```bash
python pylipid.py -f ./run_1/md.xtc ./run_2/md.xtc -c ./run_1/protein_lipids.gro ./run_2/protein_lipids.gro \
  -cutoffs 0.55 1.0 -lipids POP2 -nprot 1 -save_pose_format gro \
  -score_weights PO4:10 P1:10 P2:10 C1:10 C2:10 C3:10 -n_binding_poses 10
```

The calculation of lipid probability density uses the [KDEMultivariate](https://www.statsmodels.org/stable/generated/statsmodels.nonparametric.kernel_density.KDEMultivariate.html) function in statsmodels. It can take up to an hour for atomistic simulations or long coarse-grained simulations with large amounts of collected binding data, either because of higher granularity or a larger number of binding events. To speed up the calculation, reduce the amount of data using the `-stride` flag to analyze every X-th trajectory frame. If bound lipid coordinates are not required, use `-n_binding_poses 0` to disable binding-pose generation.

The script can also view the calculated binding sites in PyMOL by generating a Python script. This process is triggered by providing an atomistic protein structure, preferably in PDB format, to the `-pdb` flag. For coarse-grained simulations, provide the atomistic protein structure used before coarse-graining or an atomistic structure converted from a coarse-grained model. Ensure that the provided protein coordinates are consistent with the simulation configuration, including residue indices and ordering.

An example using the `-pdb` flag is:

```bash
python pylipid.py -f ./run_1/md.xtc ./run_2/md.xtc -c ./run_1/protein_lipids.gro ./run_2/protein_lipids.gro \
  -cutoffs 0.55 1.0 -lipids POPC CHOL POP2 -nprot 1 -save_dataset -pdb XXXX.pdb
```

Replace `XXXX.pdb` with the PDB file of your choice. Running the generated Python script with `python show_binding_site_info.py` opens a PyMOL session displaying the binding-site information.

**pylipid.py** allows users to specify several regions for calculation via the `-resi_list` flag. Supported syntax includes: (1) use `-` to indicate a range of protein residue indices, with both endpoints included; or (2) specify individual residue indices separated by spaces:

```bash
python pylipid.py -f ./run_1/md.xtc ./run_2/md.xtc -c ./run_1/protein_lipids.gro ./run_2/protein_lipids.gro \
  -cutoffs 0.55 1.0 -lipids POPC CHOL POP2 -nprot 1 \
  -resi_list 5 7 8 10-30 50-70 100-130 -save_dataset
```

**pylipid.py** calculates the surface area of each binding site. By default, the script uses the atom radii defined by MDTraj (https://github.com/mdtraj/mdtraj/blob/master/mdtraj/geometry/sasa.py#L56). The script also defines the radii of MARTINI coarse-grained beads: BB is 0.26 nm and SC1/2/3 are 0.23 nm. To change or define atom or bead radii, use `-radii` and specify the radii in nm.

For example, to change the MARTINI coarse-grained bead radii BB to 0.28 nm and SC1 to 0.22 nm:

```bash
python pylipid.py -f ./run_1/md.xtc ./run_2/md.xtc -c ./run_1/protein_lipids.gro ./run_2/protein_lipids.gro \
  -cutoffs 0.55 1.0 -lipids POPC CHOL POP2 -nprot 1 -radii BB:0.28 SC1:0.22
```

## Developers

- Wanling Song
- Anna Duncan
- Robin Corey
- Bertie Ansell

## Thanks for reading to the end

Writing scripts is about fixing one bug after another.

Be brave, be real, and keep going!

![GitHub Logo](https://github.com/wlsong/wlsong/blob/master/resources/dino.gif)
