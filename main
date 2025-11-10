#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include <stdio.h>
#include<unistd.h>
#include<sys/wait.h>
#include<sys/types.h>
#define max_history 20
struct var{
        char name[50];
        char value[100];
    };
struct var vars[50];
int varcount=0;
int excute_externals(char *x){
    char *args[10];
    int i=0;
    char *token=strtok(x," ");
    while(token!=NULL&&i<9){
        args[i++]=token;
        token=strtok(NULL," ");
    }
    args[i]=NULL;
    if(args[0]==NULL)
    return 1;
    pid_t pid=fork();
    if(pid<0){
        perror("fork failed");
        return 1;}
    else if (pid==0){
        execvp(args[0],args);
        perror("execvp failed");
        return(1);}
    else{
        wait(NULL);
    }
    return 0;
}
int unsetvariable(char *p){
    int idx=-1,i;
    if(p==NULL||*p=='\0'){
        printf("Error:No variable name provided.\n");
        return 1;
    }
    for(i=0;i<varcount;i++){
        if (strcmp(vars[i].name,p)==0){
            idx=i;
            break;}}
        if(idx==-1){
            printf("Variable '%s' not found ,you can check the list of variabl with 'varlist'\n",p);
            return 1;}
        for(i=idx;i<varcount-1;i++){
        vars[i]=vars[i+1];}
        varcount--;
        printf("Variable '%s' removed.\n",p);
        return 0;
    /*
    for(i=0;i<varcount;i++){
        printf("\n%d:%s =%s",i+1,vars[i].name,vars[i].value);
    }
    printf("Which one you want to delete? =>");
    scanf("%d",&idx);
    idx--;
    if(idx<0||idx>=varcount){
        printf("Invalid index.\n");
        return 1;
    }
    for(i=idx;i<varcount-1;i++){
        vars[i]=vars[i+1];}
        varcount--;
        return 0;*/
}
int setvariable(char *p){
    int B,i=0,j=0,x,idx;
    if(p==NULL||*p=='\0'){
        printf("Error:No variable name provided.\n");
        return 1;
    }
    char n[50]={0},v[100]={0};
    while(p[i]!='='&&p[i]!='\0'&&p[i]!='\n'){
        n[i]=p[i];
        i++;}
    n[i]='\0';
    if(p[i]=='=')i++;
    while(p[i]!='\0'&&p[i]!='\n'){
        v[j++]=p[i++];}
    v[j]='\0';
    for (int k=0;k< varcount;k++) {
    if (strcmp(vars[k].name, n) == 0) {
        strcpy(vars[k].value, v);
        return 0;}}
    if(varcount<50){
        strcpy(vars[varcount].name,n);
        strcpy(vars[varcount].value,v);
        varcount++;
    }
    else {
        printf("The variable list is full choose one of uour old variable to replace with new one\n");
        for(x=0;x<50;x++){
            printf("%d:%s = %s\n",x,vars[x].name,vars[x].value);
        }
        printf("Enter your index=");
        scanf("%d",&idx);
        while(getchar()!='\n');
        if(idx>=0&&idx<50){
            strcpy(vars[idx].name,n);
            strcpy(vars[idx].value,v);
        }
    }
    return 0;
}
int main(){
    printf("Commandline interpreter\n");
    char *history[max_history];
    char x[50];
    int n=0;
    while(1){
        printf("=> ");
        fflush(stdout);
        if(fgets(x,sizeof(x),stdin)==NULL)break;
        if(n<max_history){
            history[n++]=strdup(x);}
        else {
            free(history[0]);
            for(int i=1;i<=max_history;i++){
                history[i-1]=history[i];}
            history[max_history-1]=strdup(x);
        }
        x[strcspn(x,"\n")]='\0';
        if (strcmp(x,"exit")==0)break;
        else if(strncmp(x,"cd",2)==0){
            char *path=x+2;
            while(*path==' ')path++;
            if(chdir(path)!=0)printf("\nChanging directory error\n");} 
        else if(strcmp(x,"cwd")==0){
            char cwd[1024];
            if(getcwd(cwd,sizeof(cwd))!=NULL)printf("You are in:%s\n",cwd);
            else continue;
        }
        else if(strcmp(x,"help")==0){
            printf("| Command             | What happens                                                                              |\n");
            printf("| ------------------- | ------------------------------------------------------------------------------------------|\n");
            printf("|'exit'               |Exit the shell                                                                             |\n");
            printf("|'cwd '               |Current working directory                                                                  |\n");
            printf("|'cd  '               |Change directory,cd (/x) and the 'x' is the target destination you want to go              |\n");
            printf("|'help'               |What youre using now :),just the manual instruction explanation of the command you can use |\n");
            printf("|'clear'              |To clear everything you typed in the screen and start frome top left                       |\n");
            printf("|'set'                |Set an new variable,for example'set x=y'                                                   |\n");
            printf("|'unset'              |Delete a defined variable,for example'unset x'                                             |\n");
            printf("|'varlist'            |Prints the defiend variables list                                                          |\n");
            printf("|'echo'               |return what you write/'echo $x return the value of x                                       |\n");
            printf("|'ls'                 |Lists files in current directory                                                           |\n");
            printf("|'mkdir testdir'      |Creates a new directory named `testdir`                                                    |\n");
            printf("|'rmdir testdir'      |Removes an empty directory                                                                 |\n");
            printf("|'rm file.txt'        |Removes a file                                                                             |\n");
            printf("|'cp a.txt b.txt'     |Copies a file                                                                              |\n");
            printf("|'mv a.txt folder/'   |Moves or renames a file                                                                    |\n");
            printf("|'touch newfile.txt'  |Creates an empty file or updates timestamp                                                 |\n");
            printf("|'cat main.c'         |Displays file content                                                                      |\n");
            printf("|'pwd'                |Prints current working directory                                                           |\n");
            printf("|'date'               |Shows the system date/time                                                                 |\n");
            printf("|'whoami'             |Shows the current user                                                                     |\n");
            printf("|'ps'                 |Lists active processes                                                                     |\n");
            printf("|'history'            |List the last 20 inputs                                                                    |\n");

        }
        else if(strcmp(x,"clear")==0){printf("\033[H\033[J");}
        else if(strcmp(x,"history")==0){
            if(n!=0){
                for(int i=0;i<n;i++){
                    printf("%d:%s",i,history[i]);}}
            else {printf("No input yet.\n");}}
        else if(strncmp(x,"echo",4)==0){
            char *sentence=x+4;
            while(*sentence==' ')sentence++;
            if(sentence[0]=='$'){
            sentence++;
            if(varcount==0){printf("No variable set\n");}
            else{
                for(int i=0;i<varcount;i++){
                    if(strcmp(vars[i].name,sentence)==0){
                        printf("%s\n",vars[i].value);
                        break;
                }
                else{ printf("No variable set with this name:%s\n",sentence);}
            }}}
            else{
                printf("%s\n",sentence);
            }
            
        }
        else if(strncmp(x,"set",3)==0){
            char *p=x+3;
            while(*p==' ')p++;
            if(setvariable(p)==0)printf("Successfull\n");
            else printf("unsuccessfull\n");
        }
        else if(strncmp(x,"unset",5)==0){
            char *p=x+5;
            while(*p==' ')p++;
            if(unsetvariable(p)==0)printf("Successfull\n");
            else printf("unsuccessfull\n");
        }
        else if(strcmp(x,"varlist")==0){
            if(varcount==0){
                printf("No variable defined yet.\n");}
            else{
            for(int i=0;i<varcount;i++){
                printf("%d: %s=%s\n",i+1,vars[i].name,vars[i].value);
            }}}
        else {
            if(excute_externals(x)!=0){
                printf("You enter:: %s \n",x);
                printf("|----------------------------------|\n");
                printf("|which not declared in my terminal |\n");
                printf("|you can ask me to add or if you   |\n");
                printf("|type something wrong              |\n");
                printf("|you can get 'help'                |\n");
                printf("|----------------------------------|\n");
            }
        }
    }
}
