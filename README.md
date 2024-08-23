# Monopogen Setup Guide

This guide provides instructions for setting up Monopogen using a Conda environment and making necessary modifications to streamline its use.
1. Set Up Conda Environment

Create and activate a Conda environment with the required packages.

conda create -n monopogen_env  python 
conda activate monopogen_env

# Install required packages
conda install -c bioconda samtools bcftools py-bgzip openjdk 
2. Clone Monopogen Repository

Clone the Monopogen repository from GitHub.

git clone https://github.com/KChen-lab/Monopogen.git
cd Monopogen
pip install -e . 
3. Modify Monopogen.py Script

Edit `Monopogen.py` to use Conda-installed tools.

Replace:
out = os.path.abspath(args.out)
samtools  = os.path.abspath(args.app_path) + "/samtools"
bcftools = os.path.abspath(args.app_path) + "/bcftools"
bgzip = os.path.abspath(args.app_path) + "/bgzip"
With:
out = os.path.abspath(args.out)
samtools = shutil.which("samtools")
bcftools = shutil.which("bcftools")
bgzip = shutil.which("bgzip")
java = shutil.which("java")
beagle = os.path.abspath(args.app_path) + "/ beagle.27Jul16.86a.jar"
4.Run Preprocessing

Ensure necessary files are prepared as per the original instructions, then run preprocessing.

path="/path/to/Monopogen"
${path}/src/Monopogen.py preProcess -b bam.lst -o retina -a ${path}/apps -t 1
5. Germline Processing

Run the following steps individually:
# Step 1: Run bcftools pileup and normalization

/path/to/conda/envs/monopogen_env/bin/bcftools mpileup -b retina/Bam/chr20.filter.bam.lst -f GRCh38.chr20.fa -r chr20 -q 20 -Q 20 -d 10000000 | /path/to/conda/envs/monopogen_env/bin/bcftools call -mv -Ob | /path/to/conda/envs/monopogen_env/bin/bcftools norm -m-both -f GRCh38.chr20.fa | grep -v INDEL | /path/to/conda/envs/monopogen_env/bin/bgzip -c > retina/germline/chr20.gl.vcf.gz
# Step 2: Run Beagle for imputation and genotype phasing

/path/to/conda/envs/monopogen_env/bin/java -Xmx20g -jar /path/to/Monopogen/apps/ beagle.27Jul16.86a.jar gl=retina/germline/chr20.gl.vcf.gz ref=/path/to/reference/chr20.filtered.shapeit2-duohmm-phased.vcf.gz chrom=chr20 out=retina/germline/chr20.gp impute=false modelscale=2 nthreads=24 gprobs=true niterations=0
# Step 3: Filter the output using zgrep

zgrep -v '0/0' retina/germline/chr20.gp.vcf.gz > retina/germline/chr20.germline.vcf
# Step 4: Run Beagle again for phasing

/path/to/conda/envs/monopogen_env/bin/java -Xmx20g -jar /path/to/Monopogen/apps/ beagle.27Jul16.86a.jar gt=retina/germline/chr20.germline.vcf ref=/path/to/reference/chr20.filtered.shapeit2-duohmm-phased.vcf.gz chrom=chr20 out=retina/germline/chr20.phased impute=false modelscale=2 nthreads=24 gprobs=true niterations=0


For more detailed information on files needed, customization of files and scripts , refer the original Monopogen github - https://github.com/KChen-lab/Monopogen?tab=readme-ov-file
