# Execsnoop , Exitsnoop e Threadsnoop.
Execsnoop, exitsnoop e threadsnoop são ferramentas poderosas do BCC (BPF Compiler Collection) usadas para monitorar e depurar sistemas Linux que utilizam o eBPF (Extended Berkeley Packet Filter). Elas fornecem informações em tempo real sobre o comportamento dos processos com sobrecarga mínima.

1. execsnoop (Rastrear Execuções de Processos)
Objetivo: Monitorar e registrar novos processos em tempo real, execve()rastreando a chamada do sistema.
Utilização: É altamente adequado para identificar processos efêmeros que não são visíveis em ferramentas de monitoramento típicas, como o top(1)`ls`. Auxilia na depuração de scripts de shell, no monitoramento da inicialização de aplicativos e na identificação do uso excessivo de recursos sh/grep/sed/awk.
Saída: Exibe o processo pai (PPID), o ID do processo (PID), o valor de retorno e os argumentos com os quais o programa foi iniciado.

2. exitsnoop (Rastreamento do Término do Processo)
Objetivo: Monitorar o término dos processos (finalização do processo).
Utilização: Ele rastreia a função do kernel sched_process_exit(), o que significa que pode registrar o término de processos por meio de sinais de saída ou fatais. Também pode detectar processos "zumbis"
Funcionalidades: Funciona para todos os usuários e processos, incluindo aqueles em contêineres.

3. threadsnoop (Traceer Threadcreatie)
Objetivo: Rastrear chamadas para pthread_create(), fornecendo informações sobre o caminho de criação de novas threads.
Utilização: É usado para caracterização de cargas de trabalho e como ferramenta complementar para execsnoopentender como os aplicativos implantam threads.
Requisitos: Como o código rastreia pthread_create()a partir do diretório raiz libpthread.so.0, pode ser necessário ajustar os caminhos das bibliotecas no código-fonte de acordo com o seu sistema.

Importante: Como essas ferramentas usam eBPF, o acesso root (sudo) geralmente é necessário para executar execsnoop, exitsnoope .threadsnoop

Exemplo prático de execsnoop
Em um terminal execute.
sudo execsnoop

Em um outro terminal execute.
ls ou curl

O execsnoop vai imprimir:

PCOMM            PID    PPID   RET ARGS
curl             211447 208745   0 /usr/bin/curl http://localhost:8080


