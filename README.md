# PeachCI - Scalable Continuous Integration of Generation-Based Protocol Fuzzing

# Folder Structure
```
PeachCI
├── subjects: contains folders for different protocol implementations
│   └── FTP
│       └── lightftp
│           └── Dockerfile: for building the Docker image specific to the target server
│           └── run.sh: main script to run the experiment inside a Docker container
│           └── other necessary files (e.g.scripts)
├── fuzzers: contains folders for different generation-based fuzzing tool
│   └── Peach
│       └── Dockerfile: for building the Docker image specific to the fuzzing tool
│       └── run.sh: main script to fuzzing inside a Docker container
│       └── other necessary files (e.g.scripts)
├── pits: folders for different Pit files -- The naming of the pit file must match the protocol implementation.
│   └── lightftp.xml
│   └── dnsmasq.xml
└── scripts: contains all scripts for running experiments and analyzing results
    ├── execution
    │   └── exec_common.sh: main script to run fuzzing experiments
    └── analysis: How to use Prometheus and Grafana
└── README.md: this file
```

# Tutorial - Fuzzing LightFTP server with Peach
## Step-0. Set up environmental variables
```
git clone https://github.com/csu-wingmate/PeachCI.git
cd PeachCI
export CIPATH=$(pwd)
export PATH=$PATH:$CIPATH/scripts/execution:$CIPATH/scripts/analysis
./setup.sh
```

## Step-1. Build a Fuzzer Docker image and a Protocol Docker image
```bash
cd $CIPATH
cd fuzzers/Peach
docker build . -t peach
```
```bash
cd $CIPATH
cd subjects/FTP/lightftp
docker build . -t lightftp
```

## Step-2. Run fuzzing
- ***1st argument (PROTOCOL)*** : name of the protocol Implementation(e.g., lightftp)
- ***2nd argument (RUNS)***     : number of runs, one isolated Docker container is spawned for each run
- ***3rd argument (SAVETO)***   : path to a folder keeping the results
- ***4th argument (FUZZER)***   : fuzzer name (e.g., peach) 
- ***5th argument (TIMEOUT)***  : time for fuzzing in seconds


The following commands run 4 instances of Peach to simultaneously fuzz LightFTP for 5 minutes.

```bash
cd $CIPATH
mkdir results-lightftp

exec_common.sh lightftp 4 results-lightftp peach 300
```

A successful script execution will produce output similar to this:
```
Peach: Fuzzing in progress ...
Peach: Waiting for the following containers to stop:  f2da4b72b002 b7421386b288 cebbbc741f93 5c54104ddb86
Peach: I am done!
```

## Step-3. Collect the results
All results are stored in tar files within the folder created in Step-2 (results-lightftp). This includes directories named similarly to peach-1-branch and peach-1-logs, where peach-1-branch contains the collected branch coverage data and peach-1-logs contains the log files from the Peach testing process, including the number of test runs and potential bug reports.

## Step-4. Analyze the results
The data collected in Step 3 on branch coverage counts, potential vulnerabilities, etc. can be used for plotting. We used Prometheus to collect the data and Grafana for visualising data such as code coverage over time.
```bash
cd $CIPATH/results-lightftp

profuzzbench_plot.py -i <input_data> -o <output_plot_file>
```
Replace <input_data> with the path to your coverage data and <output_plot_file> with the desired filename for your plot.
This is an example of the generated code coverage report, potential vulnerabilities, and test execution speed.
![Grafana-Dashboard](https://github.com/csu-wingmate/profuzzpeach/blob/main/figures/Grafana-Dashboard.png)

# Utility Scripts
PeachCI includes scripts for building and running all fuzzers on all targets with pre-configured parameters. To build all targets for all fuzzers, run the script build_all.sh. To execute the fuzzers, use the script exec_all.sh.

# FAQs
## 1. How do I extend PeachCI?
To add a new protocol and/or a new target server for a supported protocol, follow the folder structure outlined above and complete the following steps, using LightFTP as an example:

### Step-1. Create a new folder for the protocol/target server
The folder for LightFTP server is located at subjects/FTP/LightFTP.

### Step-2. Write a Dockerfile and prepare subject-specific scripts/files
Refer to the existing folder structure for LightFTP
```
subjects/FTP/LightFTP
├── Dockerfile (required): based on this, a target-specific Docker image is built (See Step-1 in the tutorial)
├── run.sh (required): main script to run experiment inside a container
├── cov_script.sh (required): script to do code coverage analysis
├── clean.sh (optional): script to clean server states before fuzzing to improve the stability
├── fuzzing.patch (optional): code changes needed to improve fuzzing results (e.g., remove randomness)
├── gcov.patch (required): code changes needed to support code coverage analysis (e.g., enable gcov, add a signal handler)
├── ftp.dict (optional): a dictionary containing protocol-specific tokens/keywords to support fuzzing
└── in-ftp (required): a seed corpus capturing sequences of client requests sent to the server under test.
│   │       To prepare these seeds, please follow the AFLNet tutorial at https://github.com/aflnet/aflnet.
│   │       Please use ".raw" extension for all seed inputs.
│   │
│   └── ftp_requests_full_anonymous.raw
│   └── ftp_requests_full_normal.raw
└── README.md (optional): a target-specific README containing commands to run experiments
```
All the required files (i.e., Dockerfile, run.sh) follow some templates so that one can easily follow them to prepare files for a new target.
