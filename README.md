# ![nf-core/llrnaseq](docs/images/nf-core-llrnaseq_logo.png)

[![GitHub Actions CI
Status](https://github.com/nf-core/llrnaseq/workflows/nf-core%20CI/badge.svg)](https://github.com/nf-core/llrnaseq/actions?query=workflow%3A%22nf-core+CI%22)
[![GitHub Actions Linting
Status](https://github.com/nf-core/llrnaseq/workflows/nf-core%20linting/badge.svg)](https://github.com/nf-core/llrnaseq/actions?query=workflow%3A%22nf-core+linting%22)
[![AWS
CI](https://img.shields.io/badge/CI%20tests-full%20size-FF9900?labelColor=000000&logo=Amazon%20AWS)](https://nf-co.re/llrnaseq/results)
[![Cite with
Zenodo](http://img.shields.io/badge/DOI-10.5281/zenodo.XXXXXXX-1073c8?labelColor=000000)](https://doi.org/10.5281/zenodo.XXXXXXX)

[![Nextflow](https://img.shields.io/badge/nextflow%20DSL2-%E2%89%A521.04.3-23aa62.svg?labelColor=000000)](https://www.nextflow.io/)
[![run with
conda](http://img.shields.io/badge/run%20with-conda-3EB049?labelColor=000000&logo=anaconda)](https://docs.conda.io/en/latest/)
[![run with
docker](https://img.shields.io/badge/run%20with-docker-0db7ed?labelColor=000000&logo=docker)](https://www.docker.com/)
[![run with
singularity](https://img.shields.io/badge/run%20with-singularity-1d355c.svg?labelColor=000000)](https://sylabs.io/docs/)

[![Get help on
Slack](http://img.shields.io/badge/slack-nf--core%20%23llrnaseq-4A154B?labelColor=000000&logo=slack)](https://nfcore.slack.com/channels/llrnaseq)
[![Follow on
Twitter](http://img.shields.io/badge/twitter-%40nf__core-1DA1F2?labelColor=000000&logo=twitter)](https://twitter.com/nf_core)
[![Watch on
YouTube](http://img.shields.io/badge/youtube-nf--core-FF0000?labelColor=000000&logo=youtube)](https://www.youtube.com/c/nf-core)

## Introduction

<!-- TODO nf-core: Write a 1-2 sentence summary of what data the pipeline is for and what it does -->
**SpikyClip/llrnaseq** is a simple RNA-seq pipeline adapted to the Latrobe
Institute of Molecular Science (LIMS) High Performance Computing Cluster
(HPCC).

The pipeline is built using [Nextflow](https://www.nextflow.io), a workflow
tool to run tasks across multiple compute infrastructures in a very portable
manner. It uses Docker/Singularity containers making installation trivial and
results highly reproducible. The [Nextflow
DSL2](https://www.nextflow.io/docs/latest/dsl2.html) implementation of this
pipeline uses one container per process which makes it much easier to maintain
and update software dependencies. 

However, as Docker is unavailable on the LIMS-HPCC the local version of
Singularity is outdated, this pipeline has a `-profile lims` profile to switch
the process environment manager to [environment
modules](http://modules.sourceforge.net/). Nevertheless, it still has the
options to use the other native `docker` and `singularity` profiles which
should work if run on a computer that has access to those programs, or if the
LIMS-HPCC is updated to support them in the future.

<!-- TODO nf-core: Add full-sized test dataset and amend the paragraph below if applicable
On release, automated continuous integration tests run the pipeline on a full-sized dataset on the AWS cloud infrastructure. This ensures that the pipeline runs on AWS, has sensible resource allocation defaults set to run on real-world datasets, and permits the persistent storage of results to benchmark between pipeline releases and other analysis sources. The results obtained from the full-sized test can be viewed on the [nf-core website](https://nf-co.re/llrnaseq/results). -->

## Pipeline summary

<!-- TODO nf-core: Fill in short bullet-pointed list of the default steps in the pipeline -->

1. Read QC
   ([`FastQC`](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/))
2. Present QC for raw reads ([`MultiQC`](http://multiqc.info/))
3. Trim reads ([`Trim
   Galore`](https://www.bioinformatics.babraham.ac.uk/projects/trim_galore/))
4. Index genome ([`Hisat2`](http://daehwankimlab.github.io/hisat2/))
5. Align reads ([`Hisat2`](http://daehwankimlab.github.io/hisat2/))
6. Sort and index alignments ([`Samtools`](http://www.htslib.org/))
7. Read quantification ([`featureCounts`](http://subread.sourceforge.net/))

## Quick Start

1. Install [`Nextflow`](https://nf-co.re/usage/installation) (`>=21.04.3`) (see
   [`installation.md`](docs/installation.md) for more information)

2. If executing the pipeline on a computer that can support it, install any of
   [`Docker`](https://docs.docker.com/engine/installation/),
   [`Singularity`](https://www.sylabs.io/guides/3.0/user-guide/),
   [`Podman`](https://podman.io/),
   [`Shifter`](https://nersc.gitlab.io/development/shifter/how-to-use/) or
   [`Charliecloud`](https://hpc.github.io/charliecloud/) for full pipeline
   reproducibility _(please only use [`Conda`](https://conda.io/miniconda.html)
   as a last resort; see
   [docs](https://nf-co.re/usage/configuration#basic-configuration-profiles))_.
   If executing the pipeline on the LIMS-HPCC, ignore this step.

3. Download the pipeline and test it on a minimal dataset with a single
   command:
    1. If running on the LIMS-HPCC:
       ```
       module load java/1.8.0_66

       nextflow run SpikyClip/llrnaseq -r main -profile test,lims
       ```
    2. If running on a `Docker`/`Singularity` capable machine:
       ```
       nextflow run SpikyClip/llrnaseq -r main -profile test,<docker/singularity/podman/shifter/charliecloud/conda/institute>
       ```
    > * If you are using `singularity` then the pipeline will auto-detect this
    >   and attempt to download the Singularity images directly as opposed to
    >   performing a conversion from Docker images. If you are persistently
    >   observing issues downloading Singularity images directly due to timeout
    >   or network issues then please use the
    >   `--singularity_pull_docker_container` parameter to pull and convert the
    >   Docker image instead. Alternatively, it is highly recommended to use
    >   the [`nf-core
    >   download`](https://nf-co.re/tools/#downloading-pipelines-for-offline-use)
    >   command to pre-download all of the required containers before running
    >   the pipeline and to set the [`NXF_SINGULARITY_CACHEDIR` or
    >   `singularity.cacheDir`](https://www.nextflow.io/docs/latest/singularity.html?#singularity-docker-hub)
    >   Nextflow options to be able to store and re-use the images from a
    >   central location for future pipeline runs.
    > * If you are using `conda`, it is highly recommended to use the
    >   [`NXF_CONDA_CACHEDIR` or
    >   `conda.cacheDir`](https://www.nextflow.io/docs/latest/conda.html)
    >   settings to store the environments in a central location for future
    >   pipeline runs.

4. Start running your own analysis!

   1. You will first need to [create a samplesheet](docs/usage.md) with
      information about the samples you would like to analyse before running
      the pipeline.

   2. The pipeline can pull some common genome references used for alignment
      from [Illumina iGenomes](https://nf-co.re/usage/reference_genomes). Check
      out [`igenomes.config`](conf/igenomes.config) to see the full list of
      iGenomes this pipeline recognises.

      ```
      nextflow run llrnaseq -r main \
          -profile lims \
          --input <samplesheet>.csv \
          --genome GRCh37
      ```
   3. Alternatively, you can specify `genome.fa` and `genome.gtf` explicitly:

      ```
      nextflow run llrnaseq -r main \
          -profile lims \
          --input <samplesheet>.csv \
          --fasta <genome>.fa> \
          --gtf <annotation>.gtf
      ```
   4. If running a job on the LIMS-HPCC, wrap the `nextflow run` command in a
      shell script (e.g. `run_pipeline.sh`) and submit it using `slurm`:

      ```console
      sbatch run_pipeline.sh
      ```
      Consider specifying the estimated time needed in the script if the job
      may take more than 8 hours using `#SBATCH --time=<HH>:<MM>:<SS>`. This is
      to avoid the pipeline ending prematurely. However, if the job is
      interrupted, it may be resumed with the nextflow `-resume` flag.

## Documentation

The SpikyClip/llrnaseq pipeline comes with documentation about the pipeline
[usage](docs/usage.md), [parameters](docs.parameters.md) and
[output](docs/output.md).

## Credits

SpikyClip/llrnaseq was originally written by Vikesh Ajith.

We thank the following people for their extensive assistance in the development
of this pipeline:

<!-- TODO nf-core: If applicable, make list of people who have also contributed -->

## Contributions and Support

If you would like to contribute to this pipeline, please see the [contributing
guidelines](.github/CONTRIBUTING.md).

<!-- For further information or help, don't hesitate to get in touch on the [Slack
`#llrnaseq` channel](https://nfcore.slack.com/channels/llrnaseq) (you can join
with [this invite](https://nf-co.re/join/slack)). -->

## Citations

<!-- TODO nf-core: Add citation for pipeline after first release. Uncomment lines below and update Zenodo doi and badge at the top of this file. -->
<!-- If you use  nf-core/llrnaseq for your analysis, please cite it using the following doi: [10.5281/zenodo.XXXXXX](https://doi.org/10.5281/zenodo.XXXXXX) -->

<!-- TODO nf-core: Add bibliography of tools and data used in your pipeline -->
An extensive list of references for the tools used by the pipeline can be found
in the [`CITATIONS.md`](CITATIONS.md) file.

You can cite the `nf-core` publication as follows:

> **The nf-core framework for community-curated bioinformatics pipelines.**
>
> Philip Ewels, Alexander Peltzer, Sven Fillinger, Harshil Patel, Johannes
> Alneberg, Andreas Wilm, Maxime Ulysse Garcia, Paolo Di Tommaso & Sven
> Nahnsen.
>
> _Nat Biotechnol._ 2020 Feb 13. doi:
> [10.1038/s41587-020-0439-x](https://dx.doi.org/10.1038/s41587-020-0439-x).
