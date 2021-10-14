# Predicción y Anotación de Genes en Procariontes

Bienvenido al práctico de predicción y anotación en genes procariontes. 
En este práctico será la continuación de su práctico de ensamble, 
por lo que utilizaremos los resultados de sus ensambles.

## Materiales

1. Ensambles mediante OLC y De Bruijn
2. [tRNA-scanSE](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6768409/) para predecir los tRNAs
3. [barrnap](https://github.com/tseemann/barrnap) para predecir los rRNA
4. [Prodigal](https://github.com/hyattpd/prodigal/wiki) para predecir los [CDS](https://www.uniprot.org/help/cds_protein_definition)
5. [BLAST+](https://www.ncbi.nlm.nih.gov/books/NBK279690/)
6. Base de datos [SwissProt](https://www.expasy.org/resources/uniprotkb-swiss-prot) contenida en [UniProtKB](https://www.uniprot.org/) formateada para BLAST+.
7. Scripts custom para asignar función a los péptidos.

  
   
## Desarrollo

Crearemos una nueva carpeta dentro de nuestro directorio `home`.

    mkdir anotacion
    
entraremos al directorio

    cd anotacion

:warning: En los próximos comandos, recuerde reemplazar la N por el número de su grupo.


Copiaremos los resultados de nuestros ensambles en el directorio
    
    cp /mnt/md0/data4BT/btN/canu_btN/btN.contigs.fasta canu.fasta
    cp /mnt/md0/data4BT/btN/spades_btN/scaffolds.fasta spades.fasta


Ejecutaremos `prodigal` para predecir los CDS:

    prodigal -a canu.faa -d canu.fna -f gff -o canu.gff -i canu.fasta
    prodigal -a spades.faa -d spades.fna -f gff -o spades.gff -i spades.fasta
  
Ejecutaremos `tRNAscan-SE`

    tRNAscan-SE -B -o canu.trna canu.fasta
    tRNAscan-SE -B -o spades.trna spades.fasta

Ejecutaremos `barrnap` para identificar los rRNAs:
  
    barrnap canu.fasta -o canu.rrna
    barrnap spades.fasta -o spades.rrna

Ejecutamos `blastp` para anotar nuestros genes:

    blastp -num_threads 10 -db /mnt/md0/DB/uniprot_sprot.fasta -num_descriptions 5 -num_alignments 2 -evalue 1e-5 -query canu.faa -out canu.bp
    blastp -num_threads 10 -db /mnt/md0/DB/uniprot_sprot.fasta -num_descriptions 5 -num_alignments 2 -evalue 1e-5 -query spades.faa -out spades.bp
    
Script de anotación:

    
