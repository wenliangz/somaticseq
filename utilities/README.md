<b>Scripts used for SEQC2 high-confidence mutation builder</b>

highConfidenceBuilder.py is used for SEQC2 project to build high-confidence mutation calls by assigning initial tier to each call.

$somaticseq/utilities/highConfidenceBuilder.py -infile sSNV.combined.draft.beta.3.vcf.gz   -outfile sSNV.dedup_all_draft.beta.3.vcf   -ncallers 3 -all
$somaticseq/utilities/highConfidenceBuilder.py -infile sINDEL.combined.draft.beta.3.vcf.gz -outfile sINDEL.dedup_all_draft.beta.3.vcf -ncallers 2 -all

Initial tiers are assigned in the following way:
1) The 63 pairs of BAM files are seperated by aligner and site/platforms. The aligners are BWA, Bowtie2, and NovoAlign. The site/platforms are NS (9x), IL (3x), NV (3x), FD (3x), and Others (3x, i.e., EA, NC, and LL). Thus, there are 12 aligner-site/platform combinations. With the exception of "Others," they all have SomaticSeq scores based on their own aligner-site/platform data.
2) For each aligner-site/platform, count the number of replicates that are classified by SomaticSeq as PASS, REJECT, or LowQual. Also count if it's Missing (not detected so ./. in sample column) or Consensus (i.e., at least 50% callers). For non-Others, each PASS is worth 1 point and each REJECT is -1 point. Everything else does not count. Add up the score as the combined score. For IL, NV, and FD, a combined score of 2 or 3 is deemed strongestEvidence. A combined score of 1 is deemed strongEvidence. A combined score between -1 and 0 is deemed lowQual. Everything below is considered insufficientEvidence. For NS (9x replicates), the 4 thresholds are i) 3 or above, ii) between 1 and 2, iii) between -2 and 0, and iv) -3 or below. For Others, consider Consensus as PASS and non-Consensus as REJECT, and combine scores accordingly. 
3) It's now time to consider aligner-centric classifications based on cross site/platform scores for each aligner. For each aligner-site/platform, strongestEvidence gets assigned +3, strongEvidence gets assigned +1, lowQual gets assigned 0, and insufficientEvidence gets assigned -3. Then, add up those scores for each of the 5 site/platforms. Range will be from -15 and +15. For each aligner-centric classification, we used the following thresholds, i) +6 or above for strongestEvidence, ii) between 2 and 5 is strongEvidence, iii) between -1 and +1 is lowQual, and iv) -2 or below is insufficientEvidence. 
4) Initial tier assignment based on 3 aligner-centric classifications, i.e., BWA, Bowtie2, and NovoAlign.
  i) Tier1: strongestEvidence for each of the 3 aligner-centric classifications. a) AllPASS: deemed PASS/Consensus by each of the 63 data sets, b) Tier1A: not only strongestEvidence for each of the 3 aligner-centric classifications, but also strongestEvidence for each of the 5 site/platforms within each aligner-centric classifications, c) Tier1B: for each of the 3 aligner-centric classifications (deemed strongestEvidence because Tier1), at least 3 site/platforms are deemed strongestEvidence as well, and d) Tier1C: minimally Tier1, i.e., some aligner-centric Classification may get here with less evidence than the others. 
  ii) Tier2: strongestEvidence for 2 out of 3 aligner-centric classifications. a) Tier2A: with the other one being merely "strongEvidence" instead of strongestEvidence. b) Tier2B: with the other one being lowQual. c) Tier2C: with the other one being insufficientEvidence. 
  iii) Tier3: strongestEvidence for just 1 out of 3 aligner-centric classifications. a) Tier3A: with the other two being merely "strongEvidence." b) Tier3B: 1 of the other 2 is strongEvidnece. c) Tier3C: less than 3A or 3B. 
  iv) Tier4: zero strongestEvidence classification for any of the 3 aligner-centric classifications. a) Tier4A: all 3 are merely "strongEvidence." b) Tier4B: 2 of 3 are strongEvience. c) Tier4C: 1 of the 3 is a strongEvidence. d) REJECT: what remains, i.e., none of the 3 aligner-centric classiifcation is even "strongEvidence."


Then, run somaticseq/utilities/highConfidenceBuilder_2ndPass.py to refine. 