---
title: "Implement ls command in C"
date: 2023-09-24T16:21:23+09:00
draft: false
---

## 基本的な実装

```C
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>

void myls(char *path){
    DIR *d;
    struct dirent *dir;

    d = opendir(path);
    if(d == NULL){
        perror("opendir");
        exit(1);
    }

    while((dir = readdir(d)) != NULL){
        printf("%s\n", dir->d_name);
    }

    closedir(d);

}

int main(int argc, char *argv[]){
    if(argc < 2){
        printf("Usage: %s <filename>\n", argv[0]);
        exit(1);
    }

    for(int i = 1; i < argc; i++){
        myls(argv[i]);
    }

}
```

```sh
$ ./myls .
.
mytail
test.txt
myls
..
ls.c
myhead
tail.c
head.c
```

よさそう．

再帰探索を追加する．

```C
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>
#include <sys/stat.h>
#include <string.h>

void myls(char *path){
    DIR *d;
    struct dirent *dir;
    char fullpath[1024];
    struct stat st;

    d = opendir(path);
    if(d == NULL){
        perror("opendir");
        exit(1);
    }

    while((dir = readdir(d)) != NULL){
        if(strcmp(dir->d_name, "..") == 0 || strcmp(dir->d_name, ".") == 0){
            printf("%s\n", dir->d_name);
            continue;
        }

        snprintf(fullpath, sizeof(fullpath), "%s/%s", path, dir->d_name);
        //printf("OK");

        if(stat(fullpath, &st) == -1){
            perror("stat");
            continue;
        }

        if(S_ISDIR(st.st_mode)){
            printf("%s/\n", dir->d_name);
            myls(fullpath);
        }else{
            printf("%s\n", dir->d_name);
        }

    }

    closedir(d);

}

int main(int argc, char *argv[]){
    if(argc < 2){
        printf("Usage: %s <filename>\n", argv[0]);
        exit(1);
    }

    for(int i = 1; i < argc; i++){
        myls(argv[i]);
    }

}
```

dir 構造体の使い方をミスって 1 h 溶かした

```sh
$ ./linux_tutorial/head/myls ./linux_tutorial/
.
..
head/
.
mytail
test.txt
myls
..
ls.c
myhead
tail.c
head.c
grep/
.
greo.c
..
mygrep
```

よさそう．

おしまい
