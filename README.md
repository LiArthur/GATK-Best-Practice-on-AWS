
## 一、方案简介

*** 相关基本介绍请参考[Parallelcluster官方博客.](https://amazonaws-china.com/cn/blogs/china/aws-parallelcluster/) ***

### 1、现状及方案概述
当前基因行业的分析平台大致分为三类，分别是单机、HPC集群、K8S集群，占比最的的仍然是HPC集群这种形式，对于大部分公司来说，本地HPC的搭建、运维成本仍然很昂贵，并且存在着诸多问题：

+ 计算存储的扩容、缩容周期长，难以跟上业务发展的速度
+ 多套集群运维、部署、购置成本较高，难以做到备份容灾
+ 本地数据类型复杂，大量文件来源和价值不能确认，难以做到精确的备份与归档
+ 开发人员被各种集群问题所困扰，运维人员的精力被日常清理工作所消耗，难以集中做集群的优化工作。

上述问题在大部分公司都是存在的，除此之外还有很多问题会困扰大家。
为了帮助克服、解决这些问题，我们提供了一套简单易用的方案，可一键创建完整的HPC集群，除此之外通过参数配置的调整，还可以以用户习惯的架构来精细化调整HPC环境，通过模版的复制，也可以最小化的成本实现本地的测试、迁移以及云上的集群环境复制，以此来帮助生命科学领域的用户去构建安全、可靠、高效、低成本的HPC集群，将开发、运维人员的精力从琐事中解放，专注在更有创造力的事情上，借助云的优势，可以打造更灵活、高可用的系统架构。


***

### 2、解决的需求和问题
#### 1)、生产与测试系统未隔离，各产品线共用一套集群
#### 2)、本地集群无容灾，一旦受损（宕机、环境变动）、停止（如物业计划维护、断网断电等）将影响所有产品业务
![image](http://github.com/lab798/GATK-Best-Practice-on-AWS/raw/master/images/1.png)

#### 3)、计算存储资源量估算困难，部署周期长，业务线变动较大
![image](http://github.com/lab798/GATK-Best-Practice-on-AWS/raw/master/images/2.png)

#### 4)、开发迭代混乱，集群环境大家共同维护，难以做到版本控制和环境复现!
![image](http://github.com/lab798/GATK-Best-Practice-on-AWS/raw/master/images/3.png)

#### 5)、行业其他问题：
##### ①、项目管理靠人工跟踪，无信息化系统或信息化程度低
##### ②、信息环节无法做到成本核算
##### ③、数据量大且数据复杂，难以做到生命周期管理

***

## 二、详细方案说明
### 1、集群解决方案
![集群解决方案](http://github.com/lab798/GATK-Best-Practice-on-AWS/raw/master/images/4.png)
### 2、端到端的基因大数据分析、归档、交付方案
![端到端的基因大数据分析、归档、交付方案](http://github.com/lab798/GATK-Best-Practice-on-AWS/raw/master/images/5.png)
## 三、方案部署及测试文档
### 1、10分钟集群部署
下述文档示例会启动一个完整的HPC集群，包括主节点、计算节点、共享存储以及预装SGE作业调度系统，AMI为预装GATK相关软件的镜像，包括bwa，Samtools，gatk4等，镜像snapshot为GATK公开数据集，包括数据库及测试文件，启动后挂载到/genomics目录下。
*** 注：测试以北京区为例 ***
#### 1)、 安装pip及awscli并配置必要信息
    #pip安装
    curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
    python get-pip.py
    
    #awscli安装
    sudo pip install awscli
    
    #aws配置
    #根据具体情况设置AK,SK，所在区域及输出格式
    aws configure 
    
#### 2)、安装pcluster
    sudo pip install aws-parallelcluster

#### 3)、pcluster安装及配置
##### ①、相关准备
+ aws_access_key_id及aws_secret_access_key
** 请登录console，并点击 *我的安全凭证* **
** 创建访问密钥并记录aws_access_key_id及aws_secret_access_key **
<br>
![](http://github.com/lab798/GATK-Best-Practice-on-AWS/raw/master/images/6.png)
<br>
![](http://github.com/lab798/GATK-Best-Practice-on-AWS/raw/master/images/7.png)
<br>
<br>
+ VPC及子网
**请搜索VPC服务，并选择你要启动集群的VPC**
**记录vpc_id，master_subnet_id**
<br>
![](http://github.com/lab798/GATK-Best-Practice-on-AWS/raw/master/images/8.png)
<br>
![](http://github.com/lab798/GATK-Best-Practice-on-AWS/raw/master/images/9.png)
<br>
![](http://github.com/lab798/GATK-Best-Practice-on-AWS/raw/master/images/10.png)
<br>
<br>
+ EC2访问密钥
** 搜索EC2服务，并进入EC2服务页面选择密钥对，创建新密钥并下载密钥文件 **
** 记录key_name为创建密钥对输入的名字 **
<br>
![](http://github.com/lab798/GATK-Best-Practice-on-AWS/raw/master/images/11.png)

##### ②、配置pcluster config(可参考官方博客)
***注：标注为 XXXXXXXXXX 的请修改为上面获取到的值***

    #创建配置模版,会提示无配置，请忽略错误信息
    pcluster create new
    
    #编辑配置文档
    vim ~/.parallelcluster/config

    #复制下述配置信息，粘贴到配置文档~/.parallelcluster/config
    [aws]
    aws_region_name = cn-northwest-1
    aws_access_key_id = AKIATIM7AOQ7IHJCM3 //需要修改
    aws_secret_access_key = ih4RN0rES+ytUy67Q377/RGfxwAiZqpWhCJAKA  //需要修改
    
    [vpc public]
    vpc_id = vpc-a817aac5  //需要修改
    master_subnet_id = subnet-26fcc86cd  //需要修改
    
    [cluster GATK-pipeline]
    base_os = alinux
    custom_ami = ami-09fe399cffad04f67
    vpc_settings = public
    scheduler = slurm
    key_name = NX_key  //需要修改
    compute_instance_type = m4.xlarge
    master_instance_type = m4.xlarge
    compute_root_volume_size = 50
    master_root_volume_size = 50
    ebs_settings = genomes
    s3_read_resource = arn:aws-cn:s3:::parallelcluster-gatk/*
    scaling_settings = GATK-ASG
    initial_queue_size = 1
    pre_install = s3://parallelcluster-gatk/00.ParallelCluster/preinstall.sh 
    max_queue_size = 4
    maintain_initial_size = false
    extra_json = { "cluster" : { "ganglia_enabled" : "yes" ,"cfn_scheduler_slots" : "cores" } }
    
    [scaling GATK-ASG]
    scaledown_idletime = 5
    
    [ebs genomes]
    shared_dir = genomes
    ebs_snapshot_id = snap-040c71fd2bb5d4236
    volume_type = gp2
    volume_size =  1024
    
    [global]
    update_check = true
    sanity_check = true
    cluster_template = GATK-pipeline
    
    [aliases]
    ssh = ssh {CFN_USER}@{MASTER_IP} {ARGS}
    
#### 4)、启动集群
    pcluster create GATK-pipeline
    
#### 5)、登陆集群master节点
    #根据集群启动后的反馈信息输入
    ssh -i <private key_name> ec2-user@master-public-ip
    
#### 6)、投递任务
默认预装SGE作业调度系统，所以可直接qsub投递计算任务，举例如下：
    
    echo "sleep 180" | qsub
    echo "sh run.sh" | qsub -l vf=2G,s_core=1 -q all.q
    for((i=1;i<=10;i++));do echo "sh /genomes/temp/run.sh $i" | qsub -cwd -S /bin/bash -l vf=2G,s_core=1 -q all.q;done
    
示例slurm调度系统投递命令参考如下：

    sbatch -n 4 run.sh  //4核，可根据需要修改
    squeue //查看队列情况
    sinfo //查看节点情况
    scancel jobid //取消任务
    

### 2、基于workflow工具调度的WES分析
补充材料，可自行测试
#### 1)、准备条件
##### ①、HPC集群
##### ②、nextflow调度软件
##### ③、脚本及配置文件
#### 2)、脚本文件
保存以下代码为指定文件名，需要与后续运行命令相匹配。
FileName：***custom.conf***
```
process{
    executor="sge"
    clusterOptions='-S /bin/bash'
}
aws {
  accessKey = 'xxx'
  secretKey = 'xxx'
  region = 'cn-north-1'
}
```

FileName：***genome.nf***:

```
#!/usr/bin/env nextflow
/* wget -qO- https://get.nextflow.io | bash*/

VERSION="0.2"

log.info "===================================================================="
log.info "GATK4 Best Practice Nextflow Pipeline (v${VERSION})                        "
log.info "===================================================================="

params.help = ""
if (params.help) {
  log.info " "
  log.info "USAGE: "
  log.info " "
  log.info "nextflow run oliverSI/GATK4_Best_Practice --fastq1 read_R1.fastq.gz --fastq2 read_R2.fastq.gz"
  log.info " "
  log.info "Mandatory arguments:"
  log.info "    --fastq1        FILE               Fastq(.gz) file for read1"
  log.info "    --fastq2        FILE               Fastq(.gz) file for read2"
  log.info " "
  log.info "Optional arguments:"
  log.info "    --outdir        DIR                Output directory(default: ./Results)"
  log.info "    --samplename    STRING             Sample name(dafault: fastq1 basename)"
  log.info "    --rg            STRING             Read group tag(dafault: fastq1 basename)"
  log.info " "
  log.info "===================================================================="
  exit 1
}


fastq1 = file("$params.fastq1")
fastq2 = file("$params.fastq2")
params.outdir = "./Results"
params.samplename = fastq1.baseName
params.rg = fastq1.baseName
reference = file("/genomes/reference/hg19/v0/Homo_sapiens_assembly19.fasta")
dbsnp = file("/genomes/reference/hg19/v0/Homo_sapiens_assembly19.dbsnp138.vcf")
golden_indel = file("/genomes/reference/hg19/v0/Homo_sapiens_assembly19.known_indels_20120518.vcf")
hapmap = file("/genomes/reference/hg19/v0/hapmap_3.3.b37.vcf.gz")
omni = file("/genomes/reference/hg19/v0/1000G_omni2.5.b37.vcf.gz")

process BWA {
    publishDir "${params.outdir}/MappedRead"

    output:
    file 'aln-pe.sam' into samfile
    
    """
    bwa mem -M -t 8 -R '@RG\\tID:${params.rg}\\tSM:${params.samplename}\\tPL:Illumina' $reference $fastq1 $fastq2 > aln-pe.sam
    """
        
}

process BWA_sort {
    publishDir "${params.outdir}/MappedRead"
    
    input:
    file samfile

    output:
    file 'aln-pe-sorted.bam' into bam_sort

    """
    samtools sort -@ 8 -o aln-pe-sorted.bam -O BAM $samfile
    """

}

process MarkDuplicates {
    publishDir "${params.outdir}/MappedRead"
    
    input:
    file bam_sort

    output:
    file 'aln-pe_MarkDup.bam' into bam_markdup

    """
    /Pipeline/01.software/gatk/build/install/gatk/bin/gatk MarkDuplicates -I $bam_sort -M metrics.txt -O aln-pe_MarkDup.bam    
    """

}

process BaseRecalibrator {
    publishDir "${params.outdir}/BaseRecalibrator"
    
    input:
    file bam_markdup

    output:
    file 'recal_data.table' into BaseRecalibrator_table

    """
    /Pipeline/01.software/gatk/build/install/gatk/bin/gatk BaseRecalibrator \
    -I $bam_markdup \
    --known-sites $dbsnp \
    --known-sites $golden_indel \
    -O recal_data.table \
    -R $reference
    """
}

process ApplyBQSR {
    publishDir "${params.outdir}/BaseRecalibrator"
    
    input:
    file BaseRecalibrator_table
    file bam_markdup

    output:
    file 'aln-pe_bqsr.bam' into bam_bqsr
    
    script:
    """
    /Pipeline/01.software/gatk/build/install/gatk/bin/gatk ApplyBQSR -I $bam_markdup -bqsr $BaseRecalibrator_table -O aln-pe_bqsr.bam
    """
}

process HaplotypeCaller {
    publishDir "${params.outdir}/HaplotypeCaller"
    
    input:
    file bam_bqsr

    output:
    file 'haplotypecaller.g.vcf' into haplotypecaller_gvcf
    
    script:
    """
    /Pipeline/01.software/gatk/build/install/gatk/bin/gatk HaplotypeCaller -I $bam_bqsr -O haplotypecaller.g.vcf --emit-ref-confidence GVCF -R $reference
    """
}

process GenotypeGVCFs {
    publishDir "${params.outdir}/HaplotypeCaller"
    
    input:
    file haplotypecaller_gvcf

    output:
    file 'haplotypecaller.vcf' into haplotypecaller_vcf
    
    script:
    """
    /Pipeline/01.software/gatk/build/install/gatk/bin/gatk GenotypeGVCFs  --dbsnp $dbsnp --variant haplotypecaller.g.vcf -R $reference -O haplotypecaller.vcf
    """
}

process finish {
    publishDir "${params.outdir}/Report", mode: 'copy'

    input:
    file haplotypecaller_vcf

    output:
    file "${params.samplename}.vcf"
    
    """
    mv $haplotypecaller_vcf ${params.samplename}.vcf
    """

}
```
#### 3)、运行方法
```
#nextflow run -c ***[custom.conf]*** ***[nextflow script]*** --fastq1 ***[fastq1]*** --fastq2 ***[fastq2] -with-report xxx -with-timeline xxx -with-dag xxx ***-resume******
nextflow run -c custom.conf genome.nf --fastq1 /genomes/project/nf/SRR622461_1.fastq.gz --fastq2 /genomes/project/nf/SRR622461_1.fastq.gz ***-with-report xxx -with-timeline xxx -with-dag xxx ***-resume******
```



## 四、参考资料：
+ [Parallelcluster官方博客.](https://amazonaws-china.com/cn/blogs/china/aws-parallelcluster/)
+ [parallelcluster文档.](https://aws-parallelcluster.readthedocs.io/en/latest/)
+ [aws-parallelcluster GitHub 存储库.](https://github.com/aws/aws-parallelcluster)
+ 镜像版本迭代:

|系统	|版本号	|AMI ID	|更新描述	|地域	|是否公开	|可用性	|备注	|
|---	|---	|---	|---	|---	|---	|---	|---	|
|alinux-base	|	|ami-0da67c26ce2e8d111	|基础镜像	|BJS	|是	|是	|	|
|ubuntu-base	|16.04	|ami-0ae967dc97d5eb57a	|基础镜像	|BJS	|是	|是	|	|
|alinux	|0.1	|ami-07cb07d99e56894ce	|基础软件环境AMI	|BJS	|是	|是	|	|
|alinux	|0.2	|ami-0cad4e9d804bd9c15	|基础软件环境AMI + Golang环境 + goofys；修复pip问题并安装awscli；修复goofys无法挂载问题，安装fuse依赖	|BJS	|是	|是	|	|
|ubuntu	|0.1	|ami-097d3bf901991372e	|基础软件环境AMI	|BJS	|是	|是	|	|
|ubuntu	|0.2	|ami-041e4a3bce09385b9	|修改ubuntu默认shell(dash)为bash	|BJS	|是	|是	|不再更新	|
|ubuntu	|0.2-a	|ami-026882b56146cdc1b	|基础软件环境AMI + Golang环境 + goofys	|BJS	|是	|是	|	|
|alinux	|0.1	|ami-007f6ed61542ae017	|基础软件环境AMI	|NX	|是	|是	|	|
|ubuntu	|0.1	|ami-0a1d99c2c70e3f86c	|基础软件环境AMI	|NX	|是	|否	|	|
|ubuntu	|0.2	|ami-071aa7a2927cc02a8	|修改ubuntu默认shell(dash)为bash	|NX	|是	|否	|不再更新	|
|ubuntu	|0.2-a	|ami-015f3a018cc98b6cc	|基础软件环境AMI + Golang环境 + goofys	|NX	|是	|否	|	|


•   参考文件EBS快照迭代：

|名称	|版本号	|snap ID	|大小	|更新描述	|地域	|
|---	|---	|---	|---	|---	|---	|
|gatk-reference v0.1	|0.1	|snap-09c16ac9809cf4359	|100G	|基础环境快照，包含hg19参考基因组	|BJS	|
|gatk-reference v0.2	|0.2	|snap-06f5e874571e44510	|100G	|增加hg38及GATK数据集	|BJS	|
|gatk-reference-v0.3	|0.3	|snap-08a4b975a2f40736f	|1T	|增加测试文件及GATK-TEST-DATA	|BJS	|
|gatk-reference-v0.3	|0.3	|snap-040c71fd2bb5d4236	|1T	|增加测试文件及GATK-TEST-DATA	|NX	|
|	|	|	|	|	|	|


## 五、FAQ

