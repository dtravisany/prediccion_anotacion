# Predicción y Anotación de Genes en Procariontes

Bienvenido al práctico de predicción y anotación en genes procariontes. 
En este práctico será la continuación de su práctico de ensamble, 
por lo que utilizaremos los resultados de sus ensambles.

## Materiales

1. Ensambles mediante OLC y De Bruijn
2. [Prodigal](https://github.com/hyattpd/prodigal/wiki) para predecir los [CDS](https://www.uniprot.org/help/cds_protein_definition)
3. [tRNAscan-SE](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6768409/) para predecir los tRNAs
4. [barrnap](https://github.com/tseemann/barrnap) para predecir los rRNA
5. [BLAST+](https://www.ncbi.nlm.nih.gov/books/NBK279690/)
6. Base de datos [SwissProt](https://www.expasy.org/resources/uniprotkb-swiss-prot) contenida en [UniProtKB](https://www.uniprot.org/) formateada para BLAST+.
7. Scripts custom para asignar función a los péptidos.
8. [eggnog-mapper](https://github.com/eggnogdb/eggnog-mapper) para anotación rápida del genoma.
9. [BUSCO](https://busco.ezlab.org/busco_userguide.html) Para revisar la completitud de nuestro genoma.
10. [GenoVi](https://github.com/robotoD/GenoVi) Para visualizar nuestros resultados.


### 1. Mover los Ensambles

Crearemos una nueva carpeta dentro de nuestro directorio `home`.

    mkdir anotacion
    
entraremos al directorio

    cd anotacion

:warning: En los próximos comandos, recuerde reemplazar la N por el número de su grupo.


Copiaremos los resultados de nuestros ensambles en el directorio que acabamos de crear y les cambiaremos el nombre
    
    cp ~/reads/canu_grupo9_DT/grupo9_DT.contigs.fasta canu.fasta
    cp ~/reads/spades_grupo9_DT/spades_grupo9_DT/scaffolds.fasta spades.fasta

### 2. `Prodigal` para la detección de CDS

Ejecutaremos `prodigal` para predecir los CDS:

    prodigal -a canu.faa -d canu.fna -f gff -o canu.gff -i canu.fasta
    prodigal -a spades.faa -d spades.fna -f gff -o spades.gff -i spades.fasta

### 3. `tRNAscan-SE` para la predicción de ARN de transferencia
  
Ejecutaremos `tRNAscan-SE` (el flag `-B` indica que el organismo es bacteriano):

    tRNAscan-SE -B -o canu.trna canu.fasta
    tRNAscan-SE -B -o spades.trna spades.fasta

### 4. `barrnap` para la detección de ARN ribosomales

Ejecutaremos `barrnap` para identificar los rRNAs.

:warning: `barrnap` escribe la anotación **GFF por la salida estándar (stdout)** y, con `--outseq`, guarda en un FASTA las secuencias de los rRNA. Por eso redirigimos el GFF a un archivo (`*.barrnap.gff`), que es el que necesitaremos más adelante en GenoVi:

    barrnap canu.fasta --outseq canu.rrna.fa > canu.barrnap.gff
    barrnap spades.fasta --outseq spades.rrna.fa > spades.barrnap.gff

### 5. `blastp` para anotar nuestros péptidos:

Ejecutamos `blastp` para anotar nuestras proteínas:

    blastp -num_threads 10 -db /opt/DB/swissprot16062026 -num_descriptions 5 -num_alignments 2 -evalue 1e-5 -query canu.faa -out canu.bp
    blastp -num_threads 10 -db /opt/DB/swissprot16062026 -num_descriptions 5 -num_alignments 2 -evalue 1e-5 -query spades.faa -out spades.bp
    
### 6. Scripts custom para asignar función a los péptidos.

Usamos `blastparser.pl` para extraer la mejor anotación desde el resultado de BLAST:

    blastparser.pl [-h] -b blastfile -p percent_identity > archivo_salida
     
    Parámetros:
    b	file	Archivo de salida de blast
    p	int	porcentaje de identidad [por defecto 35]
    archivo_salida	file	nombre del archivo de salida, por ejemplo: canu.parsed.bp, spades.parsed.bp

Por ejemplo:

    blastparser.pl -b canu.bp -p 35 > canu.parsed.bp
    blastparser.pl -b spades.bp -p 35 > spades.parsed.bp

### 7. [eggnog-mapper](https://github.com/eggnogdb/eggnog-mapper) para anotación rápida de los péptidos del genoma.

Ejecutamos:

    conda activate emapper

    emapper.py --cpu 4 --itype proteins --outfmt_short -i canu.faa -o canu_emapper
    emapper.py --cpu 4 --itype proteins --outfmt_short -i spades.faa -o spades_emapper

    conda deactivate
    
### 8. [BUSCO](https://busco.ezlab.org/busco_userguide.html) Para revisar la completitud de nuestro genoma.

Activamos el ambiente de conda donde está instalado busco

    conda activate busco

debería ver algo así: 
```console
    (busco)bioinfoN@devastator:~/anotacion$
```
Ejecutamos `busco` sobre el **proteoma predicho** (`.faa`) con los siguientes parámetros.

    busco --in canu.faa -o busco_canu -l bacteria_odb10 -c 24 --mode proteins --download_path /opt/DB/busco/
    busco --in spades.faa -o busco_spades -l bacteria_odb10 -c 24 --mode proteins --download_path /opt/DB/busco/

### 8.1 `BUSCO` en modo genome

Además del proteoma, evaluamos la completitud del **ensamble** en sí corriendo BUSCO en `--mode genome` sobre los FASTA del ensamble (`canu.fasta`/`spades.fasta`). En este modo usamos `--auto-lineage-prok` para que BUSCO seleccione automáticamente el linaje procarionte adecuado.

    busco --in canu.fasta -o busco_genome_canu --auto-lineage-prok -c 24 --mode genome --download_path /opt/DB/busco/
    busco --in spades.fasta -o busco_genome_spades --auto-lineage-prok -c 24 --mode genome --download_path /opt/DB/busco/

Desactivamos el ambiente de conda donde está instalado busco

    conda deactivate

debería ver algo así: 
```console
    (base)bioinfoN@devastator:~/anotacion$
```


### 9. [GenoVi](https://github.com/robotoD/GenoVi) Para visualizar nuestros resultados.

Primero creamos un archivo `gbk` que integra todas las anotaciones anteriores:

    generate_gbk.py --fasta canu.fasta --gff canu.gff --faa canu.faa --fna canu.fna --barrnap canu.barrnap.gff --trna canu.trna --swissprot canu.parsed.bp --emapper canu_emapper.emapper.annotations --output canu.gbk

Repetir para spades.

Finalmente generamos la figura circular del genoma con GenoVi. Como trabajamos con ensambles borrador (múltiples contigs/scaffolds), usamos `-s draft`:

    genovi -i canu.gbk -s draft -o canu_genovi
    genovi -i spades.gbk -s draft -o spades_genovi
