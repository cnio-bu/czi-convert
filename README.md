# CZI to TIFF Conversion Pipeline

A Snakemake pipeline for batch conversion of Carl Zeiss Image (CZI) files to TIFF format using Bio-Formats' `bfconvert` tool.

## Overview

This pipeline automatically discovers all CZI files in a specified input directory and converts them to TIFF format with configurable parameters. It leverages Bio-Formats for reliable handling of complex microscopy image formats.

## Requirements

- **Snakemake** (workflow management)
- **Conda/Mamba** (environment management)
- **bftools** (Bio-Formats command-line tools, installed via conda)

## Installation

1. Clone this repository:
   ```bash
   git clone <repository-url>
   cd czi-convert
   ```

2. Install Snakemake (if not already installed):
   ```bash
   conda install -c conda-forge -c bioconda snakemake
   ```

3. The pipeline will automatically create the required conda environment with `bftools` on first run.

## Configuration

1. Copy the example configuration file:
   ```bash
   cp config/config.yaml.example config/config.yaml
   ```

2. Edit `config/config.yaml` to set your parameters:
   ```yaml
   input_dir: "/path/to/your/czi/files"

   params: "-overwrite -compression LZW"

   resources:
     mem_mb: 64000      # Memory allocation in MB
     threads: 1          # Number of threads per conversion
     runtime: 30         # Maximum runtime in minutes
   ```

### Configuration Parameters

#### `input_dir`
Path to the directory containing your `.czi` files. The pipeline will automatically discover all CZI files in this directory.

#### `params`
Command-line parameters passed to `bfconvert`. Common options include:

**Output Control:**
- `-overwrite` — Overwrite existing output files
- `-nooverwrite` — Skip conversion if output exists
- `-compression LZW` — Apply LZW compression (also: JPEG, JPEG-2000, Uncompressed)
- `-bigtiff` — Force BigTiff format for large files

**Series & Dimension Selection:**
- `-series N` — Convert only series N from multi-series files
- `-timepoint N` — Extract specific timepoint
- `-channel N` — Select specific channel
- `-z N` — Extract specific Z-section
- `-range START END` — Convert images between indices (inclusive)

**Tiling & Cropping:**
- `-tilex 512 -tiley 512` — Set tile dimensions
- `-crop X,Y,WIDTH,HEIGHT` — Extract rectangular region

**Advanced Options:**
- `-option KEY VALUE` — Pass format-specific writer options
- `-padded` — Zero-pad filename indices in output patterns

For complete parameter documentation, see: https://docs.openmicroscopy.org/bio-formats/5.7.1/users/comlinetools/conversion.html

#### `resources`
- `mem_mb`: Memory allocation per job (default: 64000 MB = 64 GB)
- `threads`: Number of CPU threads per conversion (default: 1)
- `runtime`: Maximum runtime per job in minutes (default: 30)

Adjust these based on your image sizes and available resources.

## Usage

### Basic Execution

Run the pipeline with default settings:
```bash
snakemake --sdm conda --cores 4
```

### Dry Run

Preview what the pipeline will do without executing:
```bash
snakemake --sdm conda --cores 4 -n
```

### Parallel Processing

Process multiple files in parallel (adjust based on available resources):
```bash
snakemake --sdm conda --cores 8
```

### With Cluster Execution

For HPC environments, create a cluster profile or use cluster execution:
```bash
snakemake --sdm conda --executor slurm --jobs 10
```

## Output

Converted TIFF files are saved to the `results/` directory with the same base filename:
```
input_dir/
  ├── sample1.czi
  └── sample2.czi

results/
  ├── sample1.tiff
  └── sample2.tiff
```

Conversion logs are stored in `logs/` directory:
```
logs/
  ├── sample1.log
  └── sample2.log
```

## Pipeline Structure

```
czi-convert/
├── config/
│   ├── config.yaml.example    # Example configuration
│   └── config.yaml            # Your configuration (create from example)
├── workflow/
│   ├── Snakefile              # Main pipeline definition
│   └── envs/
│       └── bftools.yaml       # Conda environment specification
├── results/                   # Output TIFF files (created on run)
├── logs/                      # Conversion logs (created on run)
└── README.md                  # This file
```

## Troubleshooting

### Memory Issues

If conversions fail due to memory errors, increase `mem_mb` in `config/config.yaml`:
```yaml
resources:
  mem_mb: 128000  # Increase to 128 GB
```

### Large File Handling

For very large images (>4GB), ensure BigTiff output:
```yaml
params: "-overwrite -compression LZW -bigtiff"
```

### Checking Conversion Logs

If a conversion fails, check the corresponding log file:
```bash
cat logs/sample_name.log
```

### Conda Environment Issues

If the bftools environment fails to create, try manually creating it:
```bash
conda env create -f workflow/envs/bftools.yaml
```

## Advanced Usage

### Converting Specific Series

To convert only the first series from multi-series CZI files:
```yaml
params: "-overwrite -compression LZW -series 0"
```

### Custom Output Formats

While this pipeline outputs TIFF by default, you can modify `workflow/Snakefile` to output other formats supported by Bio-Formats (OME-TIFF, PNG, etc.) by changing the output file extension.

### Tile-based Processing

For extremely large images, consider tile-based output:
```yaml
params: "-overwrite -tilex 1024 -tiley 1024"
```

## References

- **Bio-Formats Documentation**: https://docs.openmicroscopy.org/bio-formats/
- **bfconvert Command Reference**: https://docs.openmicroscopy.org/bio-formats/5.7.1/users/comlinetools/conversion.html
- **Snakemake Documentation**: https://snakemake.readthedocs.io/

## Support

For issues or questions:
1. Check the logs in `logs/` directory
2. Review Bio-Formats documentation for parameter options
3. Consult Snakemake documentation for workflow issues
