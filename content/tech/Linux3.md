---
title: "Implement grep command in C"
date: 2023-09-23T16:09:59+09:00
draft: false
---

今回は， ```mygrep``` を実装する．
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
#include <regex.h>

void mygrep(regex_t *preg, FILE *fp){
    char buf[4096];
    int err;

    while(fgets(buf, sizeof(buf), fp) != NULL){
        err = regexec(preg, buf, 0, NULL, 0);
        if(err == 0){
            fputs(buf, stdout);
        }
    }
}

int main(int argc, char *argv[]){

    regex_t preg;
    int err;
    char *pattern;
    FILE *fp;


    if(argc < 2){
        printf("Usage %s <pattern> <filename>", argv[0]);
        exit(1);
    }

    pattern = argv[1];
    err = regcomp(&preg, pattern, REG_EXTENDED | REG_NOSUB);
    if(err != 0){
        char buf[1024];
        regerror(err, &preg, buf, sizeof(buf));
        puts(buf);
        exit(1);
    }

    if(argc == 2){
        mygrep(&preg, stdin);
    }else{
        for(int i = 2;i < argc; i++){
            fp = fopen(argv[i], "r");
            if(fp == NULL){
                perror(argv[i]);
                exit(1);
            }
            mygrep(&preg, fp);
            fclose(fp);
        }
    }
    exit(0);
}
```

```sh
lubuntu@lubuntu-virtualbox:~/linux_tutorial$ ./mygrep m.* ./grep.c 
void mygrep(regex_t *preg, FILE *fp){
int main(int argc, char *argv[]){
        printf("Usage %s <pattern> <filename>", argv[0]);
    err = regcomp(&preg, pattern, REG_EXTENDED | REG_NOSUB);
        mygrep(&preg, stdin);
            mygrep(&preg, fp);
```

よさそう．

おしまい
