# operatingsystems1


Consider a scheduling
approach which is non pre-emptive similar to shortest job next in nature. The
priority of each job is dependent on its estimated run time, and also the
amount of time it has spent waiting. Jobs gain higher priority the longer they
wait, which prevents indefinite postponement. The jobs that have spent a long
time waiting compete against those estimated to have short run times. The
priority can be computed as :
Priority = 1+
Waiting time / Estimated run time
Write
a program to implement such an algorithm








#include<stdio.h>

#include<unistd.h>

#include<stdlib.h>

#include<malloc.h>

#include<time.h>

#include<pthread.h>

#include<semaphore.h>

struct Process {

 int
time,Atime,Btime,id;

 clock_t
arrival;

 int
flag,completed,p;

 sem_t
se;

 struct
Process *next;

 int
priority;

 int
WT;

};

int i=0,k=0;

typedef struct Process node;

clock_t start;

float A_TAT=0,A_WT=0;

node *Sp1=NULL,*Sp2=NULL,*temp;

void *processor(node *S) {

 clock_t
count;

 while(1)
{

           sem_wait(&S->se);

           if((S->Atime<=(clock()-start)/CLOCKS_PER_SEC
&& S->p==1)) {

                     S->p=0;

                     count=clock();

           }

           if(S->flag==1) {

                     printf("\nProcess-%d
Running \nTimer :%d",S->id,(clock()-start)/CLOCKS_PER_SEC);

                     S->flag=0;

                     S->arrival=clock();

           }

           if((clock()-count)/CLOCKS_PER_SEC==1)
{

                     count=clock();

                     printf("\nTimer
:%d",(clock()-start)/CLOCKS_PER_SEC);

                     S->time-=1;

                     if(S->time==0)
{

                               S->WT=((clock()-start)/CLOCKS_PER_SEC)-S->Btime-S->Atime;

                               A_TAT+=(clock()-start)/CLOCKS_PER_SEC-S->Atime;

                               A_WT+=((clock()-start)/CLOCKS_PER_SEC)-S->Btime-S->Atime;

                               sleep(2);

                               node *start=Sp2;

                               while(start!=NULL) {

                                         if(start->next==S)
{

                                                   start->next=S->next;

                                                   break;

                                         }

                                         if(Sp2==S)
{

                                                   Sp2=Sp2->next;

                                                   break;

                                         }

                                         start=start->next;

                               }

                               printf("\nProcess-%d Completed
",S->id);

                               if(Sp2!=NULL){

                                         printf("next
Process-%d",Sp2->id);

                               }

                               S->completed=7;

                               if(Sp2!=NULL){

                                         sem_post(&Sp2->se);

                               }

                     }

                     }

                                         if(S->completed==7)
{

                     break;

           }

 sem_post(&S->se);

 }

} 

void spush(node *temp) {

 int
k;

 node
*start=Sp2;

 k=temp->priority;

 k=1+(temp->WT/temp->time);

 if(Sp2==NULL)
{

           Sp2=temp;

           Sp2->next=NULL;

 }

 else{

 int
t=temp->time;

 if(start->priority<k){

           temp->next = Sp2;

        Sp2=temp;

 }

 else
if (start->time > t) {

           temp->next
= Sp2;

           Sp2=temp;

   
       }

   
else {

           while
(start->next != NULL && start->next->time< t) {

                              start = start->next;

           }

        temp->next = start->next;

        start->next = temp;

   
}

 }

}

void push() {

 temp=(node
*)malloc(sizeof(node));

 printf("\nEnter
Arrival Time of %d Process :",(i+1));

 scanf("%d",&temp->Atime);

 printf("Enter
Burst Time :");

 scanf("%d",&temp->time);

 temp->id=i+1;

 temp->p=1;

 temp->flag=1;

 temp->Btime=temp->time;

 temp->priority=1+(temp->WT/temp->time);

 sem_init(&temp->se,0,0);

 int
t=temp->Atime;

 node
*start=Sp1;

 if
(start->Atime > t) {

           temp->next
= Sp1;

           Sp1=temp;

   
       }

   
       else {

           while
(start->next != NULL && start->next->Atime < t) {

                              start = start->next;

           }

        temp->next = start->next;

    
   start->next = temp;

   
}

}

void main() {

 printf("\t\t\t\t\t\t\t\t\t\t*********Shortest
Job Next-Priority**********\n");

 int
n,l=1;

 pthread_t
p[10];

 printf("\nEnter
No.of Processes :");

 scanf("%d",&n);

 while(i<n)
{

           if(Sp1==NULL) {

                     Sp1=(node
*)malloc(sizeof(node));

                     printf("Enter
Arrival Time of %d Process :",(i+1));

                     scanf("%d",&Sp1->Atime);

                     printf("Enter
Burst Time :");

                     scanf("%d",&Sp1->time);

                     Sp1->id=i+1;

                     Sp1->flag=1;

                     Sp1->p=1;

                     Sp1->WT=0;

                     Sp1->Btime=Sp1->time;

                     Sp1->priority=1+(Sp1->WT/Sp1->time);

                     sem_init(&Sp1->se,0,0);

                     Sp1->next=NULL;

           }

           else {

                     push();

           }

           i++;

 }

 i=0;

 system("cls");

 printf("\t\t\t\t\t\t\t\t\t\t\t\t\**********PROCESSOR************\n\n");

 start=clock();

 while(i<n)
{

           temp=Sp1; 

           if(temp->Atime<=0) {

                     printf("Process-%d
is Dicarded Due to Incorrect Arrival Time\n",temp->id);

                     Sp1=temp->next;

                     temp=Sp1;

                     i++;

           }

           if(l==1) {

                               l=0;

                               sem_post(&temp->se);

                     }

           if((clock()-start)/CLOCKS_PER_SEC==temp->Atime)
{

           printf("Process-%d is
created\n",temp->id);

                     pthread_create(&p[i],NULL,processor,temp);

                     Sp1=Sp1->next;

                     spush(temp);

                     i++;

           }}

 for(i=0;i<n;i++)
{

           pthread_join(p[i],NULL);

 }

 printf("\nAverage
Waiting Time :%f\nAverage Turn Around Time :%f",(float)A_WT/n,(float)A_TAT/n);

