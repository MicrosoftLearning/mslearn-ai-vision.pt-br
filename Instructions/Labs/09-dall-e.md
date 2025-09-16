---
lab:
  title: Gerar imagens com IA
  description: Use um modelo DALL-E do OpenAI na Fábrica de IA do Azure para gerar imagens.
---

# Gerar imagens com IA

Neste exercício, você usará o modelo de IA generativa OpenAI DALL-E para gerar imagens. Você também usa o SDK do OpenAI para Python para criar um aplicativo simples para gerar imagens com base em seus prompts.

> **Observação**: Este exercício é baseado em software de SDK pré-lançamento, que pode estar sujeito a alterações. Quando necessário, usamos versões específicas de pacotes que podem não refletir as versões mais recentes disponíveis. Você pode experimentar algum comportamento inesperado, avisos ou erros.

Embora este exercício seja baseado no SDK do OpenAI para Python, você pode desenvolver aplicativos de chat de IA usando vários SDKs específicos de linguagem; incluindo:

* [Projetos de OpenAI para Microsoft .NET](https://www.nuget.org/packages/OpenAI)
* [Projetos de OpenAI para JavaScript](https://www.npmjs.com/package/openai)

Este exercício levará aproximadamente **30** minutos.

## Abra o portal do Azure AI Foundry

Vamos começar entrando no portal da Fábrica de IA do Azure.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica de IA do Azure** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir (feche o painel **Ajuda** se estiver aberto):

    ![Captura de tela do portal do Azure AI Foundry.](./media/ai-foundry-home.png)

1. Revise as informações na home page.

## Escolher um modelo para iniciar um projeto

Um *projeto* da IA do Azure fornecerá um workspace colaborativo para desenvolvimento de IA. Vamos começar escolhendo um modelo com o qual queremos trabalhar e criando um projeto para usá-lo.

> **Observação**: os projetos da Fábrica de IA podem ser baseados em um recurso da *Fábrica de IA do Azure*, que fornece acesso a modelos de IA (incluindo o OpenAI do Azure), aos Serviços de IA do Azure e outros recursos para desenvolver agentes de IA e soluções de chat. Como alternativa, os projetos podem ser baseados em recursos do *hub de IA*, que incluem conexões com recursos do Azure para armazenamento seguro, computação e ferramentas especializadas. Os projetos baseados na Fábrica de IA do Azure são ótimos para desenvolvedores que desejam gerenciar recursos para o desenvolvimento de aplicativos de chat ou agente de IA. Os projetos baseados em hub de IA são mais adequados para equipes de desenvolvimento empresarial que trabalham em soluções de IA complexas.

1. Na home page, na seção **Explorar modelos e recursos**, pesquise pelo modelo `dall-e-3`, que usaremos em nosso projeto.

1. Nos resultados da pesquisa, selecione o modelo **dall-e-3** para ver os detalhes e, na parte superior da página do modelo, clique em **Usar este modelo**.

1. Quando solicitado a criar um projeto, insira um nome válido para o projeto e expanda **Opções avançadas**.

1. Selecione **Personalizar** e especifique as seguintes configurações para o hub:
    - **Recurso da Fábrica de IA do Azure**: *um nome válido para o recurso da Fábrica de IA do Azure*
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Região**: *selecione qualquer **AI Foundry recomendado***\*

    > \* Alguns recursos da IA do Azure são restritos por cotas de modelo regional. Caso um limite de cota seja excedido posteriormente no exercício, é possível que você precise criar outro recurso em uma região diferente.

1. Clique em **Criar** e aguarde a criação do projeto, incluindo a implantação do modelo dall-e-3 selecionado.

    > Observação: Dependendo da seleção do modelo, você pode receber solicitações adicionais durante o processo de criação do projeto. Concorde com os termos e finalize a implantação.

1. Quando o projeto for criado, seu modelo será exibido na página **Modelos + pontos de extremidade**.

## Testar o modelo no playground

Antes de criar um aplicativo cliente, vamos testar o modelo DALL-E no playground.

1. Selecione **Playgrounds** e, em seguida, **Playground de Imagens**.

1. Certifique-se de que a implantação do modelo DALL-E esteja selecionada. Em seguida, na caixa próxima à parte inferior da página, insira um prompt como `Create an image of an robot eating spaghetti` e selecione **Gerar**.

1. Revise a imagem resultante no playground:

    ![Captura de tela do playground de imagens com uma imagem gerada.](../media/images-playground.png)

1. Insira um prompt de acompanhamento, como `Show the robot in a restaurant`, e revise a imagem resultante.

1. Continue testando novos prompts para refinar a imagem até estar contente com o resultado. 

1. Selecione o botão **\</\>Exibir Código** e verifique se você está na guia **Autenticação do Entra ID**. Em seguida, registre as seguintes informações para uso posteriormente no exercício. Observe que os valores são exemplos, certifique-se de registrar as informações de sua implantação.

    * Ponto de extremidade OpenAI: *https://dall-e-aus-resource.cognitiveservices.azure.com/*
    * Versão da API do OpenAI: *2024-04-01-preview*
    * Nome da implantação (nome do modelo): *dall-e-3*

## Criar um aplicativo cliente

O modelo parece funcionar no playground. Agora você pode usar o SDK do OpenAI para usá-lo em um aplicativo cliente.

### Preparar a configuração de aplicativo

1. Abra uma nova guia do navegador (mantendo o portal da Fábrica de IA do Azure aberto na guia existente). Em seguida, na nova guia, navegue até o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`; efetue login com suas credenciais do Azure, se solicitado.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

    > **Observação**: Se o portal solicitar que você selecione um armazenamento para persistir seus arquivos, escolha **Nenhuma conta de armazenamento necessária**, selecione a assinatura que você está usando e pressione **Aplicar**.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. No painel do Cloud Shell, insira os seguintes comandos para clonar o repositório GitHub que contém os arquivos de código para este exercício (digite o comando ou copie-o para a área de transferência e clique com o botão direito do mouse na linha de comando e cole como texto sem formatação):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Dica**: conforme você colar comandos no cloudshell, a saída pode ocupar uma grande quantidade do espaço da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. Após o repositório ser clonado, navegue até a pasta que contém os arquivos de código do aplicativo:  

    ```
   cd mslearn-ai-vision/Labfiles/dalle-client/python
    ```

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity openai requests
    ```

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. Substitua os espaços reservados **your_endpoint**, **your_model_deployment** e **your_api_version** pelos valores que você anotou no **playground de Imagens**.

1. Depois de substituir os espaços reservados, use o comando **CTRL+S** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

### Escrever código para se conectar ao seu projeto e conversar com seu modelo

> **Dica**: ao adicionar código, certifique-se de manter o recuo correto.

1. Digite o seguinte comando para editar o arquivo de código que foi fornecido:

    ```
   code dalle-client.py
    ```

1. No arquivo de código, observe as instruções existentes que foram adicionadas na parte superior do arquivo para importar os namespaces do SDK necessários. Em seguida, no comentário **Adicionar referências**, adicione o seguinte código para referenciar os namespaces nas bibliotecas que você instalou anteriormente:

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential, get_bearer_token_provider
   from openai import AzureOpenAI
   import requests
    ```

1. Na função **main**, no comentário **Get configuration settings**, observe que o código carrega os valores de ponto de extremidade, versão da API e nome de implantação de modelo que você definiu no arquivo de configuração.

1. No comentário **Initialize the client**, adicione o seguinte código para se conectar ao seu modelo usando as credenciais do Azure com as quais você está conectado no momento:

    ```python
   # Initialize the client
   token_provider = get_bearer_token_provider(
       DefaultAzureCredential(exclude_environment_credential=True,
           exclude_managed_identity_credential=True), 
       "https://cognitiveservices.azure.com/.default"
   )
    
   client = AzureOpenAI(
       api_version=api_version,
       azure_endpoint=endpoint,
       azure_ad_token_provider=token_provider
   )
    ```

1. Observe que o código inclui um loop para permitir que o usuário insira um prompt até digitar "quit". Em seguida, na seção de loop, no comentário **Generate an image**, adicione o seguinte código para enviar o prompt e recuperar o URL da imagem gerada a partir do modelo:

    **Python**

    ```python
   # Generate an image
   result = client.images.generate(
        model=model_deployment,
        prompt=input_text,
        n=1
    )

   json_response = json.loads(result.model_dump_json())
   image_url = json_response["data"][0]["url"] 
    ```

1. O código no restante da função **main** passa o URL da imagem e um nome de arquivo para uma função fornecida, que baixa a imagem gerada e a salva como um arquivo .png.

1. Use o comando **CTRL+S** para salvar suas alterações no arquivo de código e, em seguida, use o comando **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

### Executar o aplicativo cliente

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para entrar no Azure.

    ```
   az login
    ```

    **<font color="red">Você deve entrar no Azure, mesmo que a sessão do Cloud Shell já esteja autenticada.</font>**

    > **Observação**: na maioria dos cenários, apenas usar *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant* . Consulte [Entrar no Azure interativamente usando a CLI do Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.

1. Quando solicitado, siga as instruções para abrir a página de entrada em uma nova guia e insira o código de autenticação fornecido e suas credenciais do Azure. Em seguida, conclua o processo de entrada na linha de comando, selecionando a assinatura que contém o hub da Fábrica de IA do Azure, se solicitado.

1. No painel de linha de comando do Cloud Shell, insira o seguinte comando para executar o aplicativo:

    ```
   python dalle-client.py
    ```

1. Quando solicitado, insira uma solicitação de imagem, como `Create an image of a robot eating pizza`. Após alguns instantes, o aplicativo confirmará que a imagem foi salva.

1. Teste mais alguns prompts. Quando terminar, digite `quit` para sair do programa.

    > **Observação**: neste aplicativo simples, não implementamos lógica para reter o histórico da conversa; portanto, o modelo tratará cada prompt como uma nova solicitação sem contexto do prompt anterior.

1. Para baixar e exibir as imagens que foram geradas pelo seu aplicativo, use o comando **download** do Cloud Shell, especificando o arquivo .png que foi gerado:

    ```
   download ./images/image_1.png
    ```

    O comando download cria um link pop-up no canto inferior direito do seu navegador, que você pode selecionar para baixar e abrir o arquivo.

## Resumo

Neste exercício, você usou a Fábrica de IA do Azure e o SDK do OpenAI do Azure para criar um aplicativo cliente usando um modelo DALL-E para gerar imagens.

## Limpar

Se tiver terminado de explorar o modelo DALL-E, você deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou reabra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` em uma nova guia do navegador) e visualize o conteúdo do grupo de recursos onde você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
