# goblin
GOBLIN - Generate trusted prOteins to supplement BacteriaL annotatIoN

## Inspiration

I've been using Prokka for many years now, and have made use of the `--proteins` feature
whenever possible. This feature allows you provide a set of proteins to use for a first-pass
annotation. Depending on how good the annotations you provide were, the number of predicted 
_hypothetical_ proteins could be significantly reduced.

Fast forward some years, and as part of [Staphopia](https://staphopia.emory.edu/) I started using
a set of _Staphylococcus aureus_ proteins for input to Prokka. This worked fine, but as I started
developing [Bactopia](https://bactopia.github.io/), I wanted to do this automatically for any
bacterial species.

I stumbled upon [Thanh Lê's make_prokka_db](https://github.com/thanhleviet/make_prokka_db), which
heavily inspired my process in Bactopia for automatic creation of these proteins files.

Fast forward a few more years, and I thought it was time to pull this out of Bactopia,
and turn it into a stand alone tool. Thus, goblin was born.

## Main Steps

1. Query NCBI, can be an organism name, tax id, or file of accessions
2. Download GenBank for each hit
3. Extract genome size and proteins for each genome
4. Cluster proteins using CD-HIT
5. Output summary stats

## Quick Start

```{bash}
goblin --query 1280 --prefix saureus --limit 10
Downloading genomes from NCBI...

Parsing GenBank files...

Running CD-HIT...

Cleaning up temporary files...

                                         Goblin Summary
┏━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ key                     ┃ value                                                               ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ command                 │ goblin --query 1280 --prefix saureus --limit 10                     │
│ date                    │ 2023-01-19T06:56:31Z                                                │
│ versions                │ goblin:0.0.1;biopython:1.80;cd-hit:4.8.1;ncbi-genome-download:0.3.1 │
│ total_query_hits        │ 931                                                                 │
│ total_genomes           │ 10                                                                  │
│ total_cds               │ 27634                                                               │
│ total_hypothetical      │ 0                                                                   │
│ total_pseudogenes       │ 797                                                                 │
│ total_proteins          │ 26837                                                               │
│ total_clusters          │ 4039                                                                │
│ percent_reduced         │ 0.85                                                                │
│ genome_size_min         │ 2746829                                                             │
│ genome_size_median      │ 2837965                                                             │
│ genome_size_mean        │ 2849240                                                             │
│ genome_size_max         │ 3018303                                                             │
│ genome_size_description │ Genome size values are based on 10 genomes                          │
└─────────────────────────┴─────────────────────────────────────────────────────────────────────┘

ls saureus/
genome_size.json  goblin-summary.txt  goblin.log.gz  ncbi-accessions.txt  ncbi-metadata.txt  saureus.faa.gz
```

## Installation

Goblin is available from Bioconda, which means it's also available from Docker and Singularity.
```{bash}
conda create -n goblin -c conda-forge -c bioconda goblin
```

## Usage

```{bash}

 Usage: goblin [OPTIONS]

 Generate trusted prOteins to supplement BacteriaL annotatIoN

╭─ Required ────────────────────────────────────────────────────────────────────────────╮
│ *  --query     TEXT  Download genomes for a given query (organism, taxid, file of     │
│                      accessions)                                                      │
│                      [required]                                                       │
│ *  --prefix    TEXT  Prefix to use to save outputs [required]                         │
╰───────────────────────────────────────────────────────────────────────────────────────╯
╭─ ncbi-genome-download ────────────────────────────────────────────────────────────────╮
│ --assembly_level    [all|complete|chromosome|scaffo  Assembly levels of genomes to    │
│                     ld|contig|complete,chromosome|c  download                         │
│                     hromosome,complete]              [default: complete]              │
│ --limit             INTEGER                          Maximum number of genomes to     │
│                                                      download                         │
│                                                      [default: 100]                   │
╰───────────────────────────────────────────────────────────────────────────────────────╯
╭─ CD-HIT ──────────────────────────────────────────────────────────────────────────────╮
│ --identity        FLOAT    CD-HIT (-c) sequence identity threshold [default: 0.9]     │
│ --overlap         FLOAT    CD-HIT (-s) length difference cutoff [default: 0.8]        │
│ --max_memory      INTEGER  CD-HIT (-M) memory limit (in MB) (0 removes memory limits) │
│                            [default: 0]                                               │
│ --fast_cluster             Use CD-HIT (-g 0) fast clustering algorithm                │
╰───────────────────────────────────────────────────────────────────────────────────────╯
╭─ Options ─────────────────────────────────────────────────────────────────────────────╮
│ --version                  Show the version and exit.                                 │
│ --is_taxid                 Given taxid should be treated as a taxid (not a species    │
│                            taxid)                                                     │
│ --outdir          TEXT     Directory to save output files [default: ./]               │
│ --force                    Overwrite existing files                                   │
│ --compress                 Compress final clustered proteins and log file             │
│ --cpus            INTEGER  Number of cpus to use [default: 1]                         │
│ --keep_files               Keep all downloaded and intermediate files                 │
│ --debug                    Print additional debug information                         │
│ --check                    Check dependencies are installed, then exit                │
│ --help        -h           Show this message and exit.                                │
╰───────────────────────────────────────────────────────────────────────────────────────╯

```

### --query

This is the query you would like to submit to NCBI. This query can be a bacterial genus or species, a taxon ID,
or a set of NCBI Assembly accessions to use.

### --prefix

This will determine how to name the output files.

### --assembly_level

While I recommend you stick with `complete` (_the default_), I recognize for many bacterial species there are not
enough (_if any_) completed genomes. So, use this parameter to adjust the level of completeness to download.

### --limit

Use this to limit the number of genomes to use for the creation of your proteins file. The default is 100, which is
probably reasonable. Please consider as the number increases, it's likely the size of the proteins file will grow, which
in turn will cause annotators like Prokka to take longer to complete.

### --identity & --overlap

Adjust how similar proteins must be to be clustered by CD-HIT. I imagine, there may be use cases where you might want to 
play around with these parameters.

### --max_memory

There is the chance if you include a large amount of genomes CD-HIT may use too much memory. You can use the parameter
to force CD-HIT to stay under a certain amount of memory.

### --fast_cluster

This will enable CD-HIT's faster clustering algorithm, which is yes, much faster, but comes at the cost of accuracy. I've
personally never had a need to use this, but you might.

### --is_taxid

By default `--species-taxid` is used for tax ID queries. By using the species taxid we can capture all descendent tax IDs.
If this isn't what you want, then this parameter is the one to limit queries to only your tax ID of interest.

### --keep_files && --debug

When the need arises, use these to figure out where issues might be happening.

## Output Files

Filename | Description
---------|------------
`<PREFIX>.faa.gz` | The final set of clustered proteins
`goblin-summary.json` | A summary of the run in JSON format
`goblin-summary.tsv` | A summary of the run in TSV format
`goblin.log.gz` | Outputs from each of the commands executed
`ncbi-accessions.txt` | The set of accessions that were downloaded and used
`ncbi-metadata.txt` | Information about the accessions downloaded

## FAQ

* _Can I provided multiple species, tax IDs, etc...?_  

  No. This is a design choice. My primary use-case will be with Bactopia, where Nextflow will handle the submission
  of jobs. So, one at a time makes more sense. It also keeps things simple.

* _Will this work with Bakta?_

  No. I considered making this compatible with [Bakta](https://github.com/oschwengers/bakta), but I honestly don't think
  an automated method like Goblin could ever get close to the databases used by Bakta. A lot of effort was put in by Oliver
  to make sure the databases in Bakta are top notch. So, I honestly recommend to use them directly. However, there may
  be a use case for `--proteins` in Bakta, but if that's the case there's probably a really good reason. Goblin is not a
  good reason 😉!

## Feedback

Please file questions, bugs or ideas to the [Issue Tracker](https://github.com/rpetit3/goblin/issues)

## Acknowledgements

I would like to personally extend my many thanks and gratitude to the authors of these software packages. Really, thank you very much!

## Citation
If you use this tool please cite the following:

__[goblin](https://github.com/rpetit3/pbptyper)__  
Petit III RA [goblin: Generate trusted prOteins to supplement BacteriaL annotatIoN](https://github.com/rpetit3/pbptyper) (GitHub)  

__[Biopython](https://biopython.org/)__  
Cock PJA, Antao T, Chang JT, Chapman BA, Cox CJ, Dalke A, Friedberg I, Hamelryck T, Kauff F, Wilczynski B, de Hoon MJL [Biopython: freely available Python tools for computational molecular biology and bioinformatics.](https://doi.org/10.1093/bioinformatics/btp163) Bioinformatics , 25(11), 1422–1423. (2009)  

__[CD-HIT-EST](https://github.com/weizhongli/cdhit)__  
Fu L, Niu B, Zhu Z, Wu S, Li W [CD-HIT: accelerated for clustering the next-generation sequencing data](http://dx.doi.org/10.1093/bioinformatics/bts565). _Bioinformatics_ 28, 3150–3152 (2012)  

__[ncbi-genome-download](https://github.com/kblin/ncbi-genome-download)__   
Blin K [ncbi-genome-download: Scripts to download genomes from the NCBI FTP servers](https://github.com/kblin/ncbi-genome-download) (GitHub)  

__[Pigz](https://zlib.net/pigz/)__  
Adler M. [pigz: A parallel implementation of gzip for modern multi-processor, multi-core machines.](https://zlib.net/pigz/) _Jet Propulsion Laboratory_ (2015)  

## Author

* Robert A. Petit III
* Web: [https://www.robertpetit.com](https://www.robertpetit.com)
* Twitter: [@rpetit3](https://twitter.com/rpetit3)
