# Trabalhando com eText Templates no OAC

### Como gerar arquivos .txt no BI Publisher e alterar sua formatação

Recentemente eu me deparei com esse desafio em um dos clientes que utilizam o Oracle Analytics Cloud: eles precisavam gerar arquivos .txt através do BI Publisher que respeitassem uma formatação específica definida por eles.
O BI Publisher possui uma série de outputs padrão: Excel, PDF, CSV, entre outros, mas o .txt não é uma dessas opções que pode ser habilitada apenas com um click.
Para gerar esse tipo de output precisaremos utilizar eText Templates, que são modelos de formatação baseados em um arquivo .rtf e em um .xml que representa a estrutura de dados.

### Passo 1 - Gerando o XML da Query

Assumirei aqui que o seu ambiente está configurado e uma conexão com uma fonte de dados já é existente. Caso isso não seja verdade, certifique-se de criá-la na área de administração do ambiente.

Para gerar o XML é necessário primeiro criar um Data Model. Vá ate a opção **Create** do Menu e selecione **Data Model**.

![](https://i.imgur.com/DN36HC4.png)

Feito isso, construa sua query selecionando as colunas necessárias para seu report e alterando a configuração de cada uma delas de acordo com sua necessidade. No meu exemplo, usarei um simples SELECT * FROM TABLE.

![](https://i.imgur.com/ryQfSEt.png)

Com sua query pronta, valide o resultado na aba **Data**. Clickando em **View** você verá o resultado da query, e caso o comportamento esteja de acordo com sua expectativa, seleciona **Export** para gerar o arquivo XML. 
Você pode ver meu arquivo gerado no print abaixo.

![](https://i.imgur.com/hLXOPqx.png)

Não se esqueça de salvar o resultado como Sample Data e **salvar o seu Data Model**. Ele será necessário nos passos seguintes.

### Passo 2 - Instalando o Template Builder

Uma vez com o XML em mãos, podemos criar o nosso modelo de relatório. Para simplificar esse processo, utilizaremos uma ferramenta disponibilizada pela própria Oracle: O Template Builder for Word. Você pode fazer o download dele na própria página inicial do OAC ou então [nesse link](https://www.oracle.com/middleware/technologies/analytics-publisher.html).
Certifique-se de utilizar uma versão compatível com seu Microsoft Office (32 ou 64 bits)

![](https://i.imgur.com/AkqG9QC.png)

Uma vez instalada, a ferramenta fica disponível como uma aba no Microsoft Word, e auxilia na criação e edição de arquivos RTF.

![](https://i.imgur.com/yNUUZQh.png)

Essa ferramenta nos auxiliará com dois recursos extremamente importantes para a criação do nosso arquivo RTF. O primeiro deles é um diretório com alguns samples de RTF para geração de arquivos texto, que fica localizado em:

C:\Program Files (x86)\Oracle\Oracle Analytics Publisher\Oracle Analytics Publisher Desktop\Template Builder for Word\samples

![](https://i.imgur.com/uCo5Pq7.png)

E um visualizador de Templates que simulará o resultado, localizado em: 

C:\Program Files (x86)\Oracle\Oracle Analytics Publisher\Oracle Analytics Publisher Desktop\TemplateViewer\tmplviewer.jar

![](https://i.imgur.com/zUX2IC6.png)

### Passo 3 - Formatando o arquivo RTF

A formatação do arquivo RTF vai depender muito da entrada e da saída do seu caso específico, portanto pode variar muito. O processo com o qual obtive maior sucesso é modificar os samples para adequar ao XML gerado pelo modelo de dados.

A primeira tabela do RTF define algumas diretrizes gerais, como se os seus registros serão divididos baseados em um delimitador ou posição fixa, e o charset.

![](https://i.imgur.com/sh26uzG.png)

Nosso XML possui dois níveis: DATA_DS e G_1, logo teremos que criar duas tabelas no nosso arquivo RTF.

![](https://i.imgur.com/4R4vXaS.png)

A primeira delas trará o cabeçalho do nosso arquivo, portanto deixei os valores fixos com os nomes das colunas.

![](https://i.imgur.com/5LoBcb9.png)

A segunda tabela traz em ordem o conteúdo de cada uma das colunas, e uma vírgula como delimitador dividindo os registros.

![](https://i.imgur.com/vCWWd9L.png)

Salve seu arquivo RTF e execute ele junto com o XML gerado no passo 1. Não se esqueça de deixar o output format como eText. Clicke em **Start Processing** e o arquivo TXT deverá ser criado. Valide o formato e atualize o RTF se necessário.

![](https://i.imgur.com/WsWS7Bh.png)

Esses exemplos são extremamente simples, mas é possível incluir regras de formatação mais sofisticadas, como condicionais ou cálculos matemáticos. Se precisar se aventurar nesses aspectos, recomendo [este vídeo](https://www.youtube.com/watch?v=bnSXRO9woVM) 

![](https://i.imgur.com/rsz0NhU.png)
![](.pastes\2022-04-29-10-39-06.png)

Você pode fazer download dos arquivos gerados por mim para esse exemplo [neste link](https://objectstorage.sa-saopaulo-1.oraclecloud.com/n/grri30nzv1ul/b/MateriaisTreinamento/o/eTextExemplo.rar) 

### Passo 4 - Criando o relatório no OAC

Uma vez que temos o arquivo RTF, apenas nos resta criar esse relatório no OAC. O processo é bastante simples.

Clicke em **Create > Report**. Feche a janela que aparece na tela, ela é um wizard para criação de relatórios interativos, e não nos deixará gerar outputs TXT.

![](https://i.imgur.com/Z2mzcg8.png)

Clicke na lupa no superior esquerdo e selecione o modelo de dados salvo no passo 1. 

![](https://i.imgur.com/B5875Mn.png)

Selecione **Upload or Generate Layout**

![](https://i.imgur.com/3AvQSsz.png)

Suba seu arquivo RTF, selecione Type eText Template e a língua utilizada, antes de clicar em **Upload**.

![](https://i.imgur.com/3kLD6ox.png)

Você deverá ver uma tela assim como a disponível abaixo. Salve e vá para **View Report** para visualizar seu relatório. 

![](https://i.imgur.com/tOckC5X.png)

Seu relatório será gerado com a formatação indicada no arquivo RTF. 

![](https://i.imgur.com/jQlmwFX.png)

Pronto! Agora você possui um relatório que será gerado com output TXT. Você pode exportar o resultado para fazer download ou então criar um Schedule para que ele seja realizado com determinada recorrência.
