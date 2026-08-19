# Assembly Quality Control

This page describes the quality-control workflow used to evaluate haplotype-resolved genome assemblies. Each assembly haplotype was assessed using complementary metrics covering consensus accuracy, phasing, assembly statistics, gene completeness and duplication, interchromosomal misjoins, and read-supported assembly errors.

The QC workflow included:

- `yak qv`
- `yak trioeval` for trio samples
- QUAST
- `minimap2` + `paftools.js asmgene`
- `minigraph` + `paftools.js misjoin`
- Inspector

---

## 1. Yak QV

Consensus sequence accuracy was assessed independently for each haplotype using `yak qv`.

### Example

```bash
yak count \
    -k31 \
    -b37 \
    -t100 \
    -o \
    /ERGP/Pangenome/Assembly/02_yak/GDXX1.yak \
    /ERGP/Pangenome/Assembly/00_fastq/GDXX1.merged.fastq.gz
```

```bash
yak qv \
    -t 110 \
    -p \
    -K 3.2g \
    -l 100k \
    /ERGP/Pangenome/Assembly/02_yak/GDXX1.yak \
    /ERGP/Pangenome/fa_files_new/GDXX1.Hap1.fa \
    > /ERGP/Pangenome/Assembly/quality_checks/yak_qv/GDXX1.hap1.yak.qv.txt
```


## 2. Yak trioeval

For trio samples, haplotype phasing was additionally evaluated using `yak trioeval`.

### Example

```bash
yak trioeval \
    -t 110 \
    /ERGP/Pangenome/samples/00_preprocessing/yak_files/GDXX1_F.yak \
    /ERGP/Pangenome/samples/00_preprocessing/yak_files/GDXX1_M.yak \
    /ERGP/Pangenome/fa_files_new/GDXX1.Hap1.fa \
    > /ERGP/Pangenome/Assembly/quality_checks/yak_trioeval/GDXX1.hap1.yak.trioeval.txt
```

---

## 3. QUAST

General assembly statistics were calculated using QUAST.

Assemblies were evaluated against the **T2T-CHM13 v2.0** reference genome, with CHM13 gene annotations supplied to QUAST.

### Example

```bash
quast \
    /ERGP/Pangenome/fa_files_new/GDXX1.Hap1.fa \
    -r /_references/CHM13/chm13v2.0.fa \
    --features gene:/_references/CHM13/chm13.draft_v2.0.gene_annotation.gff3 \
    -o /ERGP/Pangenome/Assembly/quality_checks/quast/GDXX1.hap1.quast \
    --large \
    --est-ref-size 3100000000 \
    --no-icarus \
    -t 110 \
    &> /ERGP/Pangenome/Assembly/quality_checks/logs/GDXX1.hap1.quast.log
```


## 4. Gene completeness and duplication — asmgene

Gene representation and duplication were assessed using `minimap2` and the `asmgene` functionality of `paftools.js`.

Human cDNA sequences were first aligned against each assembled haplotype using `minimap2`.

### cDNA alignment

```bash
minimap2 \
    -cxsplice:hq \
    -t40 \
    /ERGP/Pangenome/fa_files_new/GDXX1.Hap1.fa \
    /_references/asmGene/Homo_sapiens.GRCh38.cdna.all.fa.gz \
    > /ERGP/Pangenome/Assembly/quality_checks/asmgene_paf/GDXX1.hap1.paf
```

The same procedure was performed for haplotype 2.

### Assessment against CHM13

```bash
paftools.js asmgene \
    /_references/asmGene/CHM13.ref.cdna.paf \
    /ERGP/Pangenome/Assembly/quality_checks/asmgene_paf/GDXX1.hap1.paf \
    > /ERGP/Pangenome/Assembly/quality_checks/asmgene_stat/GDXX1.hap1_CHM13.tsv
```

Autosome-restricted statistics were also generated:

```bash
paftools.js asmgene -a \
    /_references/asmGene/CHM13.ref.cdna.paf \
    /ERGP/Pangenome/Assembly/quality_checks/asmgene_paf/GDXX1.hap1.paf \
    > /ERGP/Pangenome/Assembly/quality_checks/asmgene_stat/GDXX1.hap1_CHM13_autosomes.tsv
```

### Assessment against GRCh38

```bash
paftools.js asmgene \
    /_references/asmGene/hg38.ref.cdna.paf \
    /ERGP/Pangenome/Assembly/quality_checks/asmgene_paf/GDXX1.hap1.paf \
    > /ERGP/Pangenome/Assembly/quality_checks/asmgene_stat/GDXX1.hap1_hg38.tsv
```

Autosome-restricted statistics were generated using:

```bash
paftools.js asmgene -a \
    /_references/asmGene/hg38.ref.cdna.paf \
    /ERGP/Pangenome/Assembly/quality_checks/asmgene_paf/GDXX1.hap1.paf \
    > /ERGP/Pangenome/Assembly/quality_checks/asmgene_stat/GDXX1.hap1_hg38_autosomes.tsv
```

Both haplotypes were evaluated against **CHM13** and **GRCh38**.

---

## 5. Interchromosomal misjoin assessment

Potential interchromosomal assembly misjoins were evaluated by aligning each haplotype against both **GRCh38** and **T2T-CHM13 v2.0** using `minigraph`.

### Alignment against GRCh38

```bash
minigraph \
    -cxasm \
    -t 60 \
    /_references/hg38/hg38.fa \
    /ERGP/Pangenome/fa_files_new/GDXX1.Hap1.fa \
    > /ERGP/Pangenome/Assembly/quality_checks/misjoin_paf/GDXX1.hap1_hg38_misjoin.paf
```

### Alignment against CHM13

```bash
minigraph \
    -cxasm \
    -t 60 \
    /_references/CHM13/chm13v2.0.fa \
    /ERGP/Pangenome/fa_files_new/GDXX1.Hap1.fa \
    > /ERGP/Pangenome/Assembly/quality_checks/misjoin_paf/GDXX1.hap1_CHM_misjoin.paf
```

Potential misjoins were summarized using `paftools.js misjoin`.

```bash
paftools.js misjoin \
    /ERGP/Pangenome/Assembly/quality_checks/misjoin_paf/GDXX1.hap1_hg38_misjoin.paf \
    > /ERGP/Pangenome/Assembly/quality_checks/misjoin_stat/GDXX1.hap1_hg38_misjoin.tsv
```

### Centromere-aware misjoin assessment

Detailed misjoin tables were generated using the corresponding centromere annotations.

For GRCh38:

```bash
paftools.js misjoin \
    -e \
    -c /_references/centromeres/centromeres_hg38.bed \
    /ERGP/Pangenome/Assembly/quality_checks/misjoin_paf/GDXX1.hap1_hg38_misjoin.paf \
    > /ERGP/Pangenome/Assembly/quality_checks/misjoin_table/GDXX1.hap1_hg38_misjoin.tsv
```

For CHM13:

```bash
paftools.js misjoin \
    -e \
    -c /_references/centromeres/chm13v2.0_centromeres.bed \
    /ERGP/Pangenome/Assembly/quality_checks/misjoin_paf/GDXX1.hap1_CHM_misjoin.paf \
    > /ERGP/Pangenome/Assembly/quality_checks/misjoin_table/GDXX1.hap1_CHM_misjoin.tsv
```

The analysis was performed independently for both haplotypes.

---

## 6. Inspector

Assembly quality was further evaluated using **Inspector**

### Haplotype 1

```bash
inspector.py \
    -c /ERGP/Pangenome/fa_files_new/GDXX1.Hap1.fa \
    -r /ERGP/Pangenome/samples/00_preprocessing/02_merged/F113_PGP0070868_C.merged.fastq.gz \
    --ref /_references/CHM13/chm13v2.0.fa \
    -o /ERGP/Pangenome/Assembly/quality_checks/inspector_I/GDXX1.hap1 \
    --datatype hifi \
    -t 110 \
    &> /ERGP/Pangenome/Assembly/quality_checks/logs/GDXX1.hap1.inspector.log
```

### Haplotype 2

```bash
inspector.py \
    -c /ERGP/Pangenome/fa_files_new/GDXX1.Hap2.fa \
    -r /ERGP/Pangenome/samples/00_preprocessing/02_merged/F113_PGP0070868_C.merged.fastq.gz \
    --ref /_references/CHM13/chm13v2.0.fa \
    -o /ERGP/Pangenome/Assembly/quality_checks/inspector_I/GDXX1.hap2 \
    --datatype hifi \
    -t 110 \
    &> /ERGP/Pangenome/Assembly/quality_checks/logs/GDXX1.hap2.inspector.log
```



