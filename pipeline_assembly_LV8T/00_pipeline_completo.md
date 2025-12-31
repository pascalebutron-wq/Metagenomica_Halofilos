# Pipeline completo (LV8T)

## 1) Trimming (Trim Galore)
nohup trim_galore --paired --quality 30 --stringency 5 --fastqc --illumina \
  LV8Hv_CKDN240014724-1A_HKN7JDSXC_L3_1.fq.gz \
  LV8Hv_CKDN240014724-1A_HKN7JDSXC_L3_2.fq.gz \
  -o ./datos_filtrados &

##assembly (SPAdes)
nohup spades.py \
  -1 LV8Hv_CKDN240014724-1A_HKN7JDSXC_L3_1_val_1.fq.gz \
  -2 LV8Hv_CKDN240014724-1A_HKN7JDSXC_L3_2_val_2.fq.gz \
  --isolate --careful --cov-cutoff 10 -k 21,33,55,77 -t 30 \
  -o ./resultados_spades &

##Assembly (MEGAHIT)
nohup megahit \
  -1 LV8Hv_CKDN240014724-1A_HKN7JDSXC_L3_1_val_1.fq.gz \
  -2 LV8Hv_CKDN240014724-1A_HKN7JDSXC_L3_2_val_2.fq.gz \
  --min-contig-len 1000 --k-list 27,37,57,77 --mem-flag 2 \
  --out-dir ./resultados_megahit &

#evaluacion (QUAST)
quast final.contigs.fa -o ./resultados_quast --min-contig 500 --large --threads 30

#anotacion (Prodigal + eggnog-mapper)
prodigal -i final.contigs.fa -o genes_output.gbk -a proteins_output.faa -d nucleotides_output.fna

emapper.py -i proteins_output.faa \
  --output al_contigs.fasta.faa.emapper6 \
  --cpu 28 \
  --data_dir /datos2/ecogenomicalab/BasesDeDatos/eggnogmapper_v6/eggnog-maper6/

#Taxonomia (Kraken2)
kraken2-build --download-library bacteria --db Kraken_db

#taxonomia (Kaiju)
kaiju \
  -t /datos2/ecogenomicalab/BasesDeDatos/kaijudb/nr_euk/nodes.dmp \
  -f /datos2/ecogenomicalab/BasesDeDatos/kaijudb/nr_euk/kaiju_db_euk.fmi \
  -i final.contigs.fa \
  -o kaiju_output.txt

kaiju2table \
  -t /datos2/ecogenomicalab/BasesDeDatos/kaijudb/nr_euk/nodes.dmp \
  -n /datos2/ecogenomicalab/BasesDeDatos/kaijudb/nr_euk/names.dmp \
  -r genus \
  -o reporte_kaiju.txt \
  kaiju_output.txt

#Descarga de genomas (NCBI datasets) + Arbolito

contador=4
for zipfile in ncbi_datasethalo*.zip; do
  carpeta_destino="/datos2/ecogenomicalab/raw_data/HalomonasLV8/Genomas/Genoma_$contador"
  mkdir -p "$carpeta_destino"
  unzip -o "$zipfile" -d "$carpeta_destino"
  contador=$((contador + 1))
done

