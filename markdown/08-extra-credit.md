# Welcome!

This takes `eukaryota` data and run a gene sequence alignment on the data.

Here's the pipeline, split up into 4 parts:

```JSON
{
  "import-data": [
    "bionode-ncbi search genome eukaryota",
    "dat import -d eukaryota --key=uid"
  ],
  "search-ncbi": [
    "dat export -d eukaryota",
    "grep Guillardia",
    "tool-stream extractProperty assemblyid",
    "bionode-ncbi download assembly -",
    "tool-stream collectMatch status completed",
    "tool-stream extractProperty uid",
    "bionode-ncbi link assembly bioproject -",
    "tool-stream extractProperty destUID",
    "bionode-ncbi link bioproject sra -",
    "tool-stream extractProperty destUID",
    "grep 35526",
    "bionode-ncbi download sra -",
    "tool-stream collectMatch status completed",
    "tee > metadata.json"
  ],
  "index-and-align": [
    "cat metadata.json",
    "bionode-sra fastq-dump -",
    "tool-stream extractProperty destFile",
    "bionode-bwa mem **/*fna.gz"
  ],
  "convert-to-bam": [
    "bionode-sam 35526/SRR070675.sam"
  ]
}
```

First, make sure your terminal is inside the `eukaryota` directory that you cloned earlier.

Create a new empty file called `gasket.json` and copy/paste the above pipeline into the file. Verify that the copy/paste worked by typing `gasket ls` and verify that it prints out the 4 different named pipelines:

```
$ gasket ls
import-data
search-ncbi
index-and-align
convert-to-bam
```

You can run a named pipeline by running `gasket run NAME`, where `NAME` is the name of your pipeline.

We can skip the `import-data` pipeline since we have already cloned the `eukaryota` data into a local dat database.

The second pipeline, `search-ncbi`, starts by doing `dat export` and then takes the data from dat and uses it to download additional datasets from NCBI (National Center for Biotechnology Information), a server where lots of bioinformatics datasets are hosted.

Run this pipeline:

```
gasket run search-ncbi
```

## More output

By default there is no output while the pipeline is running, but if you want to see what's happening under the hood you can run the pipeline again with the `DEBUG` environment variable set to `*` (to show all possible debug output):

```
DEBUG=* gasket run search-ncbi
```

This pipeline should create a couple of folders and download some files into them.

## Sequence alignment

The next pipeline, `index-and-align`, uses the downloaded genetic data from the `search-ncbi` pipeline and runs process called a DNA sequence alignment.

Try running the next pipeline, this one will take a few minutes to complete:

```
DEBUG=* gasket run index-and-align
```

When this finishes, it should have created even more files in the folders from the previous step.

The final pipeline, `convert-to-bam`, converts the output of the alignment into a different file format. This pipeline should be pretty fast.

```
DEBUG=* gasket run convert-to-bam
```

Congratulations, you just ran a DNA sequence alignment!
