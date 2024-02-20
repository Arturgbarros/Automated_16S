# Automated-16S
![image](https://github.com/Arturgbarros/Automated-16S/assets/125391314/2ad1adb1-c148-4778-8011-040f5d6cdef1)
In this script, I aimed to automate the entire process of 16S analysis, allowing for flexibility in adjusting each command to suit different situations. Users can simply input the required parameters for each step, enabling seamless utilization of the code without any modifications.


## Dependences
* Conda general environment with:
    * fastqc
      ```
      conda install -c bioconda fastqc
      ```
    * multiqc
      ```
      conda install -c bioconda multiqc
      ```
    * trimmomatic
      ```
      conda install -c bioconda trimmomatic
      ```
    * trimgalore
      ```
      conda install -c bioconda trim-galore
      ```
    * pear
      ```
      conda install -c bioconda pear
      ```
* conda qiime enviroment with
    * qiime
      ```
      https://docs.qiime2.org/2023.9/install/native/
      ```
# The comand line without qiime
```bash
#!/bin/bash

echo 'The smallest number of the sequences'
read starte
echo 'The largest number of the sequences'
read end
echo 'letter start the archives'
read letter
echo 'number of threads'
read threads
echo 'where do you stoped?
-1 -> my first time
0 -> quality
1 -> trimming
2-> merge
3 -> qiime
'
read start
mkdir -p qiime dadosbrutos qualidade trimmagem qualidade/bruto merge trimmagem/trimgalore trimmagem/trimmomatic
mv ${letter}* dadosbrutos/
trimmo=""
trim=""
question=""
change=""
nameqiime=""
if [ "${start}" == 0 ] || [ "${start}" == -1 ];then
    for i in *fq*; do 
        fastqc dadosbrutos/$i -t ${threads} -o qualidade/bruto 
    done
    multiqc qualidade/
    question=n
    while [ "${question}" == "n" ]; do
        echo 'Is the ONLY thing you want to do remove the adapters? (y/n)'
        read trim
        if [ "${trim}" == "y" ]; then
            for ((i= ${starte}; i<= ${end}; i++)); do 
                trim_galore --paired -o trimmagem/trimgalore dadosbrutos/${letter}${i}_1* dadosbrutos/${letter}${i}_2*
            done
            rm trimmagem/trimgalore/*.txt*
            for i in *fq*; do 
                fastqc trimmagem/trimgalore/$i -t ${threads} -o trimmagem/trimgalore
            done
            multiqc trimmagem/trimgalore -o trimmagem/trimgalore/
        fi

        if [ "${trim}" == "n" ];then
            echo 'Is sequence trimming necessary? (y/n)'
            read trimmo
            if [ "${trimmo}" == "y" ]; then
                echo 'comand for i in ${list}; do trimmomatic -PE *${i}_1* *${i}_2* trimmagem/trimmomatic/${letter}${i}_1.pair.fq trimmagem/trimmomatic/${letter}${i}_1.unpair.fq trimmagem/trimmomatic/${letter}${i}_2.pair.fq trimmagem/trimmomatic/${letter}${i}_2.unpair.fq MINLEN:100 SLIDINGWINDOW:4:20'
                echo 'do you want to change the comand?(y/n)'
                read change
            

                if [ "${change}" == "y" ]; then
                    echo 'write your comand and the output for the dir trimmagem/trimmomatic/'
                    read comand
                    ${comand}
                    rm trimmagem/trimmomatic/*unpair*
                    for i in *fq*; do 
                        fastqc trimmagem/trimmomatic/$i -t ${threads} -o trimmagem/trimmomatic
                    done
                    multiqc trimmagem/trimmomatic -o trimmagem/trimmomatic/
                else
                    for ((i= ${starte}; i<= ${end}; i++)); do 
                        trimmomatic PE dadosbrutos/*${i}_1* dadosbrutos/*${i}_2* trimmagem/trimmomatic/${letter}${i}_1.pair.fq trimmagem/trimmomatic/${letter}${i}_1.unpair.fq trimmagem/trimmomatic/${letter}${i}_2.pair.fq trimmagem/trimmomatic/${letter}${i}_2.unpair.fq MINLEN:100 SLIDINGWINDOW:4:20
                    done
                    rm trimmagem/trimmomatic/*unpair*
                    for i in *fq*; do 
                        fastqc trimmagem/trimmomatic/$i -t ${threads} -o trimmagem/trimmomatic/
                    done
                    multiqc trimmagem/trimmomatic -o trimmagem/trimmomatic/ 
                fi
            fi
        fi
        question=y

        if [ "${trim}" == "y" ] || [ "${trimmo}" == "y" ]; then 
            echo 'Are the trimmings satisfactory? (y/n)'
            read question
            while [ "${question}" != 'n' ] && [ "${question}" != 'y' ]; do
                echo 'Are the trimmings satisfactory? (y/n)'
                read question
            done
            if [ "${trim}" == "y" ] && [ "${question}" == "n" ]; then
                rm -r trimmagem/trimgalore/*
            fi
            if [ "${trimmo}" == "y" ] && [ "${question}" == "n" ]; then
                rm -r trimmagem/trimmomatic/*
            fi
        fi
    done    

    if [ "${trim}" == "y" ]; then
    	rm trimmagem/trimgalore *fastqc.html*
        for ((i= ${starte}; i<= ${end}; i++)); do 
            pear -f trimmagem/trimgalore/${letter}${i}_1* -r trimmagem/trimgalore/${letter}${i}_2* -o merge/${letter}${i}_merged -j ${threads}
        done
        rm merge/*unassembled* merge/*discarded*
        echo 'Is the mapfile prepared? (y/n)'
        read ok
        if [ "${ok}" == "n" ];then
            echo 'Make the mapfile and run the command again'
            exit 1
        fi
    fi

    if [ "${trimmo}" == "y" ]; then
    	rm trimmagem/trimmomatic *fastqc.html*
        for ((i= ${starte}; i<= ${end}; i++)); do 
            pear -f trimmagem/trimmomatic/${letter}${i}_1* -r trimmagem/trimmomatic/${letter}${i}_2* -o merge/${letter}${i}_merged -j ${threads}
        done
        rm merge/*unassembled* merge/*discarded*
        echo 'Is the mapfile prepared? (y/n)'
        read ok
        if [ "${ok}" == "n" ];then
            echo 'Make the mapfile and run the command again'
            exit 1
        fi
    fi

    if [ "${trimmo}" == "n" ] || [ "${trim}" == 'n' ]; then
        for ((i= ${starte}; i<= ${end}; i++)); do 
            pear -f dadosbrutos/${letter}${i}_1* -r dadosbrutos/${letter}${i}_2* -o merge/${letter}${i}_merged -j ${threads}
        done
        rm merge/*unassembled* merge/*discarded*
        echo 'Is the mapfile prepared? (y/n)'
        read ok
        if [ "${ok}" == "n" ];then
            echo 'Make the mapfile and run the command again'
            break
        fi
    fi
    ./qiime.sh
fi

if [ "${start}" == 1 ];then
    question=n
    while [ "${question}" == "n" ]; do
        echo 'Is the ONLY thing you want to do remove the adapters? (y/n)'
        read trim
        if [ "${trim}" == "y" ]; then
            for ((i= ${starte}; i<= ${end}; i++)); do 
                trim_galore --paired -o trimmagem/trimgalore dadosbrutos/${letter}${i}_1* dadosbrutos/${letter}${i}_2*
            done
            rm trimmagem/trimgalore/*.txt*
            for i in *fq*; do 
                fastqc trimmagem/trimgalore/$i -t ${threads} -o trimmagem/trimgalore
            done
            multiqc trimmagem/trimgalore -o trimmagem/trimgalore/
        fi

        if [ "${trim}" == "n" ];then
            echo 'Is sequence trimming necessary? (y/n)'
            read trimmo
            if [ "${trimmo}" == "y" ]; then
                echo 'comand for i in ${list}; do trimmomatic -PE *${i}_1* *${i}_2* trimmagem/trimmomatic/${letter}${i}_1.pair.fq trimmagem/trimmomatic/${letter}${i}_1.unpair.fq trimmagem/trimmomatic/${letter}${i}_2.pair.fq trimmagem/trimmomatic/${letter}${i}_2.unpair.fq MINLEN:100 SLIDINGWINDOW:4:20'
                echo 'do you want to change the comand?(y/n)'
                read change
            

                if [ "${change}" == "y" ]; then
                    echo 'write your comand and the output for the dir trimmagem/trimmomatic/'
                    read comand
                    ${comand}
                    rm trimmagem/trimmomatic/*unpair*
                    for i in *fq*; do 
                        fastqc trimmagem/trimmomatic/$i -t ${threads} -o trimmagem/trimmomatic
                    done
                    multiqc trimmagem/trimmomatic -o trimmagem/trimmomatic/
                else
                    for ((i= ${starte}; i<= ${end}; i++)); do 
                        trimmomatic PE dadosbrutos/*${i}_1* dadosbrutos/*${i}_2* trimmagem/trimmomatic/${letter}${i}_1.pair.fq trimmagem/trimmomatic/${letter}${i}_1.unpair.fq trimmagem/trimmomatic/${letter}${i}_2.pair.fq trimmagem/trimmomatic/${letter}${i}_2.unpair.fq MINLEN:100 SLIDINGWINDOW:4:20
                    done
                    rm trimmagem/trimmomatic/*unpair*
                    for i in *fq*; do 
                        fastqc trimmagem/trimmomatic/$i -t ${threads} -o trimmagem/trimmomatic/
                    done
                    multiqc trimmagem/trimmomatic -o trimmagem/trimmomatic/ 
                fi
            fi
        fi
        question=y

        if [ "${trim}" == "y" ] || [ "${trimmo}" == "y" ]; then 
            echo 'Are the trimmings satisfactory? (y/n)'
            read question
            while [ "${question}" != 'n' ] && [ "${question}" != 'y' ]; do
                echo 'Are the trimmings satisfactory? (y/n)'
                read question
            done
            if [ "${trim}" == "y" ] && [ "${question}" == "n" ]; then
                rm -r trimmagem/trimgalore/*
            fi
            if [ "${trimmo}" == "y" ] && [ "${question}" == "n" ]; then
                rm -r trimmagem/trimmomatic/*
            fi
        fi
    done    

    if [ "${trim}" == "y" ]; then
    	rm trimmagem/trimgalore *fastqc.html*
        for ((i= ${starte}; i<= ${end}; i++)); do 
            pear -f trimmagem/trimgalore/${letter}${i}_1* -r trimmagem/trimgalore/${letter}${i}_2* -o merge/${letter}${i}_merged -j ${threads}
        done
        rm merge/*unassembled* merge/*discarded*
        echo 'Is the mapfile prepared? (y/n)'
        read ok
        if [ "${ok}" == "n" ];then
            echo 'Make the mapfile and run the command again'
            exit 1
        fi
    fi

    if [ "${trimmo}" == "y" ]; then
    	rm trimmagem/trimmomatic *fastqc.html*
        for ((i= ${starte}; i<= ${end}; i++)); do 
            pear -f trimmagem/trimmomatic/${letter}${i}_1* -r trimmagem/trimmomatic/${letter}${i}_2* -o merge/${letter}${i}_merged -j ${threads}
        done
        rm merge/*unassembled* merge/*discarded*
        echo 'Is the mapfile prepared? (y/n)'
        read ok
        if [ "${ok}" == "n" ];then
            echo 'Make the mapfile and run the command again'
            exit 1
        fi
    fi

    if [ "${trimmo}" == "n" ] || [ "${trim}" == 'n' ]; then
        for ((i= ${starte}; i<= ${end}; i++)); do 
            pear -f dadosbrutos/${letter}${i}_1* -r dadosbrutos/${letter}${i}_2* -o merge/${letter}${i}_merged -j ${threads}
        done
        rm merge/*unassembled* merge/*discarded*
        echo 'Is the mapfile prepared? (y/n)'
        read ok
        if [ "${ok}" == "n" ];then
            echo 'Make the mapfile and run the command again'
            break
        fi
    fi
    ./qiime.sh
fi

if [ "${start}" == 2 ];then
    echo 'fez 
    2 = trimmomatic
    1 = trimgalore
    0 = nenhum?'
    read trimmagem
    if [ "${trimmagem}" == 1 ];then
        trim=y
    fi
    if [ "${trimmagem}" == 2 ];then
        trimmo=y
    fi
    if [ "${trimmagem}" == 0 ];then
        trim=n
        trimmo=n
    fi
    if [ "${trim}" == "y" ]; then
        for ((i= ${starte}; i<= ${end}; i++)); do 
            pear -f trimmagem/trimgalore/${letter}${i}_1* -r trimmagem/trimgalore/${letter}${i}_2* -o merge/${letter}${i}_merged -j ${threads}
        done
        rm merge/*unassembled* merge/*discarded*
        echo 'Is the mapfile prepared? (y/n)'
        read ok
        if [ "${ok}" == "n" ];then
            echo 'Make the mapfile and run the command again'
            exit 1
        fi
    fi

    if [ "${trimmo}" == "y" ]; then
        for ((i= ${starte}; i<= ${end}; i++)); do 
            pear -f trimmagem/trimmomatic/${letter}${i}_1* -r trimmagem/trimmomatic/${letter}${i}_2* -o merge/${letter}${i}_merged -j ${threads}
        done
        rm merge/*unassembled* merge/*discarded*
        echo 'Is the mapfile prepared? (y/n)'
        read ok
        if [ "${ok}" == "n" ];then
            echo 'Make the mapfile and run the command again'
            exit 1
        fi
    fi

    if [ "${trimmo}" == "n" ] || [ "${trim}" == 'n' ]; then
        for ((i= ${starte}; i<= ${end}; i++)); do 
            pear -f dadosbrutos/${letter}${i}_1* -r dadosbrutos/${letter}${i}_2* -o merge/${letter}${i}_merged -j ${threads}
        done
        rm merge/*unassembled* merge/*discarded*
        echo 'Is the mapfile prepared? (y/n)'
        read ok
        if [ "${ok}" == "n" ];then
            echo 'Make the mapfile and run the command again'
            break
        fi
    fi
    ./qiime.sh
fi

if [ "${start}" == 3 ];then
	echo 'Is the mapfile prepared? (y/n)'
        read ok
        if [ "${ok}" == "n" ];then
            echo 'Make the mapfile and run the command again'
            break
        fi
	./qiime.sh
fi
```
