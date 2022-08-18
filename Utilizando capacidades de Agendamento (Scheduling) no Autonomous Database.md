# Utilizando capacidades de Agendamento (Scheduling) no Autonomous Database

Em ambientes de bancos de dados relacionais utilizados para fins analíticos (Data Warehouse e afins), é bastante comum que precisemos executar códigos com uma recorrência pré-definida.
Isso pode ser uma carga de dados para refletir transações que aconteceram durante o dia ou então o Refresh de uma Materialized View.
Neste rápido exemplo mostrarei como criar uma Procedure que realiza uma carga de dados a partir de um arquivo em Object Storage para uma tabela do banco de dados todos os dias à meia noite.

## Criação da Tabela e Credencial

Para que os passos descritos abaixo funcionem perfeitamente, deve-se antes de mais nada deve-se:
- Fazer o download do arquivo csv.
- Carregar o arquivo para o Object Storage.
- Criar um Auth Token na Console OCI.
- Gerar uma credencial dentro do Autonomous Database.
- Criar a tabela que receberá os dados.

Não passarei pelos detalhes desses passos, mas você pode encontrá-los no Workshop de Autonomous Database disponível [no meu github](https://github.com/brenorc/autonomous-analytics/blob/master/Autonomous%20Tutorial.md#loading-data-to-adw).

## Criação da Stored Procedure

Uma stored procedure é um código PL/SQL que fica salvo na forma de uma função, e pode ser executado de uma maneira simples posteriormente, chamando a Procedure diretamente.

Para a criação dela utilizaremos o código abaixo:
```
CREATE OR REPLACE Procedure CargaCSV
AS

BEGIN
 EXECUTE IMMEDIATE 'TRUNCATE TABLE CREDIT_SCORING_100K';
 dbms_cloud.copy_data(
    table_name =>'CREDIT_SCORING_100K',
    credential_name =>'OBJ_STORE_CRED',
    file_uri_list => 'https://objectstorage.us-ashburn-1.oraclecloud.com/n/id3kyspkytmr/b/ArquivosPublicos/o/credit_scoring_example.csv',
    format => json_object('ignoremissingcolumns' value 'true', 'removequotes' value 'true', 'dateformat' value 'YYYY-MM-DD HH24:MI:SS', 'blankasnull' value 'true', 'delimiter' value ',')
 );
END;
```
Pronto! É só isso. Ao executar o código você deve receber uma mensagem como essa aqui:

![](https://i.imgur.com/2Ozkiwu.png)

Isso significa que sua Procedure está salva e agora pode ser chamada através do comando abaixo:
```
EXEC CargaCSV;
```
Para validar se a carga realmente funcionou, podemos verificar quantas linhas existem na tabela CREDIT_SCORING_100K:
```
SELECT COUNT(*) FROM CREDIT_SCORING_100K
```
![](https://i.imgur.com/0sHshHh.png)

Temos 100 mil linhas na tabela. Sinal de que está tudo certo.

## Criando o Scheduler

Agora que nossa Procedure já foi criada, podemos gerar nosso agendamento. Ele pode ser feito diretamente através da Console, através de uma interface gráfica.

Para isso, acesse o seu Autonomous Database e clique em **Database Actions**.

![](https://i.imgur.com/mxfpULn.png)

Dentro do menu, encontre a opção **Scheduling**.

![](https://i.imgur.com/XrN5sDo.png)

Você acessará uma tela com uma visão geral das rotinas que foram criadas nesse banco de dados e se alguma delas apresentou erro nas últimas execuções. Na área superior esquerda, clique no botão **Create Job** para criar um novo planejamento de execução da sua Procedure.

![](https://i.imgur.com/JePqPOk.png)

Selecione seu **Schema** em que seu Job será criada, dê um **Nome** para ele e escolha o tipo **Stored Procedure**. Mantenha a classe como **SYS.DEFAULT_JOB_CLASS**, e encontre sua **Procedure** no **Schema** em que ela foi criada.

![](https://i.imgur.com/CY5lT8K.png)
![](https://i.imgur.com/gkajywe.png)

Em Execution Mode, faremos uma automatização para que essa Procedure seja executada todos os dias às 06h da manhã. Selecione o modo **Repeating**, a **Start e End Date** referentes ao período que desejamos que o Job aconteça, assim como o **Intervalo de Repetição (Daily**)

![](https://i.imgur.com/HHqvRI6.png)

Sempre preste atenção no fuso horário do seu banco Autonomous. O padrão dele é UTC, enquanto em São Paulo estamos em UTC-3. Portanto, executar a rotina às 03h do meu banco significa executá-la à meia noite em São Paulo. Feito tudo isso, clique no botão **Create** para criar o Job. Uma mensagem de sucesso deverá aparecer na sua tela.
No meu caso, aguardei alguns dias e voltei para verificar o progresso do meu job. Na tela pode-se ver uma visão geral de quantas vezes o Job foi executado com sucesso, quantas vezes ocasionou falhas e quando foi a última e será a próxima execução.
Pode-se também executar um job manualmente ou editá-lo quando necessário.

![](https://i.imgur.com/tWXLz69.png)

Ótimo! Chegamos ao nosso resultado final. O mesmo processo de criação do Job poderia ter sido feito pelo código abaixo, mas a interface nos auxilia a não cometer erros e nos oferece uma central de controle para monitorar todos esses Jobs de maneira simples e interativa. Abaixo alguns código úteis que faz sentido conhecer caso você precise interagir com o Job.

## Exemplos com código

**Criar o Job**
```
BEGIN
dbms_scheduler.create_job (
   job_name           =>  'CARGACSV_JOB2',
   job_type           =>  'STORED_PROCEDURE',
   job_action         =>  'CargaCSV',
   start_date         =>  '11-AGO-2022 03:00:00 AM',
   repeat_interval    =>  'FREQ=DAILY',
   enabled            =>  TRUE);
END;
/
```
**Deletar completamente o Job**
```
BEGIN
dbms_scheduler.drop_job ('CARGACSV_JOB2');
END;
/
```
**Executar o Job Manualmente**
```
BEGIN
  DBMS_SCHEDULER.run_job (job_name => 'CARGACSV_JOB2');
END;
```
**Habilitar e Desabilitar um Job**
```
BEGIN
  DBMS_SCHEDULER.Enable ('CARGACSV_JOB2');
END
```
```
BEGIN
  DBMS_SCHEDULER.Disable ('CARGACSV_JOB2');
END
```
