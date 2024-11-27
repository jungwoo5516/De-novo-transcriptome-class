# RNA-seq: De novo transcriptome

E-mail: [jungwoo5516@gmail.com](mailto:jungwoo5516@gmail.com)

![image.png](RNA-seq%20De%20novo%20transcriptome%20bfa6ff89384c4cf19251ed4d391edb91/image.png)

# Install

---

## 1. Bioconda setting

```bash
# conda에 bioconda 채널 추가
conda config --add channels bioconda
conda config --add channels conda-forge
conda config --set channel_priority strict
```

**Bioconda를 사용하는 이유**

- Trimmomatic, Trinity 등 Bioinformatics에서 사용하는 tool들을 가장 간단한 방법으로 설치할 수 있다.

## 2. Setting tools

- Tool들의 역할
    1. **Trimmomatic**
    - read들의 adapter 제거
    - quality가 안좋은 nucleotide 제거
    2. **FastQC**
    - read들의 quality를 평가
    3. **Trinity**
    - Read들을 assemble해서 transcript를 제작
    4. **Transdecoder**
    - Transcript들의 ORF를 예측하여 CDS locus를 추정
    5. **CD-hit**
    - 중복되는 Transcripts를 제거
    6. **BLAST**
    - Transcript들의 기능을 파악하기 위해 유사한 서열 검색

```bash
# 설치
conda install trimmomatic
# 설치확인
trimmomatic

conda install fastqc
fastqc -h

conda install trinity
Trinity

conda install transdecoder
TransDecoder.Predict

conda install rsem
rsem-simulate-reads

conda install cd-hit
cd-hit

conda install blast
blastp -h
```

- Error가 발생했을 경우
    
    ```bash
    conda install -c conda-forge conda-libmamba-solver
    
    # 위 코드를 실행 후 다시 conda install를 진행
    
    warning  libmamba Problem type not implemented SOLVER_RULE_STRICT_REPO_PRIORITY
    warning  libmamba Problem type not implemented SOLVER_RULE_STRICT_REPO_PRIORITY
    
    # 위 워링 메세지가 나와도 무시
    ```
    

```bash
# Conda에 설치된 패키지 확인
conda list | grep '찾고자하는 프로그램'

# blast 예시
conda list | grep 'blast'
```

## 3. Setting RNA-seq protocol

```bash
mkdir 01.RNA-seq
cd 01.RNA-seq

# 3개의 디렉토리를 동시에 생성
mkdir 1.Raw_data 2.Quality_control 3.Transcript_assemble
```

## 4. Raw data download

https://zenodo.org/records/3541678

- Duck의 liver force-feeding, control RNA-seq 데이터

```bash
cd 1.Raw_data

wget https://zenodo.org/record/3541678/files/A1_left.fq.gz
wget https://zenodo.org/record/3541678/files/A1_right.fq.gz
wget https://zenodo.org/record/3541678/files/B1_left.fq.gz
wget https://zenodo.org/record/3541678/files/B1_right.fq.gz
```

```bash
# 01.RNA-seq 디렉토리로 이동
# RNA-seq 디렉토리의 구성파악

tree 
```

![tree 커맨드 결과](RNA-seq%20De%20novo%20transcriptome%20bfa6ff89384c4cf19251ed4d391edb91/image%201.png)

tree 커맨드 결과

# Raw data preprocessing

---

![image.png](RNA-seq%20De%20novo%20transcriptome%20bfa6ff89384c4cf19251ed4d391edb91/image%202.png)

## 1. Quality check

```bash
# FastQC를 통한 raw data의 퀄리티확인
# 1.Raw data 디렉토리 이동

fastqc A1_left.fq.gz A1_right.fq.gz B1_left.fq.gz B1_right.fq.gz
```

- **목적**
    
    Illumina sequencing을 통해서 얻은 read들의 품질을 평가 
    
- **분석방법**
    - Output으로 나온 A1_left_fastqc.html 파일을 scp로 이용하여 로컬 컴퓨터로 이동.
    - fastqc.html 확인 (예시 파일)
        
        [A1_left_fastqc.html](RNA-seq%20De%20novo%20transcriptome%20bfa6ff89384c4cf19251ed4d391edb91/A1_left_fastqc.html)
        
- **분석결과**
    
    

## 2. Raw data trimming

![image.png](RNA-seq%20De%20novo%20transcriptome%20bfa6ff89384c4cf19251ed4d391edb91/image%203.png)

```bash
# Trimmomatic을 통한 low quality base pair 및 adapter 제거
# 2.Raw data trimming 디렉토리 이동

trimmomatic PE -phred33 ../1.Raw_data/A1_left.fq.gz ../1.Raw_data/A1_right.fq.gz A1_forward_paired.fq A1_forward_unpaired.fq A1_reverse_paired.fq A1_reverse_unpaired.fq \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:50 2>&1 | tee A1.log

trimmomatic PE -phred33 ../1.Raw_data/B1_left.fq.gz ../1.Raw_data/B1_right.fq.gz B1_forward_paired.fq B1_forward_unpaired.fq B1_reverse_paired.fq B1_reverse_unpaired.fq \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:50 2>&1 | tee B1.log
```

- **목적**
    - Sequencing된 read들에는 adapter가 붙어있는 경우와 low quality base pair가 존재하는 경우가 있기 때문에 이를 제거
    - Quality control을 진행하지 않은 read를 사용하여 후속처리를 진행하면 최적의 결과 도출에 문제 발생 여지가 존재
- **분석방법**
    - Trimmomatic log파일에서 **surviving rate**를 확인
    
    ```bash
    TrimmomaticPE: Started with arguments:
     -phred33 ../01.raw_data/A1_left.fq.gz ../01.raw_data/A1_right.fq.gz A1_forward_paired.fq A1_forward_unpaired.fq A1_reverse_paired.fq A1_r
    everse_unpaired.fq LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:50
    Multiple cores found: Using 4 threads
    Input Read Pairs: 10000 Both Surviving: 9189 (91.89%) Forward Only Surviving: 366 (3.66%) Reverse Only Surviving: 339 (3.39%) Dropped: 106
     (1.06%)
    TrimmomaticPE: Completed successfully
    ```
    
- **분석결과**
    
    

# Transcript assemble

---

![image.png](RNA-seq%20De%20novo%20transcriptome%20bfa6ff89384c4cf19251ed4d391edb91/image%204.png)

```bash
# Trinity를 통해 read들을 조합하여 transcripts 제작
# 3.Transcript_assemble 디렉토리 이동

Trinity --seqType fq --left ../2.Quality_control/A1_forward_paired.fq --right ../2.Quality_control/A1_reverse_paired.fq --max_memory 10G --output A1_trinity.fasta --full_cleanup 2>&1 | tee A1_trinity.log
Trinity --seqType fq --left ../2.Quality_control/B1_forward_paired.fq --right ../2.Quality_control/B1_reverse_paired.fq --max_memory 10G --output B1_trinity.fasta --full_cleanup 2>&1 | tee B1_trinity.log
```

- **목적**
    - Read들을 대상으로 transripts를 조립
- **분석방법**
    - transcript assemble protocol ([https://www.nature.com/articles/nbt.1883](https://www.nature.com/articles/nbt.1883))
        
        ![image.png](RNA-seq%20De%20novo%20transcriptome%20bfa6ff89384c4cf19251ed4d391edb91/image%205.png)
        
    - 생성된 Trinity.fasta 파일을 후속분석에 사용하여 유전자의 발현패턴 등을 확인한다.
        
        ![image.png](RNA-seq%20De%20novo%20transcriptome%20bfa6ff89384c4cf19251ed4d391edb91/image%206.png)
        

## Path

---

![image.png](RNA-seq%20De%20novo%20transcriptome%20bfa6ff89384c4cf19251ed4d391edb91/image%207.png)

```bash
2.Quality_control을 기준으로
	# 상대경로
	1)A1_left.fq.gz
	 : ../A1_left.fq.gz
	
	# 절대경로
	2)A1_left.fq.gz
   : /home/guest_181599/RNA-seq_2/01.RNA-seq/1.Raw_data/A1_left.fq.gz
```

# Gene predict

---

![image.png](RNA-seq%20De%20novo%20transcriptome%20bfa6ff89384c4cf19251ed4d391edb91/image%208.png)

## 0. Make gene predict directory

```bash
mkdir 4.Gene_prdict

# 4.Gene_preidct 안에서 디렉토리 생성
mkdir 1.Transdecoder.LongOrf
mkdir 2.homology_search
mkdir 3.Transdecoder.Predict
```

## 1. Find long ORF region in transcripts

![image.png](RNA-seq%20De%20novo%20transcriptome%20bfa6ff89384c4cf19251ed4d391edb91/image%209.png)

### Protein homology search

- 패스설명
    
    ![image.png](RNA-seq%20De%20novo%20transcriptome%20bfa6ff89384c4cf19251ed4d391edb91/image%2010.png)
    

```bash
# TransDecoder.Predict를 활용하여 transcripts에서 protein을 coding하는 region 탐색 
# Gene predict 디렉토리 이동 후 1.Transdecoder.LongOrf 디렉토리 이동
# 
TransDecoder.LongOrfs -t ../../3.Transcript_assemble/A1_trinity.fasta.Trinity.fasta
TransDecoder.LongOrfs -t ../../3.Transcript_assemble/B1_trinity.fasta.Trinity.fasta
```

- **목적**
    
    조립된 transcript에서 단백질이 코딩된 부분을 찾는 과정
    
- **분석방법**
    
    

```bash
# 생성된 transcript에서 gene predict을 할때 사용할 참고데이터를 만들기 위한 과정
# 2.homology_search 디렉토리에서 실행

blastp -query ../1.TransDecorder_LongOrfs/A1_trinity.fasta.Trinity.fasta.transdecoder_dir/longest_orfs.pep -db /home/guest_119/uniprot_DB/uniprot_sprot.fasta -outfmt 6 -evalue 1e-5 > A1_blast.out
blastp -query ../1.TransDecorder_LongOrfs/B1_trinity.fasta.Trinity.fasta.transdecoder_dir/longest_orfs.pep -db /home/guest_119/uniprot_DB/uniprot_sprot.fasta -outfmt 6 -evalue 1e-5 > B1_blast.out
```

- **목적**
    
    Trasndecoder.Predict를 해서 coding region으로 추측되는 부분을 찾을 때 blast에 대응되는 부분은 반드시 남기기 위해서 사전 데이터 생성
    

# 첫번째 실습 마무리

---

## 2. Gene predict

```bash
# TransDecoder.Predict를 활용하여 실제 

TransDecoder.Predict -t ../../3.Transrcipt_assemble/A1_trinity.fasta.Trinity.fasta --retain_blastp_hits ../2.homolgy_search/A1_blast.out --single_best_only
TransDecoder.Predict -t ../../3.Transrcipt_assemble/B1_trinity.fasta.Trinity.fasta --retain_blastp_hits ../2.homolgy_search/B1_blast.out --single_best_only
```

## 3. Remove redundant transcripts

```bash
# Cd-hit을 통해서

cd-hit -i ../3.TransDecorder_Predict/A1_trinity.fasta.Trinity.fasta.transdecoder.pep -c 0.99 -n 5 -o A1.Trinity.transdecoder.cdhit.pep
cd-hit -i ../3.TransDecorder_Predict/B1_trinity.fasta.Trinity.fasta.transdecoder.pep -c 0.99 -n 5 -o B1.Trinity.transdecoder.cdhit.pep
```

# Annotation

---

```bash
# blast

blastp -query ../4.Gene_predict/4.Remove_redudant_transripts/A1.Trinity.transdecoder.cdhit.pep -db ../4.Gene_predict/2.homolgy_search/uniprot_DB/uniprot_sprot.fasta -out A1_blast.out -outfmt 6 -evalue 1e-10
```
