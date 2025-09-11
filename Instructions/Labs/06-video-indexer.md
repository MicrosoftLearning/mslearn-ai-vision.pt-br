---
lab:
  title: Analisar vídeo
  description: Use o Azure AI Video Indexer para analisar um vídeo.
---

# Analisar vídeo

Grande parte dos dados criados e consumidos hoje está no formato de vídeo. O **Azure AI Video Indexer** é um serviço alimentado por IA que você pode usar para indexar vídeos e extrair insights a partir deles.

> **Observação**: a partir de 21 de junho de 2022, os recursos dos serviços de IA do Azure que retornam informações de identificação pessoal são restritos aos clientes que receberam [acesso ilimitado](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Sem obter aprovação limitada de acesso, o reconhecimento de pessoas e celebridades com o Video Indexer para esse laboratório não está disponível. Para obter mais detalhes sobre as alterações feitas pela Microsoft e por quê – consulte [Investimentos em IA responsável e proteções para reconhecimento facial](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/).

## Carregar um vídeo no Video Indexer

Primeiro, você precisará entrar no portal do Video Indexer e carregar um vídeo.

1. No navegador, abra o [portal do Video Indexer](https://www.videoindexer.ai) em `https://www.videoindexer.ai`.
1. Se você tiver uma conta existente do Video Indexer, entre. Caso contrário, inscreva-se em uma conta gratuita e entre usando sua conta Microsoft (ou qualquer outro tipo de conta válido). Se tiver dificuldade em iniciar sessão, tente abrir uma sessão privada do navegador.

    > Observação: Se esta for a primeira vez que você faz login, você poderá ver um formulário pop-up solicitando que você verifique como usará o serviço. 

1. Em uma nova aba, faça o download do vídeo de IA Responsável visitando `https://aka.ms/responsible-ai-video`. Salve o arquivo.
1. No Video Indexer, selecione a opção **Carregar**. Em seguida, selecione a opção **Procurar arquivos**, selecione o vídeo baixado e clique em **Adicionar**. Altere o texto no campo **Nome do arquivo** para **IA responsável**. Selecione **Examinar + carregar**, examine a visão geral do resumo, marque a caixa de seleção para verificar a conformidade com as políticas da Microsoft para reconhecimento facial e carregue o arquivo.
1. Depois que o arquivo for carregado, aguarde alguns minutos enquanto o Video Indexer o indexa automaticamente.

> **Observação**: Neste exercício, estamos usando este vídeo para explorar a funcionalidade do Video Indexer; mas você deve ter tempo para vê-lo na íntegra quando terminar o exercício, pois ele contém informações úteis e diretrizes para desenvolver aplicativos habilitados para IA de forma responsável! 

## Revise os insights do vídeo

O processo de indexação extrai insights do vídeo, que você pode visualizar no portal.

1. No portal do Video Indexer, quando o vídeo for indexado, selecione-o para exibi-lo. Você verá o player de vídeo ao lado de um painel que mostra as informações extraídas do vídeo.

    > **Observação**: devido à política de acesso limitado para proteger identidades individuais, talvez você não veja nomes ao indexar o vídeo.

    ![Video Indexer com um player de vídeo e painel Insights](../media/video-indexer-insights.png)

1. À medida que o vídeo é reproduzido, selecione a guia **Linha do tempo** para exibir uma transcrição do áudio do vídeo.

    ![Video Indexer com um player de vídeo e painel Linha do Tempo mostrando a transcrição do vídeo.](../media/video-indexer-transcript.png)

1. No canto superior direito do portal, selecione o símbolo **Exibir** (que se parece com &#128455;) e, na lista de insights para **Transcrição**, selecione **OCR** e **Locutores**.

    ![Menu de exibição do Video Indexer com Transcrição, OCR e Alto-falantes selecionados](../media/video-indexer-view-menu.png)

1. Observe que o painel **Linha do tempo** agora inclui:
    - Transcrição da narração em áudio.
    - Texto visível no vídeo.
    - Indicações de locutores que aparecem no vídeo. Algumas pessoas conhecidas são automaticamente reconhecidas pelo nome, outras são indicadas pelo número (por exemplo *o locutor #1*).
1. Volte para o painel **Insights** e exiba os insights contidos lá. Elas incluem:
    - Pessoas individuais que aparecem no vídeo.
    - Tópicos discutidos no vídeo.
    - Rótulos para objetos que aparecem no vídeo.
    - Entidades nomeadas, como pessoas e marcas que aparecem no vídeo.
    - Cenas-chave.
1. Com o painel **Insights** visível, selecione o símbolo **exibição** novamente e, na lista de insights, adicione **Palavras-chave** e **Sentimentos** ao painel.

    Os insights encontrados podem ajudá-lo a determinar os principais temas do vídeo. Por exemplo, os **tópicos** deste vídeo mostram que é claramente sobre tecnologia, responsabilidade social e ética.

## Pesquisar insights

Você pode usar o Video Indexer para pesquisar insights no vídeo.

1. No painel **Insights**, na caixa **Pesquisar**, digite *Abelha*. Talvez seja necessário rolar o painel Insights para ver os resultados de todos os tipos de insight.
1. Observe que uma *etiqueta* correspondente é encontrada, com sua localização no vídeo indicada abaixo.
1. Selecione o início da seção onde a presença de uma abelha é indicada e veja o vídeo nesse ponto (você pode precisar pausar o vídeo e selecionar cuidadosamente – a abelha só aparece brevemente.)

    ![Resultados da pesquisa do Video Indexer para Bee](../media/video-indexer-search.png)

1. Desmarque a caixa **Pesquisar** para mostrar todas os insights do vídeo.

## Usar a API REST do Video Indexer

O Video Indexer fornece uma API REST que você pode usar para carregar e gerenciar vídeos em sua conta.

1. Em uma nova guia do navegador, abra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` e entre usando suas credenciais do Azure. Mantenha a guia existente com o portal do Video Indexer aberto.
1. No portal do Azure, use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell*** sem armazenamento em sua assinatura.

    O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

    > **Observação**: Se o portal solicitar que você selecione um armazenamento para persistir seus arquivos, escolha **Nenhuma conta de armazenamento necessária**, selecione a assinatura que você está usando e pressione **Aplicar**.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. Redimensione o painel do Cloud Shell para conseguir ver mais dele.

    > **Dica**" Você pode redimensionar o painel arrastando a borda superior. Você também pode usar os botões minimizar e maximizar para alternar entre o Cloud Shell e a interface do portal principal.

1. No painel do Cloud Shell, insira os seguintes comandos para clonar o repositório GitHub que contém os arquivos de código para este exercício (digite o comando ou copie-o para a área de transferência e clique com o botão direito do mouse na linha de comando e cole como texto sem formatação):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Dica**: conforme você colar comandos no cloudshell, a saída pode ocupar uma grande quantidade do espaço da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. Após o repositório ser clonado, navegue até a pasta que contém o arquivo de código do aplicativo para este exercício:  

    ```
   cd mslearn-ai-vision/Labfiles/video-indexer
    ```

### Obtenha os detalhes da API

Para usar a API do Video Indexer, você precisa de algumas informações para autenticar solicitações:

1. No portal do Video Indexer, expanda o painel esquerdo e selecione a página **Configurações da Conta**.
1. Anote o **ID da conta** nesta página – você precisará dele mais tarde.
1. Abra uma nova guia do navegador e acesse o [portal do Desenvolvedor do Video Indexer](https://api-portal.videoindexer.ai) `https://api-portal.videoindexer.ai, fazendo login com suas credenciais do Azure.
1. Na página **Perfil**, exiba as **Assinaturas** associadas ao seu perfil.
1. Na página com sua(s) assinatura(s), observe que você recebeu duas chaves (primária e secundária) para cada assinatura. Em seguida, selecione **Mostrar** para qualquer uma das chaves. Você precisará dessa chave em breve.

### Usar a API REST

Agora que você tem o ID da conta e uma chave de API, pode usar a API REST para trabalhar com vídeos em sua conta. Neste procedimento, você usará um script do PowerShell para fazer chamadas REST; mas os mesmos princípios se aplicam com utilitários HTTP, como cURL ou Postman, ou qualquer linguagem de programação capaz de enviar e receber JSON sobre HTTP.

Todas as interações com a API REST do Video Indexer seguem o mesmo padrão:

- Uma solicitação inicial para o método de **token de acesso** com a chave de API no cabeçalho é usada para obter um token de acesso.
- As solicitações subsequentes usam o token de acesso para autenticar ao chamar métodos REST para trabalhar com vídeos.

1. No Cloud Shell, use o seguinte comando para abrir o script do PowerShell:

    ```
   code get-videos.ps1
    ```
    
1. No script do PowerShell, substitua os espaços reservados **YOUR_ACCOUNT_ID** e **YOUR_API_KEY** pelos valores de ID da conta e chave de API que você identificou anteriormente.
1. Observe que o *local* para uma conta gratuita é "avaliação". Se você tiver criado uma conta irrestrita do Video Indexer (com um recurso do Azure associado), poderá alterá-la para o local onde o recurso do Azure é provisionado (por exemplo, "lestedoseua").
1. Revise o código no script, observando que invoca dois métodos REST: um para obter um token de acesso e outro para listar os vídeos em sua conta.
1. Salve suas alterações (pressione *CTRL+S*), feche o editor de código (pressione *CTRL+Q*) e execute o seguinte comando para executar o script:

    ```
   ./get-videos.ps1
    ```
    
1. Exiba a resposta JSON do serviço REST, que deve conter detalhes do vídeo **IA responsável** que você indexou anteriormente.

## Usar widgets do Video Indexer

O portal do Video Indexer é uma interface útil para gerenciar projetos de indexação de vídeo. No entanto, pode haver ocasiões em que você deseja disponibilizar o vídeo e seus insights para pessoas que não têm acesso à sua conta do Video Indexer. O Video Indexer fornece widgets que você pode inserir em uma página da Web para essa finalidade.

1. Use o comando `ls` para exibir o conteúdo da pasta **video-indexer**. Observe que ele contém um arquivo **analyze-video.html**. Esta é uma página HTML básica à qual você adicionará os widgets do **Player**e **Insights** do Video Indexer.
1. Insira o seguinte comando para editar o arquivo:

    ```
   code analyze-video.html
    ```

    O arquivo é aberto em um editor de código.
   
1. Observe a referência ao script **vb.widgets.mediator.js** no cabeçalho – esse script permite que vários widgets do Video Indexer na página interajam entre si.
1. No portal do Video Indexer, retorne à página **Arquivos de mídia **e abra o vídeo de ** IA Responsável**.
1. No player de vídeo, selecione **&lt;/&gt; Incorporar** para exibir o código iframe HTML para incorporar os widgets.
1. Na caixa de diálogo **Compartilhar e incorporar**, selecione o widget **Player**, defina o tamanho do vídeo como 560 x 315 e copie o código de inserção para a área de transferência.
1. No Cloud Shell do Azure, no editor de código do arquivo **analyze-video.html**, cole o código copiado abaixo do comentário **&lt;-- Player widget goes here -- &gt;**.
1. De volta ao portal do Video Indexer, na caixa de diálogo **Compartilhar e Inserir**, selecione o widget **Insights** e copie o código de inserção para a área de transferência. Em seguida, feche a caixa de diálogo **Compartilhar e Inserir**, volte para o portal do Azure e cole o código copiado abaixo do comentário **&lt;-- Insights widget goes here -- &gt;**.
1. Após editar o arquivo, no editor de código, salve suas alterações (*CTRL+S*) e depois feche o editor de código (*CTRL+Q*), mantendo a linha de comando do Cloud Shell aberto.
1. Na barra de ferramentas do Cloud Shell, digite o seguinte comando (específico do Cloud Shell) para baixar o arquivo HTML que você editou:

    ```
    download analyze-video.html
    ```

    O comando download cria um link pop-up no canto inferior direito do seu navegador, que você pode selecionar para baixar e abrir o arquivo, que deve se parecer com isso:

    ![Widgets do Video Indexer em uma página da Web](../media/video-indexer-widgets.png)

1. Experimente os widgets, usando o widget **Insights** para pesquisar insights e ir até eles no vídeo.

