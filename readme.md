This is a setup guide to run RL models on NYU HPC Prince cluster.

### Table of Contents:
* **[Logging In](#Logging-on-Prince)**
* **[Setting up environemnt](#Setting-up-PyTorch-and-Open-AI-gym-environement)**
* **[Submitting Job](#Submitting-Job-on-Prince)**
* **[Transferring files:](#Transferring-files-to-Prince)**

# Logging on Prince
From your terminal:
```
ssh user_id@gw.hpc.nyu.edu
ssh prince.hpc.nyu.edu
```

Better way: Is to set up one off ssh tunnel, follow the following steps on your local computer:
```
mkdir ~/.ssh
chmod 700 ~/.ssh
```

Add the following lines to .ssh/config:

```
Host hpcgwtunnel
   HostName gw.hpc.nyu.edu
   ForwardX11 no
   LocalForward 8025 dumbo.hpc.nyu.edu:22
   LocalForward 8026 prince.hpc.nyu.edu:22
   User NetID 
# next we create an alias for incoming packets on the port. The
# alias corresponds to where the tunnel forwards these packets
Host dumbo
  HostName localhost
  Port 8025
  ForwardX11 yes
  User NetID

Host prince
  HostName localhost
  Port 8026
  ForwardX11 yes
  User NetID
```

Now for login just do:

```
ssh hpcgwtunnel
ssh -Y prince
```

# Setting up PyTorch and Open AI gym environement

There is no module of Open AI gym on the cluster so to run most of RL models we need to install on our own. We can't install them directly on the cluster environment so we have to create a virtual environemt where our code will run.

### Creating Virtual Enviroment:

```
cd \scratch\$user_id$\
nano create_env_sh
```

Copy the following code in the above created file
```
# Remove any preloaded modules
module purge
# activate existed python3 module to get virtualenv
module load python3/intel/3.7.3
module load cuda/10.0.130

# create virtual environment with python3
virtualenv --system-site-packages mlenv

# activate virtual environment
source $HOME/mlenv/bin/activate


# install torch and gym
pip install torch torchvision
pip install gym
pip install gym[atari]
```

In the above code firstly we load already installed modules on the cluster such as `python3` and `cuda`. You can also try loading `anaconda3` but it did't work for me. There is no need for `cudann` module as it will be loaded automatically with `python3`.

To load any other module use can use `module avail` to see available list of modules, or search using `module keyword $keyword$`. After activating environment install all the modules using `pip` you need for project. 

Now create the environment by running:
```
source create_env.sh
```

You only need to create environment once and then for starting a environment everytime you just need to load all modules and activate environment `source $HOME/mlenv/bin/activate`.

To make things easy create a script for that:
```
nano run_env.sh
```
```
module purge
module load python3/intel/3.7.3
module load cuda/10.0.130
source $HOME/mlenv/bin/activate
```

To start environment: `source run_env.sh`.

# Submitting Job on Prince

To submit a job you need to create a batch script, like:

```
#!/bin/bash
#SBATCH --gres=gpu:1
#SBATCH --mem=16000
#SBATCH --job-name="ccm_project"
#SBATCH --time=12:00:00
#SBATCH --mail-type=END
#SBATCH --mail-user=$email_id$
#SBATCH --output=slurm_%j.out
module purge
module load python3/intel/3.7.3
module load cuda/10.0.130

# Replace with your NetID
NETID=###


# activate virtual environment
source /home/user_id/mlenv/bin/activate
export PYTHONPATH=/scratch/user)id/RL/Project/Updated/:$PYTHONPATH
cd /directory of the file you want to run
python3 ./game.py
```

Make sure you change based on your requirements, such as load all modules you need and if you want to run inside a virtual environment then activate virtual environment. `cd` into the directory and the run the program. `PYTHONPATH` is neceesary if you are runnign a modularized code otherwise it will not to be able to `import` code from other files.

Now instead of loading and activating files manually you can also use the `run_env.sh` script, you created above.

```
#!/bin/bash
#SBATCH --gres=gpu:1
#SBATCH --mem=16000
#SBATCH --job-name="ccm_project"
#SBATCH --time=12:00:00
#SBATCH --mail-type=END
#SBATCH --mail-user=$email_id$
#SBATCH --output=slurm_%j.out


# Replace with your NetID
NETID=###

# activate virtual environment
source /home/user_id/run_env.sh
export PYTHONPATH=/scratch/user)id/RL/Project/Updated/:$PYTHONPATH
cd /directory of the file you want to run
python3 ./game.py
```

Running A job
```
sbatch $job-script$
```

To see already running jobs:

```
squeue -u $user_id$
```

To cancel jobs:

```
scancel $job_id$
```

Output of the file (such as logs and standard output and erro) will be created in the directory from where you subimitted the job

# Transferring files to Prince

### Using SCP (Recommended):

First login and start tunnel

```
scp source destination
```

Either source or destination can be on another (remote) host, by prefixing the path with "hostname:"

From local &rarr; remote
```
scp local_file prince:/scratch/net_id/
```

From remote &rarr; local
```
scp prince:/scratch/net_id/ local_file
```

### Using FUGU:
Download link mentioned on NYU cluster is not working so I have attached the correct version in this repository.

After installing:

**Step 1:**
Start Fugu.  Select SSH &rarr new SSH tunnel
* Create tunnel to: prince (or dumbo)
* Service or port: 22
* Local port: 8026 (8025 to dumbo)
* Tunnel host: gw.hpc.nyu.edu
* Username: <NetID>

**Step 2:**
In SFTP window
* Connect to: localhost
* Username: <NetID>
* Port: 8026 (or 8025)

**Step 3:**
* Click connect and enter <NetID> password.
* Drag and drop files to copy/paste to and from cluster.