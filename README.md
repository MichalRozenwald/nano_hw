Работу выполнили:
1) Мария Фоменко
2) Михаль Розенвальд
3) Александр Черницов
4) Филип Чиро


1. Скачайте прочтения, указанные для вашей группы, так, как было показано на практическом занятии. Учтите, что для правильного скачивания парно-концевых прочтений (в два отдельных файла) необходимо указать правильный флаг в fastq-dump.
Установим необходимые программы при помощи conda:

conda install samtools  
conda install sra-tools  
conda install filtlong  
conda install unicycler
Скачаем риды с помощью команды fastq-dump; для коротких ридов, полученных при помощи Illumina, используем дополнительно флаг --split-files, чтобы корректно считать в два файла парно-концевые прочтения).

fastq-dump SRR12282614  
fastq-dump --split-files SRR12282613 
2. Используя программу filtlong, выберите из длинных прочтений самые длинные, суммарной длиной не более 400 миллионов п.н.
Выберем самые длинные риды при помощи filtlong. Целевое значение суммарной длины — 400000000 пар нуклеотидов — указано под флагом --target-bases. Поскольку поставлена задача отобрать риды в первую очередь по длине, добавлен флаг --length_weight 10, чтобы увеличить вес длины в метрике качества, которую использует программа.

filtlong -1 SRR12282613_1.fastq -2 SRR12282613_2.fastq --length_weight 10 --target_bases 400000000 SRR12282614.fastq > output.fastq
3. Используя программу unicycler, соберите геном бактерии из выбранных длинных и всех коротких прочтений, в гибридном режиме.
Соберём геном при помощи unicycler в гибридном режиме (в качестве файла с длинными чтениями использован результат предыдущего пункта).

unicycler -1 SRR12282613_1.fastq -2 SRR12282613_2.fastq -l output.fastq --threads 15 -o unicycler_output
4. Используя NCBI Blast, определите, к какому виду принадлежит собранная бактерия.
Используем blastn (поскольку мы ищем геном — нуклеотидную последовательность — по другой нуклеотидной последовательности), в качестве запроса возьмём первые 200000 нуклеотидов кольцевой хромосомы из полученной последовательности в формате fasta.

Обрезался файл для запроса следующим образом:

samtools faidx unicycler_output/assembly.fasta 1:1-200000 > assembly_cut_3.fasta  
Поиск вёлся только по бактериям (taxid: 2). Пять результатов с лучшим score (заметим, что для всех E-value = 0.0) однозначно указывают, что перед нами бактерия Pseudomonas syringae.

5. Визуализируйте сборку при помощи программы Bandage.
С сайта https://rrwick.github.io/Bandage/ скачали программу Bandage для Windows. На вход подавали файлы с расширением .gfa - их у нас 6 штук, полученных из программы unicycler, визуализируем некоторые из них:

Граф с удаленными перекрытиями, где вершины - это различные прочтения, а ребра графа - их возможные соединения. image.png

Граф практически собранной хромосомы нашей бактерии. На нем уже отчетливо видна кольцевая форма, но не все вершины объединены. image.png

Граф полностью собранной кольцевой хромосомы бактерии: image.png

6. Найдите в вашей сборке гены антибиотикорезистентности и вирулентности. Используйте для этого программу abricate. Попробуйте все прилагаемые базы данных.
Клонируем репозиторий abricate и добавим скрипты any2fasta и Path/Tiny.pm:

git clone https://github.com/tseemann/abricate.git  
cd abricate/bin  
wget https://raw.githubusercontent.com/tseemann/any2fasta/master/any2fasta  
export PATH=~/Nanopore/abricate/bin:$PATH    
mkdir Path  
curl "https://raw.githubusercontent.com/dagolden/Path-Tiny/master/lib/Path/Tiny.pm" > Path/Tiny.pm  
Базы данных для поиска:

abricate --setupdb
Запускаем по каждой из баз данных:

 abricate --db=argannot  ~/assembly/assembly.fasta > hw.out  # -----   
 abricate --db=ecoli_vf  ~/assembly/assembly.fasta > hw.out  # -----  
 abricate --db=megares  ~/assembly/assembly.fasta > hw.out  # нашлись  
 abricate --db=ecoh  ~/assembly/assembly.fasta > hw1.out  # -----  
 abricate --db=card  ~/assembly/assembly.fasta > hw1.out  # нашлись  
 abricate --db=resfinder  ~/assembly/assembly.fasta > hw2.out # -----   
 abricate --db=vfdb  ~/assembly/assembly.fasta > hw2.out  # нашлись  
 abricate --db=ncbi  ~/assembly/assembly.fasta > hw3.out  # -----  
 abricate --db=plasmidfinder  ~/assembly/assembly.fasta > hw3.out  # -----  
В итоге гены нашлись в трёх базах; результаты мы скомпилировали в один файл genes_summary.out:

SEQUENCE	START	END	STRAND	GENE	COVERAGE	COVERAGE_MAP	GAPS	%COVERAGE	%IDENTITY	DATABASE	ACCESSION	PRODUCT	RESISTANCE
1	3355836	3358985	+	MEXF	1-3150/3180	========/======	4/8	98.93	84.81	megares	MEG_3911	Multi-compound:Drug_and_biocide_resistance:Drug_and_biocide_RND_efflux_pumps:MEXF	
1	4088983	4089650	+	CPXAR	1-668/678	========/======	9/12	97.64	80.27	megares	MEG_2117	Multi-compound:Drug_and_biocide_resistance:Drug_and_biocide_RND_efflux_regulator:CPXAR	
1	4616917	4619987	+	TTGB	1-3071/3153	========/======	18/38	96.80	80.32	megares	MEG_7289	Multi-compound:Drug_and_biocide_resistance:Drug_and_biocide_RND_efflux_pumps:TTGB	
1	4620048	4621445	+	EMHC	1-1398/1461	========/======	3/8	95.41	80.67	megares	MEG_2718	Multi-compound:Drug_and_biocide_resistance:Drug_and_biocide_RND_efflux_pumps:EMHC	
1	3355836	3358948	+	MexF	1-3119/3189	========/======	8/16	97.46	83.90	card	AE004091.2:2810008-2813197	MexF is the multidrug inner membrane transporter of the MexEF-OprN complex. mexF corresponds to 2 loci in Pseudomonas aeruginosa PAO1 (gene name: mexF/mexB) and 4 loci in Pseudomonas aeruginosa LESB58 (gene name: mexD/mexB).	diaminopyrimidine;fluoroquinolone;phenicol
1	4088983	4089650	+	Pseudomonas_aeruginosa_CpxR	1-668/678	========/======	9/12	97.64	80.27	card	LT673656.1:1885022-1884344	CpxR is directly involved in activation of expression of RND efflux pump MexAB-OprM in P. aeruginosa. CpxR is required to enhance mexAB-oprM expression and drug resistance in the absence of repressor MexR.	aminocoumarin;aminoglycoside;carbapenem;cephalosporin;cephamycin;diaminopyrimidine;fluoroquinolone;macrolide;monobactam;penam;penem;peptide;phenicol;sulfonamide;tetracycline
1	352646	353692	+	pilM	19-1065/1065	========/======	4/4	98.12	80.27	vfdb	NP_253731	(pilM) type IV pilus inner membrane platform protein PilM [Type IV pili (VF0082)] [Pseudomonas aeruginosa PAO1]	
1	354282	354887	+	pilO	19-624/624	========/======	6/10	96.31	80.85	vfdb	NP_253729	(pilO) type IV pilus inner membrane platform protein PilO [Type IV pili (VF0082)] [Pseudomonas aeruginosa PAO1]	
1	436300	437334	+	pilT	1-1035/1035	========/======	2/2	99.90	83.59	vfdb	NP_249086	(pilT) twitching motility protein PilT [Type IV pili (VF0082)] [Pseudomonas aeruginosa PAO1]	
1	445311	445694	+	pilG	1-384/408	===============	0/0	94.12	82.81	vfdb	NP_249099	(pilG) twitching motility protein PilG [Type IV pili (VF0082)] [Pseudomonas aeruginosa PAO1]	
1	445768	446133	+	pilH	1-366/366	===============	0/0	100.00	84.43	vfdb	NP_249100	(pilH) twitching motility protein PilH [Type IV pili (VF0082)] [Pseudomonas aeruginosa PAO1]	
1	502343	503302	+	waaF	1-960/1038	==============.	0/0	92.49	80.21	vfdb	NP_253699	(waaF) heptosyltransferase I [LPS (VF0085)] [Pseudomonas aeruginosa PAO1]	
1	1138012	1139415	-	algA	1-1404/1446	===============	0/0	97.10	81.41	vfdb	NP_252241	(algA) phosphomannose isomerase / guanosine 5'-diphospho-D-mannose pyrophosphorylase [Alginate (VF0091)] [Pseudomonas aeruginosa PAO1]	
1	1141675	1142958	-	algI	1-1284/1563	=============..	0/0	82.15	83.02	vfdb	NP_252238	(algI) alginate o-acetyltransferase AlgI [Alginate (VF0091)] [Pseudomonas aeruginosa PAO1]	
1	1151494	1152879	-	alg8	100-1485/1485	.==============	0/0	93.33	80.23	vfdb	NP_252231	(alg8) alginate-c5-mannuronan-epimerase AlgG [Alginate (VF0091)] [Pseudomonas aeruginosa PAO1]	
1	2048190	2048661	-	pvdS	43-514/564	.=======/=====.	2/4	83.33	82.91	vfdb	NP_251116	(pvdS) extracytoplasmic-function sigma-70 factor [Pyoverdine (VF0094)] [Pseudomonas aeruginosa PAO1]	
1	2063055	2064415	+	pvdH	7-1367/1410	========/======	2/2	96.45	80.47	vfdb	NP_251103	(pvdH) diaminobutyrate-2-oxoglutarate aminotransferase PvdH [pyoverdine (IA001)] [Pseudomonas aeruginosa PAO1]	
1	2064543	2064745	+	mbtH-like	1-203/219	==============.	0/0	92.69	83.74	vfdb	NP_251102	(mbtH-like) MbtH-like protein from the pyoverdine cluster [pyoverdine (IA001)] [Pseudomonas aeruginosa PAO1]	
1	3907227	3908035	-	fleN	10-818/843	===============	0/0	95.97	83.56	vfdb	NP_250145	(fleN) flagellar synthesis regulator FleN [Flagella (VF0273)] [Pseudomonas aeruginosa PAO1]	
1	3914185	3914873	-	fliP	70-758/768	.=======/======	2/2	89.58	82.61	vfdb	NP_250137	(fliP) flagellar biosynthetic protein FliP [Flagella (VF0273)] [Pseudomonas aeruginosa PAO1]	
1	3915916	3916856	-	fliM	1-941/972	========/======	2/2	96.71	82.70	vfdb	NP_250134	(fliM) flagellar motor switch protein FliM [Flagella (VF0273)] [Pseudomonas aeruginosa PAO1]	
1	3926409	3927402	-	fliG	22-1015/1017	===============	0/0	97.74	83.60	vfdb	NP_249793	(fliG) flagellar motor switch protein G [Flagella (VF0273)] [Pseudomonas aeruginosa PAO1]	
1	3951049	3952099	-	flgI	60-1110/1110	===============	0/0	94.68	82.49	vfdb	NP_249775	(flgI) flagellar P-ring protein precursor FlgI [Flagella (VF0273)] [Pseudomonas aeruginosa PAO1]	
1	3952994	3953749	-	flgG	10-762/786	========/======	1/3	95.80	81.75	vfdb	NP_249773	(flgG) flagellar basal-body rod protein FlgG [Flagella (VF0273)] [Pseudomonas aeruginosa PAO1]	
1	4547850	4548431	-	algU	1-582/582	===============	0/0	100.00	81.10	vfdb	NP_249453	(algU) alginate biosynthesis protein AlgZ/FimS [Alginate (VF0091)] [Pseudomonas aeruginosa PAO1]	

