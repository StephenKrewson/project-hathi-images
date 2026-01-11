# HTRC ACS project

My project "Generating Basic Illustration Metadata" ran from May 2019 to August 2020. I presented some examples from the work, using PixPlot, during HathiTrust's Community Week in October 2020.

Deliverables:

- ACS [mid-year report](https://wiki.htrc.illinois.edu/display/COM/A+Half-Century+of+Illustrated+Pages%3A+ACS+Lab+Notes)
- ACS [final reports](https://wiki.htrc.illinois.edu/display/COM/Advanced+Collaborative+Support+%29ACS%29+Awards); my [pdf](https://wiki.htrc.illinois.edu/download/attachments/31588360/ACS-2019-2020-FinalReport-HathiTrust%2BNames.pdf?version=1&modificationDate=1595948576000&api=v2)
- Project [Zenodo repository](https://zenodo.org/record/3940528)
- Community Week slides: `presentations/ACS_2020-HT-Community-Week`
- Complete 1800-1850 [image dataset](https://console.cloud.google.com/storage/browser/hathitrust-full_1800-50) on Google Cloud Storage

Once the dissertation is submitted, I would like to update the Zenodo:

- Include Jupyter notebooks for working with PixPlot
- Standardize the image names

However, this is a low priority. The update really only makes sense if I'm able to move to a PixPlot-based workflow where the data can be shared.


## Local and VM environments

N.B. Check the `_datasets` and `_image-labeling` folders on my computer for additional resources, such as `ivpy` montage tool.

Locally, I run Ubuntu 20.04 in WSL2 and use Python 3.8. I make sure to have an updated Google SDK.

On the VM, I use `conda` and Python 3.7, to match PixPlot's dependencies. To log onto the VM:

```gcloud beta compute ssh --zone "us-east1-c" "hathi-images-us-east1-c" --project "global-matrix-242515"```

One nice trick is to `grep` for this command in this README and then paste into the terminal. 

## How to create a PixPlot visualization from the ACS data

The basic steps used for my HathiTrust presentation and dissertation case studies are outlined below:

### 1. Acquire project metadata files

Download the necessary project metadata files from Zenodo. There are three:

- hathi (TODO: rename/reupload them all)
- hathicols
- rois

### 2. Search for a subset of volumes and merge all ROI metadata

This process is done with the `query_to_pixplot.ipynb` notebook.

```
# TODO: Save a separate CSV of the bucket URLs; drop the 'filename' column
gs_urls = df['filename'].map(lambda x: "gs://hathitrust-full_1800-50/" + x)
gs_urls.to_csv("{}_gs-urls.csv".format(search_label), index=None, header=None)
```

Identify a subset of volumes by filtering the Hathifile. Merge with the master ROI file to get an augmented set of fields for all illustrated regions corresponding to those volumes. For example, the search query might be on the `imprint` field in the Hathifile for a certain publisher.

- The output is a CSV file that PixPlot can accept. Since PixPlot can smartly ignore extra fields, I also add a column to this CSV with the full `gs://hathitrust-full_1800-50` bucket prefix (TODO!). I version these CSVs in the repo.
- Output the CSV peter-parley_gs-urls.csv, for use in downloading images from GCS to VM.
- I also record the queries and number of results in the JSON file `metadata/pixplot_queries.json`.

### 3. Transfer query metadata to VM and download all needed JPEGs

To transfer a pair of query CSVs to the VM, use [`gcloud compute scp`](https://cloud.google.com/sdk/gcloud/reference/compute/scp):

```
gcloud compute scp peter-parley_pixplot.csv hathi-images-us-east1-c:~/peter-parley/metadata
gcloud compute scp peter-parley_gs-urls.csv hathi-images-us-east1-c:~/peter-parley/metadata
```

Make sure your default zone is the same as the zone of your default project instance. There's no point in keeping the repo on the VM, which should be considered an ephemeral resource for running neural network inference.

To download from bucket to VM, run the [`gsutil cp`](https://cloud.google.com/storage/docs/gsutil/commands/cp) command on the list of paths prefixed with the GCS bucket. I suspect that `gsutil` is preferred for jobs that stay entirely within the Google network. Make sure to do this from the `peter-parley` directory within a `tmux` session:

```
cat ./metadata/peter-parley_gs-urls.csv | gsutil -m cp -I ./images/
```

A good convention for staying organized is the following:

```
peter-parley/
├── images
│   ├── [...]
│   └── wu.89097026173_00000703_00.jpg
├── metadata
│   ├── peter-parley_gs-urls.csv
│   └── peter-parley_pixplot.csv
└── output
    ├── assets
    ├── data
    ├── favicon.ico
    └── index.html

3 directories, 0 files
```

Each search query should be kept in its own directory. In some cases, I will want to merge them (for instance, with the publishers) before running PixPlot.

### 4. Run PixPlot analysis and check results

From the same `tmux` session, analyze the downloaded JPEGs with PixPlot, using the full metadata file. If the PixPlot job stalls, you can restart and PixPlot knows enough to skip image assets that already exist:

```python
pixplot --images ./images/*.jpg --metadata ./metadata/peter-parley_pixplot.csv
```

PixPlot will produce a folder structure like this:

```
output/
├── assets
│   ├── css
│   ├── images
│   ├── js
│   ├── json
│   └── vendor
├── data
│   ├── atlases
│   ├── heightmaps
│   ├── hotspots
│   ├── image-vectors
│   ├── imagelists
│   ├── layouts
│   ├── manifest.json
│   ├── manifests
│   ├── metadata
│   ├── originals
│   └── thumbs
├── favicon.ico
└── index.html

17 directories, 3 files
```

### 5. Download output directory for local viewing

To transfer from the VM to my local machine, we must again use `gcloud compute scp` (with the recursive option for transferring an entire directory) since the VM is protected:

```
# Transfer to C: drive for most space, run from a tmux session
gcloud compute scp --recurse hathi-images-us-east1-c:~/peter-parley/output /mnt/c/Users/stephen-krewson/Documents/_datasets/peter-parley/output
```

If this job fails and needs to be rerun, I need to research how to to skip existing files.

## Appendix: Google configuration and credentials

So far, I have not needed to use Python to access the Google Cloud APIs. Here's how to install:

```
# Within the virtual environment
pip install --upgrade google-cloud-storage
```

Create a key under "Service Accounts" in the Console. Then export it to the environment:

```
# Before running a notebook that uses the google.cloud package
export GOOGLE_APPLICATION_CREDENTIALS="global-matrix-242515-49432d870e22.json"
```

The API docs can be found at https://cloud.google.com/storage/docs/reference/libraries.

To view my current projects, I use the [`gcloud config`](https://cloud.google.com/sdk/gcloud/reference/config/set) command:

```
# Find out zone and other info for currently provisioned VMs
gcloud compute instances list

# See settings for 'compute' and 'core' groups
gcloud config list

# Set zone in default config to match the VM
gcloud config set compute/zone us-east1-c

# Now it's simple to log in (from a tmux session):
gcloud compute ssh hathi-images-us-east1-c
```

Follow the fastai steps to create a GPU-enabled VM: https://course.fast.ai/start_gcp. This worked for me in the past, so why mess with it. Pick a zone such as us-west1-b with more GPUs and GPU options. For preemptible instances, it is becoming harder to find resources: https://cloud.google.com/compute/docs/gpus/gpu-regions-zones.

- [ ] Follow the instructions to install PixPlot, which is currently recommending Anaconda. This is another good reason to use fastai's steps: https://github.com/YaleDHLab/pix-plot. Use `conda activate pixplot` to enter the environment I created. You DO NOT need PixPlot locally, just the python web server and a browser!


As of April 2021, I am using Python 3.8 with pip 20.0.2 on WSL2, Ubuntu 20.0.4. This gives me the closest match with my Linux VM and is extremely fast. The dependencies are simple: I need jupyter and pandas for working in the query notebook. I install these packages within a virtual environment.

Like `pip`, the `venv` module is [installed already](https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/) with Python 3. Just use the conventional name `env` since, unlike conda, the virtual environment is located in your project's directory.

```
python3 -m venv env
source env/bin/activate
which python3
python3 -m pip install <libraries>
deactivate
```

Make sure env/* is in the .gitignore.

## Appendix: Getting ACS data from HTRC to GCS

Boris provided the crops via a temporary `rsync` endpoint at `proxy.htrc.indiana.edu::krewson/crops`. From within a `tmux` session on a VM with a 2TB boot drive, I ran:

```
$ rsync -aP --itemize-changes proxy.htrc.indiana.edu::krewson/crops .
```

This retrieved the entire `crops` folder containing the stubbytree hierarchy to the VM's persistent disk. The size was 553GB. Unfortunately, `gsutil rsync` tool did not work, since the HTRC server was not in Cloud Object format.

Since GPU quotas and buckets are also associated with projects, I stuck with the `global-matrix-245215` project. From the VM, after renaming the `crops` folder, I ran the  [`gsutil rsync`](https://cloud.google.com/storage/docs/gsutil/commands/rsync) command to copy the data to the cheaper bucket storage:

```
# Note that gsutil commands only work with cloud buckets! Not VMs
# The -m flag optimizes/parallelizes jobs involving cloud buckets
gsutil -m rsync -r hathitrust-full_1800-50/ gs://hathitrust-full_1800-50
```

I first tested with the flag `-n` to see what would happen. Don't select `-d` since this is a one-time transfer, and there is no need to delete extra files from the bucket.

A key principle is to keep lots of data in the buckets, where it's cheaper to store (about $6 per month for the 600 GB ACS dataset). Then, extract subsets and run compute-intensive tasks on them (for example, PixPlot vectorization).

- Conclusion: make boot disk the 2TB maximum. Mounting and resizing extra disks requires LOTS of extra knowledge of df and lsblk and growpart. Something to study up on, though.
- Other breakthrough: run tmux ON the remote server!! Then detach and exit. Job will hum merrily along. Otherwise, as soon as my laptop goes to sleep, the SSH connection is broken and it's all over. And it was too weird to run tmux within tmux. Just like the Zoo c. 2017!

Once, because I had zipped training images on Windows, I needed the [P7 tool](https://anaconda.org/bioconda/p7zip) to unzip them on the Linux Google VM. You need to create a conda environment to get the right permissions -- difficult!

```
conda install -c bioconda p7zip
7za x [.zip]
```


## HathiTrust APIs

UPDATE: I now favor `pip` and `venv` for all dependency management. `conda` is too unpredictable. Bibliographic metadata can be fetched from HTRC workset [toolkit](https://github.com/htrc/HTRC-WorksetToolkit). Hathi's UI for advanced catalog search is [here](https://catalog.hathitrust.org/Search/Advanced). You still cannot add to a collection from catalog search! This is very limiting.

I have used and contributed a minor version update to the [hathitrust-api](https://github.com/rlmv/hathitrust-api) library. It provides access to Hathi's Data, Bib, and Solr (search) APIs. Install with `pip install hathitrust-api`.

My HTRC capsule credentials are as follows:

Login: `stephenkrewson` (password saved with Chrome)
Email: `stephen.krewson@yale.edu`
