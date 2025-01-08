---
title: "Implement small shell with fork and exec"
date: 2023-09-25T16:43:36+09:00
draft: false
---

小さなシェルを実装する．
シェルとしてのループを実行中に，標準入力からコマンドが入ると fork() して子プロセスを exec() で塗り替える

```C
// implementation for shell with fork and exec

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/types.h>

#define MAX_ARGS 100

void myexec(char *input){
    char *args[MAX_ARGS];
    int arg_count = 0;

    char *p = strtok(input, " \t\n");
    while(p != NULL && arg_count < MAX_ARGS){
        args[arg_count++] = p;
        p = strtok(NULL, " \t\n");
    }

    args[arg_count] = NULL;

    if(arg_count == 0){
        return;
    }

    pid_t pid = fork();

    if(pid < 0){
        perror("fork");
        exit(1);
    }

    if(pid == 0){ // child 
        int err;
        err = execvp(args[0], args);
        if(err < 0){
            perror("execvp");
            exit(1);
        } 
    }else{ // parent
        int status;
        waitpid(pid, &status, 0);
    }
}

int main(){
    char buf[1024];
    printf("---- my shell ----\n");

    while(1){
        printf("myshell> ");
        fflush(stdout);

        if(fgets(buf, sizeof(buf), stdin) == NULL){
            break;
        }

        if(strncmp(buf, "exit", 4) == 0 && buf[4] == '\n'){
            break;
        }

        myexec(buf);

    }

    printf("Good Bye\n");
    exit(0);
}
```

実行するとこんな感じ

```sh
lubuntu@lubuntu-virtualbox:~/linux_tutorial$ ./myshell
---- my shell ----
myshell> ls
grep.c  head  mycat  mycat.c  mygrep  myshell  myshell.c  t2.txt  t.txt
myshell> hello
execvp: No such file or directory
myshell> pwd
/home/lubuntu/linux_tutorial
myshell> exit
Good Bye
```

execvp のエラーが変だけどいっか．．．

次にリダイレクトを実装する．
dup2 で標準入力と標準出力のファイルディスクリプタを子プロセスに割り当ててあげる．
冗長でダサいが暫定的に下記．

例えば ">" にぶつかったら，その次に宣言されているファイルを探し，そのファイルディスクリプタを複製する．(```dup2(int oldfd, int newfd);```)
これによって，その次のファイルディスクリプタを標準入力に割り当てる．\
oldfd を複製して newfd に割り当てる．
つまり，newfd は oldfd と同じファイルを指す．
なので，標準出力はその次のファイルディスクリプタを（その fork() 内では）指す．
これによって，そのコマンドの実行結果はファイルディスクリプタに出力される．

```C
if(pid == 0){ // child
        for(int i = 0; i < arg_count; i++){
            if(strncmp(args[i], ">", 1) == 0){
                int fd = open(args[i + 1], O_WRONLY | O_CREAT | O_TRUNC, 0644);
                if(fd < 0){
                    perror("open");
                    exit(1);
                }
                dup2(fd, STDOUT_FILENO);
                close(fd);
                args[i] = NULL;
                break;
            }
            if(strncmp(args[i], ">>", 1) == 0){
                int fd = open(args[i + 1], O_WRONLY | O_CREAT | O_APPEND, 0644);
                if(fd < 0){
                    perror("open");
                    exit(1);
                }
                dup2(fd, STDOUT_FILENO);
                close(fd);
                args[i] = NULL;
                break;
            }
            if(strncmp(args[i], "<", 1) == 0){
                int fd = open(args[i + 1], O_RDONLY);
                if(fd < 0){
                    perror("open");
                    exit(1);
                }
                dup2(fd, STDIN_FILENO);
                close(fd);
                args[i] = NULL;
                break;
            }
        }
        int err;
        err = execvp(args[0], args);
        if(err < 0){
            perror("execvp");
            exit(1);
        } 
    }else{ // parent
        int status;
        waitpid(pid, &status, 0);
    }
```

```sh
lubuntu@lubuntu-virtualbox:~/linux_tutorial$ ./myshell
---- my shell ----
myshell> ls
grep.c  head  mycat  mycat.c  mygrep  myshell  myshell.c  t2.txt  t.txt
myshell> cat mycat.c > tem.txt
myshell> cat tem.txt
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
...
```

次にパイプを実装する．

```C
void mypipe(char *input){
    char *args[MAX_ARGS];
    int arg_count = 0;

    char *p = strtok(input, "|");
    while(p != NULL && arg_count < MAX_ARGS){
        args[arg_count++] = p;
        p = strtok(NULL, "|");
    }

    int pipefd[2];
    int fd_in = 0;

    for(int i = 0; i < arg_count; i++){
        pipe(pipefd);

        if(fork() == 0){ // child
            dup2(fd_in, STDIN_FILENO);
            if(i < arg_count - 1){
                dup2(pipefd[1], STDOUT_FILENO);
            }
            close(pipefd[0]);
            myexec(args[i]);
            exit(0);
        }
        close(pipefd[1]);
        fd_in = pipefd[0];
    }
    for (int i = 0; i < arg_count; i++){
        wait(NULL);
    }
}

int main(){
    char buf[1024];
    printf("---- my shell ----\n");

    while(1){
        printf("myshell> ");
        fflush(stdout);

        if(fgets(buf, sizeof(buf), stdin) == NULL){
            break;
        }

        if(strncmp(buf, "exit", 4) == 0 && buf[4] == '\n'){
            break;
        }

        if(strchr(buf, '|') != NULL){
            mypipe(buf);
        }

        myexec(buf);

    }

    printf("Good Bye\n");
    exit(0);
}
```

よさそう．
パイプラインもリダイレクトも一つにしか対応できていないがまあモックとしよう．．．
ちゃんとしたシェルはまた Rust で書こう

おしまい
