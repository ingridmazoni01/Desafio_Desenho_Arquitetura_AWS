# Desafio_Desenho_Arquitetura_AWS
Desafio da DIO na Aula de Gerenciamento de instâncias EC2 na AWS.


Arquitetura Proposta para Gerenciar um Sistema de Vendas de uma Rede de Supermercados 

- As lojas enviam arquivos de vendas diarias (ex: CSV) para um bucket S3 (ex: `supermarket-sales-data`).
- Quando um novo arquivo é carregado no S3, um evento é disparado para uma Lambda Function.
- A Lambda Function processa o arquivo: lê o CSV, valida os dados, transforma e insere em um banco de dados (que está configurado na instância EC2).
- A instância EC2 hospeda a aplicação principal (API) e o banco de dados (MySQL). O EBS é usado como disco para a instância, armazenando o SO, a aplicação e os dados do banco.
- A aplicação na EC2 também pode gerar relatórios sob demanda ou agendados, que são então salvos em outro bucket S3 (ex: `supermarket-reports`).


Detalhamento:

1. **Upload de Arquivos**:

   - Cada supermercado envia um arquivo CSV com as vendas diárias para o bucket S3 `supermarket-sales-data` via AWS CLI, SDK ou através de uma interface web.

2. **Trigger do Lambda**:

   - O bucket S3 está configurado para disparar um evento para a Lambda Function sempre que novos arquivos são adicionados
   - A Lambda Function é acionada com o evento que contém informações do arquivo (ex: nome do bucket, chave do objeto).
  
3. **Processamento com Lambda**:

   - A Lambda é acionada pelo S3 ao detectar um novo arquivo.
   - A Lambda Function lê o arquivo CSV do S3.
   - Validar integridade dos dados , aplica regras de negócio e transforma os dados (ex: converte datas, formata valores).
   - Conecta-se ao banco de dados MySQL rodando na instância EC2 
   - Insere os dados no banco.

4. **Servidor de Aplicação com EC2**:
    
    Função: Host da aplicação principal que gerencia o sistema do supermercado (backend) 
 
    - A instância EC2 roda uma aplicação Java que fornece endpoints para:
        - Consulta de vendas por período, loja, etc.
        - Gestão de estoque (atualização de estoque a partir da análise dos relatórios de vendas diárias ,  
        - A aplicação também pode gerar relatórios em PDF ou CSV, que são salvos no bucket  S3`supermarket-reports`.

        Fluxo:
        - Funcionários acessam a aplicação via navegador ou cliente desktop.
        - A aplicação consome dados processados do S3
        - Dados críticos (ex: transações em tempo real) são salvos em um banco de dados instalado na própria EC2.
        - Funcionários geram relatórios ou visualizam dashboards.

5. **EBS**:
   - O EBS está attached à instância EC2 e armazena:
        - Sistema operacional.
        - Aplicação e suas dependências.
        - Dados temporários e caches
        - Banco de dados MySQL (dados e logs).

4. **Backup e Recuperação**:
    - Snapshots do EBS são criados diariamente (Backup do EBS)
    - S3 Versioning: Mantém versões anteriores dos arquivos para auditoria e recuperação e os dados do S3 são replicados para outra região (Cross-Region Replication).

5. **Monitoramento com CloudWatch**:
    - Lambda e EC2: Métricas de desempenho (ex: tempo de execução, uso de CPU).
    - Alertas: Notificações para falhas no processamento de dados ou indisponibilidade.        


        