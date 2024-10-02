# hse24_hw1
## Рябов Олег Владиславович

### Основная часть

Первым делом я зашёл на сервер, создал папку HW1 и создал ссылки на файлы чтений .fastq

```bash
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq

ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq

ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq

ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```



После этого я воспользовался утилитой seqtk чтобы выбрать 5 миллионов рандомных чтений типа paired-end и 1,5 миллиона чтений типа mate-pairs

```bash
seqtk sample -s722 oil_R1.fastq 5000000 > PE_R1.fq

seqtk sample -s722 oil_R2.fastq 5000000 > PE_R2.fq

seqtk sample -s722 oilMP_S4_L001_R1_001.fastq 1500000 > MP_R1.fq

seqtk sample -s722 oilMP_S4_L001_R2_001.fastq 1500000 > MP_R2.fq
```

Затем при помощи программы fastQC я выполнил анализ полученных файлов и сформировал HTML-отчёты 

```bash
mkdir report_PE_R1

fastqc PE_R1.fq -o report_PE_R1/
```

И так для каждого файла с чтениями. Затем при помощи multiQC сформировал единый отчёт

```bash
multiqc -o ./multiqc_report/ ./
```

Затем при помощи утилиты **platanus_trim** я подрезал paired-end чтения

```bash
platanus_trim PE_R1.fq PE_R2.fq
```

Затем при помощи утилиты **platanus_internal_trim** я подрезал mate-pairs чтения

```bash
platanus_internal_trim MP_R1.fq MP_R2.fq
```

Затем при помощи программы fastQC я выполнил анализ полученных файлов и сформировал HTML-отчёты,
а затем при помощи multiQC сформировал единый отчёт (команды аналогичны предыдущим)

Если сравнить оба отчёта, то можно увидеть, что подрезание чтений уменьшило процент дупликаций и улучшело качество(score).
А так же четко видно, что значительно снизилось количество адапторов

Далее при помощи команды 
<code>platanus assemble -o PE -f PE_R1.fq.trimmed PE_R2.fq.trimmed</code>
я собрал контиги из подрезанных чтений.
В ноутбуке приведён код который по данному файлу считает необходимую статистику 
(общее кол-во контигов, их общая длина, длина самого длинного контига, N50).

Далее при помощи **platanus scaffold** я собрал скаффолды из контигов и подрезанных чтений

```bash
platanus scaffold -c PE/PE_contig.fa -IP1 PE/PE_R1.fq.trimmed PE/PE_R2.fq.trimmed -OP2 MP/MP_R1.fq.int_trimmed MP/MP_R2.fq.int_trimmed 
```

В ноутбуке приведён код который считает количество пропусков в скаффолдах и их общую длину

Затем я при помощи **platanus gap_close** уменьшил количество гэпов

```bash
platanus gap_close -c out_scaffold.fa -IP1 PE/PE_R1.fq.trimmed PE/PE_R2.fq.trimmed -OP2 -OP2 MP/MP_R1.fq.int_trimmed MP/MP_R2.fq.int_trimmed 
```

Как мы видим число и общая длина гэпов значительно уменьшились

После этого я закинул все необходимые файлы в одну папку и командой 
<code>scp -r ovryabov@172.21.136.17:hw1/files ./</code>
перенёс эти файлы в локальный репозиторий

Ну а потом на гитхаб всё запушил


### Бонусная часть

