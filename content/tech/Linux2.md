---
title: "Implement head & tail command in C"
date: 2023-09-21T23:48:23+09:00
draft: false
---

今回は， ```myhead``` と ```mytail``` を実装する．
開発環境は VM 上の Lubuntu で，下記の通り．

```sh
lubuntu@lubuntu-virtualbox:~$ uname -a
Linux lubuntu-virtualbox 6.8.0-50-generic #51~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Thu Nov 21 12:03:03 UTC 2 x86_64 x86_64 x86_64 GNU/Linux

lubuntu@lubuntu-virtualbox:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.5 LTS
Release:        22.04
Codename:       jammy

lubuntu@lubuntu-virtualbox:~$ cat /proc/cpuinfo | grep "model name"
model name      : AMD Ryzen 5 5600X 6-Core Processor
```

## 基本的な実装

```C
#include <stdio.h>
#include <stdlib.h>

void myhead(FILE *fp, int nlines){
     int c;
     while(nlines > 0 && (c = getc(fp)) != EOF){
          if(putchar(c) < 0){
               exit(1);
          }
          if(c == '\n'){
               nlines--;
          }
          if(nlines == 0){
               return;
          }
     }
}

int main(int argc, char *argv[]){
     int nlines;
     if(argc != 2){
          printf("Usage: %s <filename> <nlines>\n", argv[0]);
          exit(1);
     }
     nlines = atoi(argv[1]);
     myhead(stdin, nlines);
     exit(0);
}
```

オプションを受けられるようにした & ファイル入力も引き受ける

```C
#include <stdio.h>
#include <stdlib.h>
#include <getopt.h>

void myhead(FILE *fp, int nlines);

struct option longopts[] = {
    {"lines", required_argument, NULL, 'n'},
    {0, 0, 0, 0}
};

void myhead(FILE *fp, int nlines){
    int c;

    while(nlines > 0 && (c = getc(fp)) != EOF){
        if(putchar(c) < 0){
            exit(1);
        }
        if(c == '\n'){
            nlines--;
        }
        if(nlines == 0){
            return;
        }
    }
}

int main(int argc, char *argv[]){

    int nlines, opt;

    if(argc < 2){
        printf("Usage: %s <filename> <nlines>\n", argv[0]);
        exit(1);
    }

    while((opt=getopt_long(argc, argv, "n:", longopts, NULL)) != -1){
        switch(opt){
            case 'n':
                nlines = atoi(optarg);
                break;
        }
    }

    if(argc == 2){
        myhead(stdin, nlines);
        exit(0);
    } else {
        FILE *fp;
        int i;

        for(i=2; i < argc; i++){
            fp = fopen(argv[i], "r");
            if(fp == NULL){
                perror(argv[i]);
                exit(1);
            }
            myhead(fp, nlines);
        }
    }
}
```

head もできるので tail もできるだろと思ったら少し面倒だった．
リングバッファが分かれば OK

```C
#include <stdio.h>
#include <stdlib.h>

int buf_counter = 0;

void mytail(int num, FILE *fp){
    char buf[num][1024];

    while(fgets(buf[buf_counter], sizeof(buf[buf_counter]), fp) != NULL){
        buf_counter = (buf_counter + 1) % num;
    }

    for(int i = 0; i < num; i++){
        printf("%s", buf[(buf_counter + i) % num]);
    }
}

int main(int argc, char *argv[]){
    if(argc < 2){
        printf("Usage: %s <num> <filename>\n", argv[0]);
        exit(1);
    }

    int num = atoi(argv[1]);

    if(argc == 2){
        mytail(num, stdin);
    }else{
        for(int i = 2; i < argc; i++){
            FILE *fp = fopen(argv[i], "r");
            if(fp == NULL){
                perror("fopen");
                exit(1);
            }
            mytail(num, fp);
            fclose(fp);
        }
        exit(0);
    }

}
```

```sh
$ ./mytail 10 ./tail.c 
                perror("fopen");
                exit(1);
            }
            mytail(num, fp);
            fclose(fp);
        }
        exit(0);
    }

}
```

よさそう

おしまい
