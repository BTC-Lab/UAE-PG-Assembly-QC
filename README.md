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

## 1. Consensus Quality Value (QV) — yak

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
    > /ERGP/Pangenome/Assembly/quality_checks_after/yak_qv/GDXX1.hap1.yak.qv.txt
```


## 2. Trio-based phasing assessment — yak trioeval

For trio samples, haplotype phasing was additionally evaluated using `yak trioeval`.

### Example

```bash
yak trioeval \
    -t 110 \
    /ERGP/Pangenome/G42_samples/00_preprocessing/yak_files/GDXX1_F.yak \
    /ERGP/Pangenome/G42_samples/00_preprocessing/yak_files/GDXX1_M.yak \
    /ERGP/Pangenome/fa_files_new/GDXX1.Hap1.fa \
    > /ERGP/Pangenome/Assembly/quality_checks_after/yak_trioeval/GDXX1.hap1.yak.trioeval.txt
```

---

## 3. Assembly statistics — QUAST

General assembly statistics were calculated using QUAST.

Assemblies were evaluated against the **T2T-CHM13 v2.0** reference genome, with CHM13 gene annotations supplied to QUAST.

### Example

```bash
quast \
    /ERGP/Pangenome/fa_files_new/GDXX1.Hap1.fa \
    -r /_references/CHM13/chm13v2.0.fa \
    --features gene:/_references/CHM13/chm13.draft_v2.0.gene_annotation.gff3 \
    -o /ERGP/Pangenome/Assembly/quality_checks_after/quast/GDXX1.hap1.quast \
    --large \
    --est-ref-size 3100000000 \
    --no-icarus \
    -t 110 \
    &> /ERGP/Pangenome/Assembly/quality_checks_after/logs/GDXX1.hap1.quast.log
```


## 4. Gene completeness and duplication — asmgene

Gene representation and duplication were assessed using `minimap2` and the `asmgene` functionality of `paftools.js`.

Human cDNA sequences were first aligned against each assembled haplotype using `minimap2`.

### cDNA alignment

```bash
minimap2 \
    -cxsplice:hq \
    -t40 \
    /media/NAS/ERGP/Pangenome/fa_files_new/GDXX1.Hap1.fa \
    /media/NAS/_references/asmGene/Homo_sapiens.GRCh38.cdna.all.fa.gz \
    > /media/NAS/ERGP/Pangenome/Assembly/quality_checks_after/asmgene_paf/GDXX1.hap1.paf
```

The same procedure was performed for haplotype 2.

### Assessment against CHM13

```bash
paftools.js asmgene \
    /media/NAS/_references/asmGene/CHM13.ref.cdna.paf \
    /media/NAS/ERGP/Pangenome/Assembly/quality_checks_after/asmgene_paf/GDXX1.hap1.paf \
    > /media/NAS/ERGP/Pangenome/Assembly/quality_checks_after/asmgene_stat/GDXX1.hap1_CHM13.tsv
```

Autosome-restricted statistics were also generated:

```bash
paftools.js asmgene -a \
    /media/NAS/_references/asmGene/CHM13.ref.cdna.paf \
    /media/NAS/ERGP/Pangenome/Assembly/quality_checks_after/asmgene_paf/GDXX1.hap1.paf \
    > /media/NAS/ERGP/Pangenome/Assembly/quality_checks_after/asmgene_stat/GDXX1.hap1_CHM13_autosomes.tsv
```


