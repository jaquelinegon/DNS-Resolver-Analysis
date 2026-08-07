# DNS Resolver Blocking Test

This artifact accompanies the paper related to the evaluation of public DNS resolvers regarding their ability to block malicious domains. The `test_resolvers.sh` script automates the collection of malicious domain lists, queries each domain against the different DNS resolvers using the `dig` command, and records whether the domain was blocked or resolved normally, generating a CSV report with the comparative results.

# Structure

This repository consists of a single self-contained script:

- `test_resolvers.sh`: main script, responsible for downloading the malicious domain lists, querying the DNS resolvers, and generating the results report.

At the end of execution, the script generates two files in the execution folder:

- `dnsblock_hosts_<data>.txt`: consolidated list of tested domains, along with their respective source.
- `dnsblock_result_<data>.csv`: test results, in the format `domínio;fonte;resolvedor_1;resolvedor_2;...`, where each resolver column contains the returned IP, with the cell left empty when the domain is blocked.

# Badges Considered

The authors consider the following badges in the evaluation process:

- Available Artifacts
- Functional Artifacts
- Reproducible Experiments

 # Dependencies

Below are the dependencies required to run `test_resolvers.sh`.

## Hardware

- Linux operating system, tested on Ubuntu-based distributions.
- RAM: minimum 8 GB for a VM.
- Active internet connection, with no outbound blocks on port 53, used for DNS resolution, and on ports 443 and 80, used for downloading the lists.

The total execution time is mainly limited by the network latency to the queried DNS resolvers. Since the script performs one DNS query per domain for each of the five resolvers, execution time can be high even on machines with hardware resources above the minimum requirements.

## Required commands
- `dig`, from the `dnsutils` package: performs a DNS query against a specific resolver.
- `wget`: downloads the malicious domain lists.
- `parallel`, from the GNU Parallel package: runs the DNS queries in parallel.
- `ping`, from the iputils-ping package: measures the latency to each resolver.
- `awk` and `sort`, from the `gawk` and `coreutils` packages: cleaning and deduplication of the domain lists.

## Dependency installation script for Debian and Ubuntu

```
sudo apt-get update
sudo apt-get install -y dnsutils wget parallel iputils-ping gawk coreutils
```

## External resources used

The script depends on the following third-party resources, all public and free, requiring no registration or credentials to access:

- Malicious domain list maintained by the URLHaus project, available at `https://malware-filter.gitlab.io/malware-filter/urlhaus-filter-hosts.txt`.
- Malicious domain list maintained by CERT.pl, available at `https://hole.cert.pl/domains/v2/domains.txt`.
- Five public DNS resolvers: Cloudflare, at the addresses `1.1.1.1` and `1.1.1.2`; Cleanbrowsing, at the address `185.228.168.168`; AdGuard, at the address `94.140.14.15`; and Quad9, at the address `9.9.9.9`.

# Security Concerns

- Network: the script performs a large volume of parallel DNS queries, via GNU Parallel, against third-party public DNS servers. It is recommended to run it in a virtual machine or isolated environment, in order to avoid impacting production networks.
- Malicious content: the script queries domains present in malware indicator lists, but only performs DNS resolution of these domains. No HTTP or HTTPS request is ever made, nor is any content downloaded from these domains.
- Credentials: no credentials, SSH keys, or sensitive information are required to run this artifact.

# Installation

To set up the environment to run `test_resolvers.sh`, make sure the dependencies listed in the Dependencies section are installed.

Clone this repository on the machine that will run the script:

```
git clone https://github.com/jaquelinegon/DNS-Resolver-Analysis.git
```

Access the project directory:

```
cd DNS-Resolver-Analysis
```

Grant execution permission to the script:

```
chmod +x test_resolvers.sh
```

# Minimal Test

Once all dependencies are installed, run the script:

```
./test_resolvers.sh
```

The script itself checks, right at the beginning, whether all dependencies are present, halting execution with an error message if any are missing.

In the first few seconds of execution, watch the `### Checking average ping to nameservers.` and `### Testing safe hosts.` sections of the log, in which the script measures the latency of the five resolvers and tests the resolution of five known, legitimate domains.

The expected result is that these domains are resolved normally, with a valid IP, across all resolvers, and that the corresponding lines appear in the `dnsblock_result_<date>.csv` file. If this step completes without errors, the environment is correctly configured.

# Experiments

This work evaluates two aspects of the tested DNS resolvers, as presented in the paper: the malicious domain blocking rate and the response latency, comparing five public DNS resolvers against malicious domains collected from the CERT.pl and URLHaus lists.

The script starts by checking the required dependencies, downloads and consolidates the malicious domain lists, measures the latency to each resolver, tests the resolution of known legitimate domains to validate that the resolvers are responding correctly, and finally runs, in parallel, the query of each malicious domain against the five resolvers, recording whether each one blocked or resolved the domain normally.

As presented in the paper, the tested resolvers, the domain lists used, and the execution parameters are fixed and configured directly at the beginning of the `test_resolvers.sh` script, in the `ns_sp_array` and `ns_ip_array` variables and in the list download URLs.

By default, the script downloads and tests the updated malicious domain lists available at the time of execution. If it is necessary to reproduce the test with a fixed domain list, the download URLs at the beginning of the script must be replaced with the URLs of the desired files before execution.

Since the goal of the experiment is to analyze each resolver's blocking capability, the latency results and the total number of tested domains vary according to the execution date, as the lists are updated daily, and according to the network used by the machine running the test. The hardware and software specifications of the machine used in the paper are presented therein.

The following sections present the command to run the full experiment, which reproduces the two claims presented in the paper. Both claims are verified from the same script execution, with each one being extracted from a distinct part of the results file.

```
./test_resolvers.sh
```

This script downloads the malicious domain lists, measures the latency of the five resolvers, and then tests each domain against the five resolvers, generating the `dnsblock_result_<date>.csv` file.

# Claim "Malicious domain blocking rate per resolver"

It is verified that resolvers with security filtering block a significant portion of the tested malicious domains, as presented in the paper.

The script must be able to classify, for each domain and each resolver, whether it was blocked or resolved normally, allowing the calculation of the blocking rate per resolver.

From the `dnsblock_result_<date>.csv` file, each row corresponds to a tested domain, and each resolver column contains the returned IP, with the cell left empty when the domain is blocked. It is possible to calculate, per resolver column, the percentage of blocked domains relative to the total tested, reproducing the blocking rate presented in the paper.

# Claim "Latency comparison between resolvers"

It is verified that the tested resolvers show distinct average response times from one another, as presented in the paper.

This step is performed at the start of the script execution, in the `### Checking average ping to nameservers.` section of the log.

The `PING (ms)` line of the `dnsblock_result_<date>.csv` file shows the average response time, in milliseconds, of each resolver, allowing the latency comparison presented in the paper to be reproduced.

## Final Remarks

The blocking rate and latency results vary according to the execution date, since the malicious domain lists are updated daily, and according to the network characteristics of the machine used. This makes it possible to analyze the effectiveness of each resolver in filtering malicious domains, which is the focus of this work.

# LICENSE

This project is licensed under the GNU License - see the LICENSE file for details.
