# Aula 04 - Modelo de Processos: Pipeline de Dados Simplificado

## 1. Process Inventory
| Componente | Executa como | Iniciado por | Perfil | Recurso crítico | Se morrer... | Controle |
|---|---|---|---|---|---|---|
| Script de Ingestão (`ingest.py`) | Processo / Script Python | Cron / Terminal | I/O-bound | Rede / Internet | Os dados novos não são baixados da API | Logs de erro + Rerun manual/agendado |
| Script de Tratamento (`transform.py`) | Processo / Script Python | Script principal / Agendador | CPU-bound | Memória / CPU | A base não é limpa nem transformada | Try/Except com log de falha |
| Banco de Dados (SQLite / PostgreSQL) | Serviço / Daemon | Sistema / Docker | I/O-bound | Disco / Memória | As consultas e o dashboard ficam fora do ar | Auto-restart pelo Docker/Sistema |
| Painel Visual (`app.py` / Streamlit) | Serviço / Processo Web | Terminal / Script | I/O-bound | Memória | O usuário não consegue ver os gráficos | Restart manual do script |

## 2. Ciclo de Vida Simplificado
- **Script de Ingestão (`ingest.py`)**:
  - *Início*: Executado via linha de comando ou tarefa agendada (Cron).
  - *Saúde*: Gera um arquivo `.log` confirmando a execução com sucesso.
  - *Encerramento*: Encerra sozinho assim que termina de baixar o arquivo/dados.
  - *Detecção de Falha*: Retorna um código de erro no terminal e grava no log.
  - *Reinicialização*: Rodado novamente na próxima execução agendada ou manualmente.
  - *Logs*: Mensagem de "Sucesso ao baixar X registros" ou "Erro de conexão".

- **Painel Visual (`app.py`)**:
  - *Início*: Executado no terminal com `python app.py` ou `streamlit run app.py`.
  - *Saúde*: O painel abre no navegador no endereço `localhost:8501`.
  - *Encerramento*: Teclas `Ctrl + C` no terminal.
  - *Detecção de Falha*: A página não carrega no navegador ou o processo fecha no terminal.
  - *Reinicialização*: Executar o comando no terminal novamente.
  - *Logs*: Exibidos diretamente no terminal onde o script está rodando.

## 3. Hipótese de Falha
**Cenário**: O script de ingestão falha ao tentar baixar os dados porque a API/site de origem mudou ou caiu.

1. **Sintoma para o usuário**: O painel exibe dados antigos ou fica em branco.
2. **Evidência no SO/aplicação**: O arquivo `ingest.log` mostra o erro `ConnectionError` ou `HTTP 500`.
3. **Ação de recuperação**: Rodar o script novamente após a conexão voltar ou ajustar a URL da API no arquivo de configuração.
4. **Risco ao reiniciar**: Se o script for reiniciado repetidamente sem conexão, pode gerar arquivos incompletos de dados.

## 4. Decisões da Sprint
- **Decisão 1**: Salvar os dados intermediários em arquivos CSV/SQLite locais para facilitar os testes.
- **Decisão 2**: Usar blocos `try/except` básicos no Python para registrar mensagens claras em um arquivo de log simples.
- **Dívida técnica**: Criar um aviso na tela quando os dados do banco estiverem desatualizados.