# Predicción y Anotación de Genes en Procariontes

Bienvenido al práctico de predicción y anotación en genes procariontes. 
En este práctico será la continuación de su práctico de ensamble, 
por lo que utilizaremos los resultados de sus ensambles.

## Materiales

1. Ensambles mediante OLC y De Bruijn
2. [Prodigal](https://github.com/hyattpd/prodigal/wiki)
3. [BLAST+](https://www.ncbi.nlm.nih.gov/books/NBK279690/)
4. Base de datos [SwissProt](https://www.expasy.org/resources/uniprotkb-swiss-prot) contenida en [UniProtKB](https://www.uniprot.org/) formateada para BLAST+.
5. Scripts custom para asignar función a los péptidos.
  
## Desarrollo

Crearemos una nueva carpeta dentro de nuestro directorio `home`.

    mkdir anotacion
    
entraremos al directorio

    cd anotacion
    
copiaremos los resultados de nuestros ensambles en el directorio
    
:warning: En los próximos comandos, recuerde reemplazar 
la N por el número de su grupo.


  cp /mnt/md0/datos4BT/btN/canu_btN/btN.contigs.fasta .
  
  cp /mnt/md0/datos4BT/btN/spades_btN



