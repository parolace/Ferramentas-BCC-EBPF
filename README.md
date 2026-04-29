# Ferramentas de observabilidade, eBPF, para Linux.
# Execsnoop, Exitsnoop e Threadsnoop.

Ferramentas clássicas de observabilidade para Linux, execsnoop, exitsnoop e threadsnoop. Elas são ferramentas poderosas do BCC (BPF Compiler Collection) usadas para monitorar e depurar sistemas Linux que utilizam o eBPF (Extended Berkeley Packet Filter). Elas fornecem informações em tempo real sobre o comportamento dos processos com sobrecarga mínima.  

### Requisitos:  

### Instale o pacote BCC/BPFCC.  
** Para instalar pacotes bpfcc-tools. 
```bash
sudo apt update
sudo apt install bpfcc-tools linux-headers-$(uname -r)
```
** Instalação do pacote BCC completo:
```bash
sudo apt update
sudo apt install bcc-tools linux-headers-$(uname -r)
```

Verificar se instalou. Provavelmente vai mostrar execsnoop, exitsnoop, threadsnoop e outros.    

```bash
$ ls /usr/share/bcc/tools/ | grep snoop
```

### 1. execsnoop (Rastrear Execuções de Processos)  
Objetivo: Monitorar e registrar novos processos em tempo real, execve(),  especificamente a função do kernel sys_execve(). Essa é a syscall que substitui o código atual de um processo por um novo programa, normalmente chamada logo após um fork() para criar processos filhos.  
Utilização: É altamente adequado para identificar processos efêmeros que não são visíveis em ferramentas de monitoramento típicas, como o top e ls. Auxilia na depuração de scripts de shell, no monitoramento da inicialização de aplicativos e na identificação do uso excessivo de recursos sh/grep/sed/awk.  
Saída: Exibe o processo pai (PPID), o ID do processo (PID), o valor de retorno e os argumentos com os quais o programa foi iniciado.

Exemplo:  
```bash
$ sudo execsnoop-bpfcc ou $ sudo /usr/share/bcc/tools/execsnoop

PCOMM            PID    PPID   RET ARGS
curl             211447 208745   0 /usr/bin/curl http://localhost:8080
ls               215745 208745   0 /usr/bin/ls --color=auto -l /tmp/
```
** Possíveis nomes do comando: 
execsnoop; execsnoop-bpfcc

### 2. exitsnoop (Rastreamento do Término do Processo)
Objetivo: Monitorar o término dos processos (finalização do processo).  
Utilização: Ele rastreia a função do kernel sched_process_exit(), o que significa que pode registrar o término de processos por meio de sinais de saída ou fatais. Também pode detectar processos "zumbis"  
Funcionalidades: Funciona para todos os usuários e processos, incluindo aqueles em contêineres.

Exemplo:  
```bash
$ sudo exitsnoop-bpfcc ou $ sudo /usr/share/bcc/tools/exitsnoop

PCOMM            PID    PPID   TID    AGE(s)  EXIT_CODE 
sleep            226314 208745 226314 2.01    0
sleep            227607 208745 227607 1.80    signal 2 (INT)  --> CTRL + C
curl             227660 208745 227660 0.01    0
```

** Possíveis nomes do comando: 
exitsnoop; exitsnoop-bpfcc

### 3. threadsnoop
Objetivo: Rastrear chamadas para pthread_create(), fornecendo informações sobre o caminho de criação de novas threads.  
Utilização: É usado para caracterização de cargas de trabalho e como ferramenta complementar para execsnoop entender como os aplicativos implantam threads.  
Requisitos: Como o código rastreia pthread_create() a partir do diretório raiz libpthread.so.0, pode ser necessário ajustar os caminhos das bibliotecas no código-fonte de acordo com o seu sistema.

### Exemplo:
```bash
$ sudo threadsnoop-bpfcc ou $ sudo /usr/share/bcc/tools/threadsnoop
```

Programa C simples para teste que cria threads.

### Crie teste_threads.c
```bash
cat > teste_threads.c << 'EOF'
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>

void* thread_func(void* arg) {
    sleep(1);
    return NULL;
}

int main() {
    pthread_t threads[10];
    for(int i = 0; i < 10; i++) {
        pthread_create(&threads[i], NULL, thread_func, NULL);
    }
    sleep(2);
    return 0;
}
EOF
```

### Compilando
```bash
gcc -o teste_threads teste_threads.c -lpthread
```
Obs.:
Em algumas distribuições, -lpthread também funciona, mas -pthread é a forma mais recomendada para compilar e linkar programas com threads POSIX.

### Como executar com o threadsnoop
Abra um terminal como monitor e "rode":
```bash
sudo python3 threadsnoop_libc.py
```
O threadsnoop_libc.py usa eBPF/BPF para fazer uprobes (observação de funções de usuário) no kernel. 

Em outro terminal execute o programa que foi gerado, teste_threads.
```bash
./teste_threads
```

### Provável resultado.
Você verá 10 linhas de threads sendo criadas pelo seu programa teste_threads.
```bash
TIME(ms)   PID     COMM             FUNC
56300      719439  teste_threads    thread_func
56300      719439  teste_threads    thread_func
56300      719439  teste_threads    thread_func
56300      719439  teste_threads    thread_func
56300      719439  teste_threads    thread_func
56300      719439  teste_threads    thread_func
56300      719439  teste_threads    thread_func
56300      719439  teste_threads    thread_func
56300      719439  teste_threads    thread_func
56300      719439  teste_threads    thread_func
```

** Importante: Como essas ferramentas usam eBPF, o acesso root (sudo) geralmente é necessário para executar execsnoop, exitsnoope .threadsnoop


### Exemplo prático de execsnoop
Em um terminal execute.
```bash
sudo execsnoop-bpfcc

ou

sudo execsnoop
```

Em um outro terminal execute.
```bash
ls

ou

curl www.ufes.br
```

O execsnoop vai imprimir:
```bash
PCOMM            PID    PPID   RET ARGS
curl             211447 208745   0 /usr/bin/curl http://localhost:8080
ls               215745 208745   0 /usr/bin/ls --color=auto -l /tmp/
```


### 4. opensnoop
Opensnoop é uma ferramenta BCC que rastreia todas as chamadas open() do sistema em tempo real, mostrando qual processo está tentando abrir qual arquivo. É extremamente útil para descobrir onde aplicações procuram configs, logs e arquivos de dados.

Exemplo:  
```bash
$ sudo opensnoop-bpfcc ou $ sudo /usr/share/bcc/tools/opensnoop
TIME     PID    COMM       FD  ERR  PATH
823489 docker              3   0 /etc/ld.so.cache
3615   chrome            349   0 /proc/4163/stat
3615   chrome            349   0 /proc/4078/stat
823499 jcmd                4   0 /home/paulo/.vscode/extensions/redhat.java-1.54.0-linux-x64/jre/21.0.10-linux-x86_64/lib/libjava.so
823499 jcmd                4   0 /home/paulo/.vscode/extensions/redhat.java-1.54.0-linux-x64/jre/21.0.10-linux-x86
```

** Possíveis nomes do comando: 
execsnoop; execsnoop-bpfcc


