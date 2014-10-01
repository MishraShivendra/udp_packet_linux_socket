

/*=====================================Program :1=========================================*/
/*      Program: Extract packet information and show using multiple threads               */
/*      Author: Shivendra Mishra                                                          */
/*      Environment: gcc 4.8.2 on Geany 1.23.1                                            */
/*      Input/Output: packets/ corrosponding info                                         */
/*========================================================================================*/

#ifdef __linux__
#include<stdio.h>                                          // various headers
#include<pthread.h>
#include<string.h>
#include<stdlib.h>
#include<sys/socket.h>
#include<sys/types.h>
#include<arpa/inet.h>
#include<error.h>
#include<error.h>
#include<netinet/ip_icmp.h>   
#include<netinet/udp.h>   
#include<netinet/tcp.h>   
#include<netinet/ip.h>   
#include<unistd.h>


#define PORT 5002                                                                                   // listening on this port
pthread_mutex_t my_lock;
int sock_raw;
struct sockaddr_in source,dest;
struct qu                                                                                                     // queue for packets info
{
	struct qu *ptr;
	int pockt_len, packet_id;	
	char src[20];
}*new,*first=NULL,*last=NULL,*temp;

void print_ip_header(unsigned char* Buffer, int Size)                                    // This function extracts info from packet
{
    unsigned short iphdrlen;
    struct iphdr *iph = (struct iphdr *)Buffer;
    iphdrlen =iph->ihl*4;
    memset(&source, 0, sizeof(source));
    source.sin_addr.s_addr = iph->saddr;
    memset(&dest, 0, sizeof(dest));
    dest.sin_addr.s_addr = iph->daddr;
     new = (struct qu*)malloc(sizeof(struct qu));                                     // create a new node
     new->packet_id = ntohs(iph->id);                                                       // write packet id in node
     new->pockt_len = ntohs(iph->tot_len);                                                 // write packet length
     strcpy(new->src,inet_ntoa(source.sin_addr));                                   // copy source
     new->ptr = NULL;                                                                                        // initialize pointer to NULL
     pthread_mutex_lock(&my_lock);                                                        // get lock to avoid first node conflict
     if(first==NULL)                                                                                          //if it is first pocket
     {
		 first = new;                                                                                        // make this new node as first
		 last = first;
	 }
	else
	{
	  	last->ptr=new;                                                                                         // otherwise link the node and 
	  	last = new;                                                                                        // update last node
	}	
   pthread_mutex_unlock(&my_lock);                                          // release lock
}
void show_udp_packet(unsigned char *Buffer , int Size)
{     
    unsigned short iphdrlen;     
    struct iphdr *iph = (struct iphdr *)Buffer;
    iphdrlen = iph->ihl*4;     
    struct udphdr *udph = (struct udphdr*)(Buffer + iphdrlen);     
    print_ip_header(Buffer,Size);    
}
void ProcessudpPacket(unsigned char* buffer, int size)
{
                                                                                                            //Get the IP Header part of this packet
    
    struct iphdr *iph = (struct iphdr*)buffer;
    if(iph->protocol==17)                                                              //Check the Protocol and do accordingly...
    {
           show_udp_packet(buffer , size);
    }
}
void thrd_1(int *p)
{
	printf("\nThread1\n");
	int saddr_size , data_size;
    struct sockaddr saddr;
    struct in_addr in;
    struct sockaddr_in si_me; 
    unsigned char *buffer = (unsigned char *)malloc(65536);     
    sock_raw = socket(AF_INET,SOCK_RAW , IPPROTO_UDP);
    if(sock_raw < 0)
    {
        perror("Socket Error\n");
    }    
    memset((char *) &si_me, 0, sizeof(si_me));    
    si_me.sin_family = AF_INET;
    si_me.sin_port = htons(PORT);
    si_me.sin_addr.s_addr = htonl(INADDR_ANY);                                                                                                                                                     
    if( bind(sock_raw , (struct sockaddr*)&si_me, sizeof(si_me) ) == -1)                           //bind port with socket
    {
        perror("Bind Error");
    }    
    while(1)
    {
        saddr_size = sizeof saddr;                                                                                                                                                                  
        data_size = recvfrom(sock_raw , buffer , 65536 , 0 , &saddr , &saddr_size);                                                    //Get a Raw packet
        if(data_size <0 )
        {
            perror("Recvfrom error");
        }        
        ProcessudpPacket(buffer , data_size);                                                                            // Extract info
    }
    close(sock_raw);
    pthread_exit(0);
}

void thrd_2( int *p)                                       // This thread shows extracted info
{
   printf(" I'm thread 2\r");   
   int i,fst=0;
   while(1)
   {
	   
		   if((first==NULL)||(last==NULL))                                                                    // if list not available then wait
		   {
			   if(fst==0)
			   {
				  printf("Waiting for packet......\r") ; 
				  fst=1; 
				}  
		    }
		    else
		    {                                                                                                                        // otherwise print first header
				printf("Packet ID:%d Packet Length:%d Packet Source:%s\r",first->packet_id,first->pockt_len,first->src);
			   	pthread_mutex_lock(&my_lock);                                        // get lock
			   	temp=first;                																	// store first location													
			   	first = temp->ptr;                                                                                                                         // update first node  (moving first forward since we've printed)
			   	free(temp);                                                                                                                                      // delete first node (since it is printed)
			   	pthread_mutex_unlock(&my_lock);			   	                   //release lock
			}	   
   }       	
   pthread_exit(0);	
}

int main()
{
  	int c=10,j;
  	pthread_t tid_1, tid_2;
  	printf("\e[?25l");
  	printf("\nCreating Threads.....\r");
  	pthread_create(&tid_1,NULL,(void *) &thrd_1,(void *) &c);
  	printf("Created first(Packet Capturing) Thread.....\r");
  	pthread_create(&tid_2,NULL,(void *) &thrd_2,(void *) &c);
  	printf("Created Second(Disply) Thread.....\r");
  	pthread_join(tid_1,NULL);
  	pthread_join(tid_2,NULL);
  	printf("Returned\r");
  	printf("\e[?25h");
	return 0;
}
#else
   #error "This implementation works only in linux!"
 #endif
