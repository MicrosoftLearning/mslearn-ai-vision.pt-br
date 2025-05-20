---
lab:
  title: Analisar vídeo com o Video Indexer
  module: Module 8 - Getting Started with Azure AI Vision
---

# Analisar vídeo com o Video Indexer

Grande parte dos dados criados e consumidos hoje está no formato de vídeo. O **Azure AI Video Indexer** é um serviço alimentado por IA que você pode usar para indexar vídeos e extrair insights a partir deles.

> **Observação**: a partir de 21 de junho de 2022, os recursos dos serviços de IA do Azure que retornam informações de identificação pessoal são restritos aos clientes que receberam [acesso ilimitado](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Sem obter aprovação limitada de acesso, o reconhecimento de pessoas e celebridades com o Video Indexer para esse laboratório não está disponível. Para obter mais detalhes sobre as alterações feitas pela Microsoft e por quê – consulte [Investimentos em IA responsável e proteções para reconhecimento facial](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/).

## Clone o repositório para este curso

Se você clonou o repositório de código **mslearn-ai-vision** para o ambiente em que está trabalhando neste laboratório, abra-o no Visual Studio Code, caso contrário, siga estas etapas para cloná-lo agora.

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute o comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-vision` em uma pasta local (não importa qual pasta).
3. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.
4. Aguarde enquanto os arquivos adicionais são instalados para dar suporte aos projetos de código C# no repositório.

    > **Observação**: se você for solicitado a adicionar ativos necessários para compilar e depurar, selecione **Agora não**.

## Carregar um vídeo no Video Indexer

Primeiro, você precisará entrar no portal do Video Indexer e carregar um vídeo.

> **Dica**: Se a página do Video Indexer estiver lenta para ser carregada no ambiente de laboratório hospedado, use o navegador instalado localmente. Você pode alternar para a VM hospedada para as tarefas posteriores.

1. No navegador, abra o portal do Video Indexer em `https://www.videoindexer.ai`.
2. Se você tiver uma conta existente do Video Indexer, entre. Caso contrário, inscreva-se em uma conta gratuita e entre usando sua conta Microsoft (ou qualquer outro tipo de conta válido). Se tiver dificuldade em iniciar sessão, tente abrir uma sessão privada do navegador.
3. Em uma nova aba, faça o download do vídeo de IA Responsável visitando `https://aka.ms/responsible-ai-video`. Salve o arquivo.
4. No Video Indexer, selecione a opção **Carregar**. Em seguida, selecione a opção **Procurar arquivos**, selecione o vídeo baixado e clique em **Adicionar**. Altere o nome padrão para **IA responsável**, revise a configuração padrão, marque a caixa de seleção para verificar a conformidade com as políticas da Microsoft para reconhecimento facial e carregue o arquivo.
5. Depois que o arquivo for carregado, aguarde alguns minutos enquanto o Video Indexer o indexa automaticamente.

> **Observação**: Neste exercício, estamos usando este vídeo para explorar a funcionalidade do Video Indexer; mas você deve ter tempo para vê-lo na íntegra quando terminar o exercício, pois ele contém informações úteis e diretrizes para desenvolver aplicativos habilitados para IA de forma responsável! 

## Revise os insights do vídeo

O processo de indexação extrai insights do vídeo, que você pode visualizar no portal.

1. No portal do Video Indexer, quando o vídeo for indexado, selecione-o para exibi-lo. Você verá o player de vídeo ao lado de um painel que mostra as informações extraídas do vídeo.

    > **Observação**: devido à política de acesso limitado para proteger identidades individuais, talvez você não veja nomes ao indexar o vídeo.

![Video Indexer com um player de vídeo e painel Insights](../media/video-indexer-insights.png)

2. À medida que o vídeo é reproduzido, selecione a guia **Linha do tempo** para exibir uma transcrição do áudio do vídeo.

![Video Indexer com um player de vídeo e painel Linha do Tempo mostrando a transcrição do vídeo.](../media/video-indexer-transcript.png)

3. No canto superior direito do portal, selecione o símbolo **Exibir** (que se parece com &#128455;) e, na lista de insights para **Transcrição**, selecione **OCR** e **Locutores**.

![Menu de exibição do Video Indexer com Transcrição, OCR e Alto-falantes selecionados](../media/video-indexer-view-menu.png)

4. Observe que o painel **Linha do tempo** agora inclui:
    - Transcrição da narração em áudio.
    - Texto visível no vídeo.
    - Indicações de locutores que aparecem no vídeo. Algumas pessoas conhecidas são automaticamente reconhecidas pelo nome, outras são indicadas pelo número (por exemplo *o locutor #1*).
5. Volte para o painel **Insights** e exiba os insights contidos lá. Elas incluem:
    - Pessoas individuais que aparecem no vídeo.
    - Tópicos discutidos no vídeo.
    - Rótulos para objetos que aparecem no vídeo.
    - Entidades nomeadas, como pessoas e marcas que aparecem no vídeo.
    - Cenas-chave.
6. Com o painel **Insights** visível, selecione o símbolo **exibição** novamente e, na lista de insights, adicione **Palavras-chave** e **Sentimentos** ao painel.

    Os insights encontrados podem ajudá-lo a determinar os principais temas do vídeo. Por exemplo, os **tópicos** deste vídeo mostram que é claramente sobre tecnologia, responsabilidade social e ética.

## Pesquisar insights

Você pode usar o Video Indexer para pesquisar insights no vídeo.

1. No painel **Insights**, na caixa **Pesquisar**, digite *Abelha*. Talvez seja necessário rolar o painel Insights para ver os resultados de todos os tipos de insight.
2. Observe que uma *etiqueta* correspondente é encontrada, com sua localização no vídeo indicada abaixo.
3. Selecione o início da seção onde a presença de uma abelha é indicada e veja o vídeo nesse ponto (você pode precisar pausar o vídeo e selecionar cuidadosamente – a abelha só aparece brevemente.)
4. Desmarque a caixa **Pesquisar** para mostrar todas os insights do vídeo.

![Resultados da pesquisa do Video Indexer para Bee](../media/video-indexer-search.png)

## Usar widgets do Video Indexer

O portal do Video Indexer é uma interface útil para gerenciar projetos de indexação de vídeo. No entanto, pode haver ocasiões em que você deseja disponibilizar o vídeo e seus insights para pessoas que não têm acesso à sua conta do Video Indexer. O Video Indexer fornece widgets que você pode inserir em uma página da Web para essa finalidade.

1. No Visual Studio Code, na pasta **Labfiles/06-video-indexer**, abra **analyze-video.html**. Esta é uma página HTML básica à qual você adicionará os widgets do **Player**e **Insights** do Video Indexer. Observe a referência ao script **vb.widgets.mediator.js** no cabeçalho – esse script permite que vários widgets do Video Indexer na página interajam entre si.
2. No portal do Video Indexer, retorne à página **Arquivos de mídia **e abra o vídeo de ** IA Responsável**.
3. No player de vídeo, selecione **&lt;/&gt; Incorporar** para exibir o código iframe HTML para incorporar os widgets.
4. Na caixa de diálogo **Compartilhar e incorporar**, selecione o widget **Player**, defina o tamanho do vídeo como 560 x 315 e copie o código de inserção para a área de transferência.
5. No Visual Studio Code, no arquivo **analyze-video.html**, cole o código copiado sob o comentário **&lt;-- Widget do Player vai aqui -- &gt;**.
6. De volta à caixa de diálogo **Compartilhar e incorporar**, selecione o widget **Insights** e copie o código de inserção para a área de transferência. Em seguida, feche a caixa de diálogo **Compartilhar e Incorporar**, alterne de volta para o Visual Studio Code e cole o código copiado sob o comentário **&lt;-- o widget Insights vai aqui -- &gt;**.
7. Salve o arquivo. Em seguida, no painel **Explorar**, clique com o botão direito do mouse em **analyze-video.html** e selecione **Revelar no Explorador de Arquivos**.
8. No Explorador de Arquivos, abra o **analyze-video.html** no navegador para ver a página da Web.
9. Experimente os widgets, usando o widget **Insights** para pesquisar insights e ir até eles no vídeo.

![Widgets do Video Indexer em uma página da Web](../media/video-indexer-widgets.png)

## Usar a API REST do Video Indexer

O Video Indexer fornece uma API REST que você pode usar para carregar e gerenciar vídeos em sua conta.

### Obtenha os detalhes da API

Para usar a API do Video Indexer, você precisa de algumas informações para autenticar solicitações:

1. No portal do Video Indexer, expanda o painel esquerdo e selecione a página **Configurações da Conta**.
2. Anote o **ID da conta** nesta página – você precisará dele mais tarde.
3. Abra uma nova guia do navegador e acesse o portal do desenvolvedor do Video Indexer em `https://api-portal.videoindexer.ai`, entrando usando as credenciais para sua conta do Video Indexer.
4. Na página **Perfil**, exiba as **Assinaturas** associadas ao seu perfil.
5. Na página com sua(s) assinatura(s), observe que você recebeu duas chaves (primária e secundária) para cada assinatura. Em seguida, selecione **Mostrar** para qualquer uma das chaves. Você precisará dessa chave em breve.

### Usar a API REST

Agora que você tem o ID da conta e uma chave de API, pode usar a API REST para trabalhar com vídeos em sua conta. Neste procedimento, você usará um script do PowerShell para fazer chamadas REST; mas os mesmos princípios se aplicam com utilitários HTTP, como cURL ou Postman, ou qualquer linguagem de programação capaz de enviar e receber JSON sobre HTTP.

Todas as interações com a API REST do Video Indexer seguem o mesmo padrão:

- Uma solicitação inicial para o método de **token de acesso** com a chave de API no cabeçalho é usada para obter um token de acesso.
- As solicitações subsequentes usam o token de acesso para autenticar ao chamar métodos REST para trabalhar com vídeos.

1. No Visual Studio Code, na pasta **Labfiles/06-video-indexer**, abra **get-videos.ps1**.
2. No script do PowerShell, substitua os espaços reservados **YOUR_ACCOUNT_ID** e **YOUR_API_KEY** pelos valores de ID da conta e chave de API que você identificou anteriormente.
3. Observe que o *local* para uma conta gratuita é "avaliação". Se você tiver criado uma conta irrestrita do Video Indexer (com um recurso do Azure associado), poderá alterá-la para o local onde o recurso do Azure é provisionado (por exemplo, "lestedoseua").
4. Revise o código no script, observando que invoca dois métodos REST: um para obter um token de acesso e outro para listar os vídeos em sua conta.
5. Salve as alterações e, no canto superior direito do painel de script, use o botão **&#9655;** para executar o script.
6. Exiba a resposta JSON do serviço REST, que deve conter detalhes do vídeo **IA responsável** que você indexou anteriormente.

## Mais informações

O reconhecimento de pessoas e celebridades ainda está disponível, mas seguindo o [Padrão de IA responsável](https://aka.ms/aah91ff) eles são restritos por uma política de acesso limitado. Esses recursos incluem identificação facial e reconhecimento de celebridades. Para saber mais e solicitar acesso, consulte o [acesso limitado para serviços de IA do Azure](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access).

Para obter mais informações sobre o **Video Indexer**, consulte a [documentação do Video Indexer](https://learn.microsoft.com/azure/azure-video-indexer/).
