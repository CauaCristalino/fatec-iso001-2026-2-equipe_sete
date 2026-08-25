1. Qual é a versão do kernel? Qual distribuição aparece?
Resposta: A versão do kernel do Linux em execução é a 6.8.0-1052-azure (um kernel customizado para a nuvem Microsoft Azure)
A distribuição instalada no ambiente é o Ubuntu 24.04.4 LTS (Noble Numbat)

2. Quanto de memória RAM está disponível no ambiente? Quanto de filesystem está disponível?
Resposta: Memória RAM: O sistema possui um total de 7.8 GiB de RAM, com 5.7 GiB disponíveis (e cerca de 6.0 GiB sendo usados de forma inteligente pelo SO como buffer/cache)
Não há memória Swap configurada (0B)
Filesystem (Disco): O sistema de arquivos principal (/ em overlay) possui 32 GiB de tamanho total, contendo 21 GiB livres (disponíveis) para uso.

3. Quais processos chamaram atenção no baseline?
Resposta: Os processos de maior consumo de CPU e Memória são instâncias chamadas MainThread (atreladas ao próprio VS Code Server / Codespaces, como o PID 2255 consumindo 3.3% de CPU e 4.3% de RAM)
Além deles, destacam-se os processos de segundo plano do ecossistema de containers, como o dockerd (PID 257) e o containerd (PID 569), que rodam como usuário administrador (root).

4. Para o cenário da squad, qual recurso tende a ser mais crítico: CPU, memória, armazenamento, rede ou outro? Justifique.
Resposta: Como estamos rodando em uma máquina virtual leve de apenas 2 CPUs e 7.8 GiB de RAM, a memória RAM tende a ser o recurso mais crítico. Como não há memória de troca virtual configurada, se a squad subir simultaneamente múltiplos containers pesados (por exemplo, uma API backend, um banco de dados relacional e um broker de mensageria), o consumo pode facilmente estourar o limite físico disponível e fazer com que o mecanismo de proteção do Linux (o OOM Killer) encerre abruptamente os processos da aplicação para salvar o sistema operacional.

