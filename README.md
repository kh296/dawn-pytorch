# Installing PyTorch on Dawn

## 1. Introduction

This is guidance for installing [PyTorch](https://pytorch.org/docs/stable/)
in a [conda](https://docs.conda.io/en/latest/) environment on
the [Dawn supercomputer](https://www.hpc.cam.ac.uk/d-w-n).  Dawn is
hosted at the University of Cambridge, and is part
of the [AI Resource Research (AIRR)](https://www.gov.uk/government/publications/ai-research-resource/airr-advanced-supercomputers-for-the-uk).  It was
initially installed with 256 nodes, in the form of [Dell PowerEdge XE9640](https://www.delltechnologies.com/asset/en-us/products/servers/technical-support/poweredge-xe9640-spec-sheet.pdf) servers.  Each node consisted of: 2 CPUs ([Intel Xeon Platinum 8468](https://www.intel.com/content/www/us/en/products/sku/231735/intel-xeon-platinum-8468-processor-105m-cache-2-10-ghz/specifications.html)), each with 48 cores and 512 GiB RAM; 4 GPUs ([Intel Data Centre GPU Max 1550](https://www.intel.com/content/www/us/en/products/sku/232873/intel-data-center-gpu-max-1550/specifications.html)),
each with two stacks (or tiles), 1024 compute units, and 128 GiB RAM.

The material collected here is licensed under the
[Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0).

## 2. Installation

In case you don't already have your own `conda` installation, you can find
guidance for installing `conda` on Dawn at:
- [https://github.com/kh296/dawn-conda](https://github.com/kh296/dawn-conda)

Installation of PyTorch may be performed
[via a Slurm job](#21-installation-via-a-slurm-job) or
[from the command line](#22-installation-from-the-command-line).  As
installation takes 30-60 minutes, the former is recommended

### 2.1 Installation via a Slurm job

On a Dawn login node or compute node, clone this repository,
and move to the `scripts` directory:
```
git clone https://github.com/kh296/dawn-pytorch
cd dawn-pytorch/scripts
```

Submit a Slurm job to run the installation script:
```
# Substitute for <project_account> a valid project account.
# Set CONDA_INSTALL to the path of your conda installation.
sbatch --account=<project_account> --export=CONDA_INSTALL="~/miniforge3" ./pytorch_install.sh
```

Once it starts running, the script should take 30-60 minutes to
complete.  The job output is written to `pytorch_install.log`.  If the
installation is successful, the last line of the output is the command
to set up the environment for using PyTorch.  This command references the
setup file `../envs/pytorch-setup.sh`, created during installation.

### 2.2 Installation from the command line

On a Dawn compute node, clone this repository, and move to
the `scripts` directory:
```
git clone https://github.com/kh296/dawn-pytorch
cd dawn-pytorch/scripts
```

Run the installation script:
```
# Set CONDA_INSTALL to the path of your conda installation.
CONDA_INSTALL="~/miniforge3" ./pytorch_install.sh |& tee pytorch_install.log
```

Output is written both to terminal and to the file `pytorch_install.log`.
If the installation is successful, the last line of the output is the command
to set up the environment for using PyTorch.  This command references the
setup file `../envs/pytorch-setup.sh`, created during installation.

## 3. Further information

Installation of `PyTorch` on Dawn is based on the documentation for
[Getting Started on Intel GPU](https://pytorch.org/docs/stable/notes/get_start_xpu.html).

Support for Intel GPUs in versions of PyTorch prior to 0.2.8 required
import of two external packages:
- [intel_extension_for_pytorch](https://github.com/intel/intel-extension-for-pytorch);
- [oneccl_bindings_for_pytorch](https://github.com/intel/torch-ccl).

In these versions, the backend for distributed processing needed to be
specified as `"ccl"`.  From PyTorch 0.2.8 onwards, the external packages
shouldn't be imported, and the backend for distributed processing needs
to be specified as `"xccl"`.  For more information about the updated backend,
see the last item of the [PyTorch 2.8 Release Blog](https://pytorch.org/blog/pytorch-2-8/).

The installation script [scripts/pytorch_install.sh](scripts/pytorch_install.sh)
installs the latest stable versions of `torch`, `torchvision`, `torchaudio`,
along with their dependencies.  If you want to install specific versions, you
can edit the script to indicate this.  If you want to install additional
packages, the suggested approach is to set up the `conda` environment for
using PyTorch, and then install the additional packages with `pip` or `conda`.
For example, to add `pandas`, starting from the `scripts` directory, use:
 ```
source ../envs/pytorch-setup.sh
pip install pandas
```

The installation script provides several options, for example allowing
installation to a `conda` environment with a name different from the default
(`pytorch`).  For
more information, from the `scripts` directory run:
```
./pytorch_install.sh -h
```

The setup script `envs/setup-pytorch-setup.sh`, created during installation,
sets values for a number of environment variables relevant to using
PyTorch with Intel GPUs, including for distributed processing.  These
variables are documented in the setup script.

The setup script performs all environment setup needed to use PyTorch,
including making available compatible versions of oneAPI libraries:
```
# Perform environment setup for using PyTorch.
# Substitue for <setup script> the path to the setup script.
source <setup script>
```
The script generally shouldn't be combined with system scripts and
modules for environment setup.  In particular, none of the system modules
for `conda` setup or for oneAPI setup should be loaded.
