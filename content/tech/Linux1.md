---
title: "Implement cat command in C"
date: 2023-09-20T22:42:51+09:00
draft: false
---

今回は，引数に渡したファイルを読み込んで逐次表示していく ```mycat``` を実装する．
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

愚直な実装なら何も難しくない

```C
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>

// implementation of cat command
// study for system call

void mycat(char *filename);

int main(int argc, char *argv[])
{
    if(argc < 2)
    {
        fprintf(stderr, "Usage: %s <filename> \n", argv[0]);
        exit(1);
    }

    for(int i = 1; i < argc; i++)
    {
        mycat(argv[i]);
    }
}

void mycat(char *filename){
    int fp, catted, re;
    unsigned char buf[2048];

    if((fp = open(filename, O_RDONLY)) < 0) // error for file not found
    {
        perror("fopen");
        exit(1);
    }

    while((catted = read(fp, buf, sizeof buf)) >= 0)
    {
        if(catted == 0)
        {
            break;
        }
        if((re = write(STDOUT_FILENO, buf, catted)) < 0){
            perror("write");
            exit(1);
        } 
    }

    if(close(fp) == -1)
    {
        perror("close");
        exit(1);
    }

}
```

実行するとこう

```sh
lubuntu@lubuntu-virtualbox:~/linux_tutorial$ gcc mycat.c -o mycat -Wall
lubuntu@lubuntu-virtualbox:~/linux_tutorial$ ./mycat
Usage: ./mycat <filename> 
lubuntu@lubuntu-virtualbox:~/linux_tutorial$ ./mycat tt.txt 
fopen: No such file or directory
lubuntu@lubuntu-virtualbox:~/linux_tutorial$ ./mycat t.txt 
aaaaaaaa
lubuntu@lubuntu-virtualbox:~/linux_tutorial$ ./mycat t.txt t2.txt
aaaaaaaa
bbbbbbbbbbbb
lubuntu@lubuntu-virtualbox:~/linux_tutorial$ 
```

重要なのは下記

```C
while((catted = read(fp, buf, sizeof buf)) >= 0)
    {
        if(catted == 0)
        {
            break;
        }
        if((re = write(STDOUT_FILENO, buf, catted)) < 0){
            perror("write");
            exit(1);
        } 
    }
```

```write``` の第三引数に catted を与えないと，必要以上にメモリを read することになる．
まあ ```man``` すれば分かることです．

## 拡張

### 引数が無い場合，標準入力を読み込む

これも簡単．
ファイルディスクリプタであり，ファイルポインタではない．

```C
    if(argc == 1){
        mycat("/dev/stdin");
        exit(0);
    }
```

### オプション(-n)で行番号を記述する

境界値の処理がちょいめんどうだが簡単．
おそらくオプションの処理は間違っている (libc を見る限り get_opt を使うべき?)が，とりあえずこんな感じ

```C

int main(int argc, char *argv[])
{

    (snip)

    if(argc > 1 && argv[1][1] == 'n'){
        show_line_number = 1;
        argc--;
        argv++;
    }
    
    (snip)
}

void mycat(char *filename, int show_line_number){

    (snip)

    while((catted = read(fp, buf, sizeof buf)) >= 0)
    {
        if(catted == 0)
        {
            break;
        }

        if(show_line_number && show_line_number == 1){
            int len = snprintf(line_buf, sizeof(line_buf), "%6d\t", line_number++);
            write(STDOUT_FILENO, line_buf, len);
        }

        for (ssize_t i = 0; i < catted; i++)
        {
            if(buf[i] == '\n'){
                if(show_line_number){
                    write(STDOUT_FILENO, &buf[i], 1);
                    i++;
                    int len = snprintf(line_buf, sizeof(line_buf), "%6d\t", line_number++);
                    write(STDOUT_FILENO, line_buf, len);
                }
            }

            if((re = write(STDOUT_FILENO, &buf[i], 1)) < 0){
                perror("write");
                exit(1);
            }
        }
    }

    (snip)

    write(STDOUT_FILENO, &enter, 2);

}
```

実行するとこう

```sh
lubuntu@lubuntu-virtualbox:~/linux_tutorial$ gcc mycat.c -o mycat -Wall
lubuntu@lubuntu-virtualbox:~/linux_tutorial$ ./mycat -n ./t2.txt 
     1  bbbbbbbbbbbb
     2  ccc
     3  cccc
     4  ddd
     5  eee
lubuntu@lubuntu-virtualbox:~/linux_tutorial$ cat ./t2.txt 
bbbbbbbbbbbb
ccc
cccc
ddd
eee
```

おしまい
