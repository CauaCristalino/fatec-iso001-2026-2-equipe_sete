# Arquitetura

## Visão geral
Pipeline batch diário com três etapas (extract, transform, load), executado
em um runner efêmero do GitHub Actions, gravando em um MySQL gerenciado.
Um agente RAG consulta o banco e usa um modelo Llama para responder.

Diagramas: [contexto](../diagrams/contexto.mmd) · [containers](../diagrams/containers.mmd)

## Componentes
| Componente | Tecnologia | Onde roda | Função |
|---|---|---|---|
| Agendador | GitHub Actions (cron) | Nuvem do GitHub | Dispara o pipeline às 7h |
| extract.py | Python + requests | Runner | Coleta dados da Open-Meteo |
| transform.py | Python + pandas | Runner | Padroniza e calcula média |
| load.py | mysql-connector | Runner | Upsert no banco |
| main.py | Python | Runner | Orquestra as etapas e define o código de saída |
| Banco clima | MySQL (Aiven) | Nuvem Aiven | Armazena o histórico |
| agent.py + retrieval.py | Python + GitHub Models | Máquina do usuário | Responde perguntas |
| Artefatos | GitHub Actions | Nuvem do GitHub | Guarda JSON bruto e backups |

## Fluxo principal
1. O cron do Actions cria um runner e inicia o processo main.py.
2. extract.py faz requisições HTTPS e salva o JSON em data/raw.
3. transform.py gera um DataFrame com as colunas padronizadas.
4. load.py grava no MySQL com ON DUPLICATE KEY UPDATE.
5. O processo termina com código 0 (sucesso) ou 1 (falha).
6. O runner é destruído; JSON e logs ficam como artefato.

## Processos e serviços
| Componente | Tipo | Duração |
|---|---|---|
| main.py | Processo batch | Curta (segundos) |
| MySQL | Serviço (daemon) gerenciado | Contínua |
| agent.py | Processo interativo | Enquanto o usuário usa |
| Agendador do Actions | Serviço externo | Contínua |

## Riscos operacionais
| Risco | Causa | Conceito de SO |
|---|---|---|
| Dia sem dados | API ou rede indisponível | E/S de rede, processos |
| Dados duplicados | Duas execuções simultâneas | Concorrência, exclusão mútua |
| Job travado | Requisição sem timeout | Estados de processo (bloqueado) |
| Perda de histórico | Banco perdido sem backup | Armazenamento, backup |
| Vazamento de credenciais | Senha no código | Permissões, segredos |
| Respostas inventadas | Contexto vazio ou grande demais | Memória, limites |
