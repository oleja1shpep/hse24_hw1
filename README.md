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
(все скриншоты в папках ./data/muliqc_report и ./data/muliqc_report_trimmed)

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
platanus gap_close -c out_scaffold.fa -IP1 PE/PE_R1.fq.trimmed PE/PE_R2.fq.trimmed -OP2 MP/MP_R1.fq.int_trimmed MP/MP_R2.fq.int_trimmed 
```

Как мы видим число и общая длина гэпов значительно уменьшились

После этого я закинул все необходимые файлы в одну папку и командой 
<code>scp -r ovryabov@172.21.136.17:hw1/files ./</code>
перенёс эти файлы в локальный репозиторий

Ну а потом на гитхаб всё запушил

# Вывод

Количество контигов низкое, а N50 - высокий, что говорит о качестве сборки

### Бонусная часть

Давайте соберём геном из 100000 чтений, посмотрим как это повлияет на качество сборки

Весь код аналогичен описанному выше, так что описывать его не буду

Если вкратце, то я так же обрезал чтения, собрал из них контиги и скаффолды 
(убрав из них гэпы) и воспользовался уже написанным 
в ноутбуке кодом чтобы узнать различные статистики

Сравним следующие параметры:

* **Кол-во контигов и скаффолдов**

Контигов и скаффолдов заметно меньше. 
Чем меньше контигов тем выше качество сборки, так что в данном случае
получилось так, что уменьшение количества чтений улучшило сборку\)
* **Значение N50 для контигов и скаффолдов**

Если сравнивать N50 то тут несомненно выигрывает сборка с большим количеством чтений
* **Длина самого длинного контига и скаффолда**

Длина значительно уменьшилась по сравнению со сборкой где больше чтений, что в целом указывает на низкое
качество новой сборки. Но если взять длину в процентах от суммарной длины, то выигрывает по качеству новая сборка
* **Кол-во гэпов в самом длинном контиге и скаффолде**
 
Ну а здесь так вышло что в новой сборке нет гэпов, 
что говорит о том что сборка получилась хорошая, но стоит вспомнить,
что у нас самих скаффолдов меньше и суммарная длина их меньше, так что
отсутствие пропусков не есть показатель качества чтения в этом случае

# Вывод

Если учесть все параметры которые удалось сравнить,
то сборка с бОльшим количеством чтений качественнее, чем сборка с меньшим
