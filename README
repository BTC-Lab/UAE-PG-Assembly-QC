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

The sample-specific yak k-mer database generated from the sequencing reads was compared against the corresponding assembled haplotype.

### Example

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

Logs were written to:

```text
/ERGP/Pangenome/Assembly/quality_checks_after/logs/yak.qv.logs
```
