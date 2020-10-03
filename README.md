# Docker for Variant Effect Predictor (VEP) + LOFTEE

## How can I use the Docker image?

We have the compiled image publicly available at [DockerHub](https://hub.docker.com/repository/docker/yosuketanigawa/docker-ensembl-vep-loftee).

### Pull the image from DockerHub

```{bash}
# if you have Docker environment
docker pull yosuketanigawa/docker-ensembl-vep-loftee:release_101.0

# if you have Singularity environment
singularity pull docker://yosuketanigawa/docker-ensembl-vep-loftee:release_101.0
```

### Cache the reference data for VEP

You may want to run `INSTALL.pl` to cache the reference dataset in your local directory.
Once you have successfully completed this step, you don't need to run it again.

```
docker run --user=root -it
--mount type=bind,src=/path_to_vep_data_directory,dst=/root/.vep \
yosuketanigawa/docker-ensembl-vep-loftee:release_101.0 \
perl INSTALL.pl -a cf -s homo_sapiens -y GRCh37
```

For GRCh38, you may want to specify `-y GRCh38`.

### Download the reference data for loftee plugin

There are additional reference data you need to download to run loftee.

```
# GRCh37
wget --timestamp https://personal.broadinstitute.org/konradk/loftee_data/GRCh37/GERP_scores.final.sorted.txt.gz
wget --timestamp https://personal.broadinstitute.org/konradk/loftee_data/GRCh37/GERP_scores.final.sorted.txt.gz.tbi
wget --timestamp https://personal.broadinstitute.org/konradk/loftee_data/GRCh37/human_ancestor.fa.gz
wget --timestamp https://personal.broadinstitute.org/konradk/loftee_data/GRCh37/human_ancestor.fa.gz.fai
wget --timestamp https://personal.broadinstitute.org/konradk/loftee_data/GRCh37/human_ancestor.fa.gz.gzi
wget --timestamp https://personal.broadinstitute.org/konradk/loftee_data/GRCh37/phylocsf_gerp.sql.gz
gzip -d phylocsf_gerp.sql.gz
```

For GRCh38, you need to download the reference from https://personal.broadinstitute.org/konradk/loftee_data/GRCh38.

### Run VEP with loftee

Here is an example to run the image. You may want to mount the data directories for vep and loftee, as well as the path containing your input data.
In the following example, we assume your `input.vcf` and the output file `output` are in your data directory `/cluster/u/${USER}`.

```
assembly=GRCh37

docker run -it \
  -w $(readlink -f $(pwd)) \
  --mount type=bind,src=/cluster/u/${USER},dst=/cluster/u/${USER} \
  --mount type=bind,src=/path_to_vep_data_directory,dst=/root/.vep \
  --mount type=bind,src=/path_to_loftee_data_directory/${assembly},dst=/root/.loftee,readonly \
  yosuketanigawa/docker-ensembl-vep-loftee:release_101.0 \
  vep \
  --offline --cache \
  --allele_number --everything \
  --assembly ${assembly} -i input.vcf -o output \
  --plugin LoF,loftee_path:/opt/vep/src/loftee-master,human_ancestor_fa:/root/.loftee/human_ancestor.fa.gz,conservation_file:/root/.loftee/phylocsf_gerp.sql,gerp_file:/root/.loftee/GERP_scores.final.sorted.txt.gz
```

## Technical notes

We added the loftee plugin and its dependencies to VEP's official docker image. This [commit (`ee03659`)](https://github.com/yk-tanigawa/docker-ensembl-vep-loftee/commit/ee03659f47fedd4d604142507555dadd2362d66b) illustrates the changes. We used VEP version 101 (the latest version avaiable at the moment) but it should be possible to apply similar patch to the future release of VEP.

## Reference

- [VEP official website](https://www.ensembl.org/vep)
- [VEP official Docker image](https://hub.docker.com/r/ensemblorg/ensembl-vep)
- [loftee plugin](https://github.com/konradjk/loftee)
- [loftee plugin reference data](https://personal.broadinstitute.org/konradk/loftee_data/)

